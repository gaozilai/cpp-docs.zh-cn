---
title: ARM 异常处理
ms.date: 07/11/2018
ms.assetid: fe0e615f-c033-4ad5-97f4-ff96af45b201
ms.openlocfilehash: f4e56284ce8db18ec76b0143253ee1e25f3fd82c
ms.sourcegitcommit: 28eae422049ac3381c6b1206664455dbb56cbfb6
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/31/2019
ms.locfileid: "66450490"
---
# <a name="arm-exception-handling"></a>ARM 异常处理

针对异步硬件生成的异常和同步软件生成的异常，ARM 上的 Windows 将使用相同的结构化异常处理机制。 将通过使用语言帮助器函数，基于 Windows 结构化异常处理来生成特定于语言的异常处理程序。 本文档描述了 ARM 和由 Microsoft ARM 汇编程序和 MSVC 编译器生成的代码使用的语言帮助器上的 Windows 中的异常处理。

## <a name="arm-exception-handling"></a>ARM 异常处理

使用 ARM 上的 Windows*展开代码*控制堆栈展开期间[结构化异常处理](/windows/desktop/debug/structured-exception-handling)(SEH)。 展开代码是存储在可执行映像的 .xdata 部分中的字节序列。 它们以抽象的方式描述了函数序言和尾声代码的操作，以便可以撤消函数序言的效果，从而准备展开调用方的堆栈帧。

ARM EABI（嵌入应用程序二进制接口）指定使用展开代码的异常展开模式，但这对于在 Windows 中进行 SEH 展开是不够的，SEH 展开必须处理处理器位于函数序言或尾声中间的异步情况。 Windows 还将展开控制分为函数级展开和特定于语言范围展开，然而展开控制在 ARM EABI 中是统一的。 出于这些原因，ARM 上的 Windows 指定了有关展开数据和过程的详细信息。

### <a name="assumptions"></a>假设

ARM 上的 Windows 的可执行映像采用可移植可执行 (PE) 格式。 有关详细信息，请参阅[Microsoft PE 和 COFF 规范](https://go.microsoft.com/fwlink/p/?linkid=84140)。 异常处理信息存储在映像的 .pdata 和 .xdata 部分中。

异常处理机制对遵循 ARM 上 Windows 的 ABI 的代码做出了一些假设：

- 当异常出现在函数主体内部时，无论是撤销序言操作，还是以前进的方式执行尾声操作都不重要。 这两种操作应产生完全相同的结果。

- 序言和尾声往往互为镜像。 这可以用于减小描述展开所需的元数据的大小。

- 函数往往相对较小。 多个优化依赖于此，以有效地打包数据。

- 如果条件放置在尾声上，则同样适用于该尾声中的每个指令。

- 如果堆栈指针 (SP) 保存在序言中的另一个寄存器中，则该寄存器必须在整个函数中保持不变，以便可以随时恢复原始 SP。

- 除非 SP 保存在另一个寄存器中，否则它的所有操作都必须在序言和尾声内严格执行。

- 若要展开任意堆栈帧，需要执行以下操作：

  - 以 4 字节为增量调整 r13 (SP)。

  - 弹出一个或多个整数寄存器。

  - 弹出一个或多个 VFP（虚拟浮点）寄存器。

  - 将任意的寄存器值复制到 r13 (SP)。

  - 通过使用小的后递减操作，从堆栈加载 SP。

  - 分析某个明确定义的帧类型。

### <a name="pdata-records"></a>.pdata 记录

PE 格式的映像中的 .pdata 记录是由固定长度项组成的有序数组，这些项描述了每个堆栈操作的函数。 Leaf 函数（不调用其他函数的函数）不操作堆栈时不需要 .pdata 记录。 （即它们不需要任何本地存储且不必保存或还原非易失性的寄存器。）。 可以从 .pdata 部分省略关于这些函数的记录以节省空间。 来自这些函数之一的展开操作只能将返回地址从链接寄存器 (LR) 复制到程序计数器 (PC)，以向上移动到调用方。

ARM 的每个 .pdata 记录的长度是 8 个字节。 记录的一般格式是将函数的相对虚拟地址 (RVA) 放置在头 32 位字的开始位置，后跟第二个字，该字包含指向长度可变的 .xdata 块的指针，或描述规范函数展开序列的已打包的字，如下表所示：

|字偏移量|Bits|用途|
|-----------------|----------|-------------|
|0|0-31|*函数启动 RVA*是函数开始的 32 位 RVA。 如果该函数包含 Thumb 代码，则必须设置该地址的低位。|
|1|0-1|*标志*是指示如何解释第二个.pdata 字的剩余 30 位的 2 位字段。 如果*标志*为 0，则剩余的位将形成*异常信息 RVA* (与低两位隐式 0)。 如果*标志*为非零，则剩余的位将形成*打包展开数据*结构。|
|1|2-31|*异常信息 RVA*或*打包展开数据*。<br /><br /> *异常信息 RVA*是存储在.xdata 部分中的长度可变的异常信息结构的地址。 此数据必须是对齐的 4 字节。<br /><br /> *打包展开数据*是采用规范格式的函数从展开所需的操作的概括的说明。 在这种情况下，不需要任何 .xdata 记录。|

### <a name="packed-unwind-data"></a>已打包的展开数据

对于序言和尾声遵循如下所述的规范格式的函数，可以使用已打包的展开数据。 这样便无需 .xdata 记录，并且可以显著减少提供展开数据所需的空间。 规范的序言和尾声旨在满足简单函数的常见要求，该函数不需要异常处理程序并且会按照标准顺序执行其安装和停止操作。

此表显示了具有已打包展开数据的 .pdata 记录的格式：

|字偏移量|Bits|用途|
|-----------------|----------|-------------|
|0|0-31|*函数启动 RVA*是函数开始的 32 位 RVA。 如果该函数包含 Thumb 代码，则必须设置该地址的低位。|
|1|0-1|*标志*是 2 位字段，它具有以下含义：<br /><br />-00 = 已打包展开数据未使用;剩余的位指向.xdata 记录。<br />-01 = 已打包展开数据。<br />-10 = 已打包展开的数据其中假定函数没有序言。 这对于描述未与函数开始位置进行连接的函数片段很有用。<br />-11 = 保留。|
|1|2-12|*函数长度*是一个 11 位字段，提供以字节为单位除以 2 的整个函数的长度。 如果函数的大小大于 4K 字节，则必须改用一个完整的 .xdata 记录。|
|1|13-14|*Ret*是一个 2 位字段，指示该函数将返回方式：<br /><br />-00 = 通过 pop {pc} 返回 ( *L*标志位必须在这种情况下设置为 1)。<br />-01 = 通过使用 16 位分支返回。<br />-10 = 通过使用 32 位分支返回。<br />-11 = 根本没有任何尾声。 这对于描述一个不连续的函数片段很有用，该片段只能包含一个序言，然而其尾声位于其他位置。|
|1|15|*H*是一个 1 位标志，指示是否在函数"寻址"的整数参数寄存器 (r0-r3) 通过将其推送该函数的开头并返回之前释放 16 字节的堆栈。 （0 = 不会对寄存器进行寻址，1 = 对寄存器进行寻址。）|
|1|16-18|*Reg*指示的索引的最后一个 3 位字段保存非易失寄存器。 如果*R*位为 0，则仅整数寄存器保存，并且假定 r4-rN，其中 N 等于 4 的范围是 + *Reg*。如果*R*位是 1，则仅浮点寄存器保存，并且假定 d8-dN，其中 N 等于 8 的范围为 + *Reg*。特殊的组合*R* = 1 和*Reg* = 7 表示未保存任何寄存器。|
|1|19|*R*是一个 1 位标志，指示是否已保存的非易失寄存器是整数寄存器 (0) 还是浮点寄存器 (1)。 如果*R*设置为 1 并*Reg*字段设置为 7，已推送任何非易失性寄存器。|
|1|20|*L*是一个 1 位标志，指示是否该函数将保存/还原 LR 以及由指示的其他寄存器*Reg*字段。 （0 = 不保存/还原，1 = 保存/还原。）|
|1|21|*C*是 1 位标志，它指示函数是否包括额外的指令，若要设置帧链以加快堆栈审核 (1)，或不 (0)。 如果设置了此位，则会将 r11 隐式地添加到已保存的整数非易失寄存器列表中。 (请参阅以下 if 限制*C*使用标志。)|
|1|22-31|*堆栈调整*指示为此函数，除以 4 分配堆栈的字节数是 10 位域。 但是，只能直接对 0x000-0x3F3 之间的值进行编码。 分配超过 4044 字节的堆栈的函数必须使用完整的 .xdata 记录。 如果*堆栈调整*字段大于 0x3F4 或更大，则低 4 位具有特殊含义：<br /><br />位 0-1 指示堆栈调整 (1-4) 减 1 的单词数。<br />-如果序言此调整合并到其推送操作，2 位设置为 1。<br />如果尾声此调整合并到其弹出操作位 3 设置为 1。|

由于在上述编码中可能存在冗余，因此将受到下列限制：

- 如果*C*标志设置为 1:

   - *L*标志必须也设置为 1，因为帧链需要使用 r11 和 LR。

   - r11 必须不包含所描述的寄存器集中*Reg*。也就是说，如果推送 r4-r11，则*Reg*应当只描述 r4-r10，因为*C*标志表示 r11。

- 如果*Ret*字段设置为 0， *L*标志必须设置为 1。

违反这些限制会导致出现一个不支持的序列。

对于下面的论述中，两个伪标志派生自*堆栈调整*:

- *PF*或"序言折叠"指示*堆栈调整*大于 0x3F4 或更大、 位 2 设置。

- *EF*或"尾声折叠"指示*堆栈调整*大于 0x3F4 或更大、 位 3 设置。

规范函数的序言可能最多具有 5 个指令（请注意 3a 和 3b 是互斥的）：

|指令|在下列情况下将假定存在操作码：|大小|操作码|展开代码|
|-----------------|-----------------------------------|----------|------------|------------------|
|1|*H*==1|16|`push {r0-r3}`|04|
|2|*C*= = 1 或*L*= = 1 或*R*= = 0 或 PF = = 1|16/32|`push {registers}`|80-BF/D0-DF/EC-ED|
|3a|*C*= = 1 和 (*L*= = 0 和*R*= = 1 且 PF = = 0)|16|`mov r11,sp`|C0-CF/FB|
|3b|*C*= = 1 和 (*L*= = 1 或*R*= = 0 或 PF = = 1)|32|`add r11,sp,#xx`|FC|
|4|*R*= = 1 并且*Reg* ！ = 7|32|`vpush {d8-dE}`|E0-E7|
|5|*堆栈调整*！ = 0 和 PF = = 0|16/32|`sub sp,sp,#xx`|00-7F/E8-EB|

指令 1 始终是存在如果*H*位设置为 1。

若要设置帧链，指令 3a 或 3b 是存在如果*C*位设置。 如果未推送任何寄存器（r11 和 LR 除外），则为 16 位 `mov`；否则为 32 位 `add`。

如果指定了不可折叠的调整，则指令 5 为显式堆栈调整。

指令 2 和指令 4 是根据是否需要推送进行设置的。 下表汇总了进行保存的寄存器根据*C*， *L*， *R*，以及*PF*字段。 在所有情况下， *N*等于*Reg* + 4， *E*等于*Reg* + 8，和*S*等于 (~*堆栈调整*)) (& 3。

|C|L|R|PF|已推送的整数寄存器|已推送的 VFP 寄存器|
|-------|-------|-------|--------|------------------------------|--------------------------|
|0|0|0|0|r4-r*N*|无|
|0|0|0|1|r*S*-r*N*|无|
|0|0|1|0|无|d8-d*E*|
|0|0|1|1|r*S*-r3|d8-d*E*|
|0|1|0|0|r4-r*N*，LR|无|
|0|1|0|1|r*S*-r*N*，LR|无|
|0|1|1|0|LR|d8-d*E*|
|0|1|1|1|r*S*-r3，LR|d8-d*E*|
|1|0|0|0|r4-r*N*，r11|无|
|1|0|0|1|r*S*-r*N*，r11|无|
|1|0|1|0|r11|d8-d*E*|
|1|0|1|1|r*S*-r3，r11|d8-d*E*|
|1|1|0|0|r4-r*N*，r11，LR|无|
|1|1|0|1|r*S*-r*N*，r11，LR|无|
|1|1|1|0|r11、LR|d8-d*E*|
|1|1|1|1|r*S*-r3，r11，LR|d8-d*E*|

规范函数的尾声遵循类似的形式，但按照相反的顺序并且具有一些附加选项。 尾声最长可为 5 个指令，并且其形式严格由序言的形式指定。

|指令|在下列情况下将假定存在操作码：|大小|操作码|
|-----------------|-----------------------------------|----------|------------|
|6|*堆栈调整*！ = 0 和*EF*= = 0|16/32|`add   sp,sp,#xx`|
|7|*R*= = 1 并且*Reg*！ = 7|32|`vpop  {d8-dE}`|
|8|*C*= = 1 或 (*L*= = 1 和*H*= = 0) 或*R*= = 0 或*EF*= = 1|16/32|`pop   {registers}`|
|9a|*H*= = 1 并且*L*= = 0|16|`add   sp,sp,#0x10`|
|9b|*H*= = 1 并且*L*= = 1|32|`ldr   pc,[sp],#0x14`|
|10a|*Ret*==1|16|`bx    reg`|
|10b|*Ret*==2|32|`b     address`|

如果指定了不可折叠的调整，则指令 6 为显式堆栈调整。 因为*PF*无关*EF*，可以有指令 5 存在且没有指令 6 或进行相反转换。

7 和 8 的说明使用相同的逻辑与序言来确定哪些寄存器将还原从堆栈中，但有了这些两个更改： 第一个， *EF*用来代替*PF*; 第二，如果*Ret* = 0，则 LR 将被替换为 PC 寄存器列表中和尾声将立即结束。

如果*H*设置，则存在指令 9a 或 9b 则。 使用指令 9a 时*L*为 0，以指示 LR 不在堆栈上。 在这种情况下，手动调整堆栈并*Ret*必须为 1 或 2，以指定显式返回。 使用指令 9b 时*L*为 1，以指示提早终止尾声并且同时返回并调整堆栈。

如果存在，以指示 16 位或 32 位分支尾声尚未终止，然后指令 10a 或 10b，基于的值*Ret*。

### <a name="xdata-records"></a>.xdata 记录

当已打包的展开格式不足以描述函数的展开时，必须创建长度可变的 .xdata 记录。 该记录的地址存储在 .pdata 记录的第二个字中。 .xdata 的格式是已打包的长度可变的字集，具有四个部分：

1. 1 个字或 2 个字的标头，描述 .xdata 结构的总大小并提供密钥函数数据。 第二个字时才存在*尾声计数*并*代码字*字段将设置为 0。 此表中已对这些字段进行细分：

   |字|Bits|用途|
   |----------|----------|-------------|
   |0|0-17|*函数长度*是一个 18 位字段，指示以字节为单位，除以 2 函数的总长度。 如果函数的大小大于 512 KB，则必须使用多个 .pdata 和 .xdata 记录来描述该函数。 有关详细信息，请参阅本文档中的“大函数”部分。|
   |0|18-19|*Vers*是描述剩余 xdata 的版本 2 位字段。 当前仅定义版本 0；保留了 1-3 的值。|
   |0|20|*X*是 1 位字段，指示存在 (1) 或不存在 (0) 的异常数据。|
   |0|21|*E*是 1 位字段，它指示描述单个尾声的信息将打包到标头 (1) 而不是需要其他作用域的词更高版本 (0)。|
   |0|22|*F*是 1 位字段，该值指示此记录描述函数片段 (1) 或完整函数 (0)。 片段表示没有序言，还表示应忽略所有的序言处理。|
   |0|23-27|*尾声计数*是具有两种含义，具体取决于状态 5 位字段*E*位：<br /><br /> -如果*E*为 0，则此字段是第 3 节中所述异常范围总数的计数。 如果该函数，则此字段中存在多个 31 个范围并*代码字*字段必须都设置为 0 以指示需要一个扩展字。<br />-如果*E*为 1，此字段指定仅描述尾声的第一个展开代码的索引。|
   |0|28-31|*代码字*是一个 4 位字段，指定以包含所有第 4 节中的展开代码的 32 位字的数目。 如果超过 63 个展开代码字节，此字段需要超过 15 个字并*尾声计数*字段必须都设置为 0 以指示需要一个扩展字。|
   |1|0-15|*扩展尾声计数*是提供更多的空间编码极大量的尾声的 16 位字段。 包含此字段的扩展字时才存在*尾声计数*并*代码字*第一个标头字中的字段将设置为 0。|
   |1|16-23|*扩展代码字*是一个 8 位字段，提供编码极大量的展开代码字的更多的空间。 包含此字段的扩展字时才存在*尾声计数*并*代码字*第一个标头字中的字段将设置为 0。|
   |1|24-31|保留|

1. 异常数据之后 (如果*E*标头中的位设置为 0) 是一系列有关尾声范围都将打包为一个字的其中一个并存储在升序起始偏移量的信息。 每个范围都将包含以下字段：

   |Bits|用途|
   |----------|-------------|
   |0-17|*尾声开始偏移量*是一个 18 位字段，它描述尾声，以字节为单位的一半，相对于函数的开始的偏移量。|
   |18-19|*Res*是为将来扩展保留的 2 位字段。 其值必须为 0。|
   |20-23|*条件*是 4 位字段，它提供用于执行尾声的条件。 对于无条件尾声，它应设为 0xE，指示“始终”。 （尾声必须是完全有条件的或完全无条件的，并且处于 Thumb-2 模式，以 IT 操作码后的第一个指令开头。）|
   |24-31|*尾声开始索引*是一个 8 位字段，它指示描述此尾声的第一个展开代码的字节索引。|

1. 尾声范围列表之后是包含展开代码的字节数组，本文章中的“展开代码”部分将对它们进行详细介绍。 在最接近的全字边界的末尾处填充此数组。 按 Little-Endian 的顺序存储字节，以便可以直接在 Little-Endian 模式下提取它们。

1. 如果*X*标头中的字段为 1，则展开代码字节后跟异常处理程序信息。 这由一个*异常处理程序 RVA*包含地址的异常处理程序，然后输入 （长度可变） 异常处理程序所需的数据量。

.xdata 记录的设计目的是为了能够获取前 8 个字节并计算该记录的完整大小，其中不包括它后面的大小可变的异常数据的长度。 此代码段将计算该记录大小：

```cpp
ULONG ComputeXdataSize(PULONG *Xdata)
{
    ULONG EpilogueScopes;
    ULONG Size;
    ULONG UnwindWords;

    if ((Xdata[0] >> 23) != 0) {
        Size = 4;
        EpilogueScopes = (Xdata[0] >> 23) & 0x1f;
        UnwindWords = (Xdata[0] >> 28) & 0x0f;
    } else {
        Size = 8;
        EpilogueScopes = Xdata[1] & 0xffff;
        UnwindWords = (Xdata[1] >> 16) & 0xff;
    }

    if (!(Xdata[0] & (1 << 21))) {
        Size += 4 * EpilogueScopes;
    }
    Size += 4 * UnwindWords;
    if (Xdata[0] & (1 << 20)) {
        Size += 4;
    }
    return Size;
}
```

虽然序言和每个尾声展开代码的索引，它们之间共享表。 它们都可以共享相同的展开代码的情况并不少见。 建议在这种情况下对编译器编写器进行优化，因为可指定的最大索引为 255，并且它限制了特定函数可能的展开代码的总数。

### <a name="unwind-codes"></a>展开代码

展开代码的数组是一个指令序列的池，这些指令序列准确描述了如何按照必须撤消操作的顺序来撤消序言的效果。 展开代码是编码为字节字符串的微型指令集。 执行完成后，可以在 LR 寄存器中找到调用函数的返回地址，并且非易失寄存器将还原为调用函数时它们的值。

如果确保异常仅出现在函数主体内部，并且始终不会出现在序言或尾声内部，则仅需要一个展开序列。 但是，Windows 展开模式需要能够从部分执行的序言或尾声中进行展开。 为了满足此需求，已谨慎地将展开代码设计为对序言和尾声中的每个相关操作码进行明确的一对一映射。 这具有几方面的含义：

- 可通过计算展开代码的数量来计算序言和尾声的长度。 之所以即使使用长度可变的 Thumb-2 指令此操作也可行，是因为对于 16 位和 32 位操作码，存在不同的映射。

- 通过跳过尾声范围的开始位置对指令数进行计数，可以跳过同等数量的展开代码并执行该序列的剩余部分，以完成由该尾声执行的部分执行的展开。

- 通过在序言末尾之前对指令数进行计数，可以跳过同等数量的展开代码并执行该序列的剩余部分，从而可以只撤消已完成执行的那些部分的序言。

下表显示了从展开代码到操作码的映射。 最常见的代码仅为一个字节，然而较少见的代码需要两个、三个或者甚至四个字节。 将每个代码从最高有效字节存储到最低有效字节。 展开代码结构与 ARM EABI 中描述的编码不同，因为已将这些展开代码设计为对序言和尾声中的操作码进行一对一映射，以允许对部分执行的序言和尾声进行展开。

|字节 1|字节 2|字节 3|字节 4|Opsize|说明|
|------------|------------|------------|------------|------------|-----------------|
|00-7F||||16|`add   sp,sp,#X`<br /><br /> 其中 X 等于 (Code & 0x7F) \* 4|
|80-BF|00-FF|||32|`pop   {r0-r12, lr}`<br /><br /> LR 将弹出，如果 Code & 0x2000 和 r0-r12 弹出，如果 Code & 0x1FFF 中设置了对应位|
|C0-CF||||16|`mov   sp,rX`<br /><br /> 其中 X 等于 Code & 0x0F|
|D0-D7||||16|`pop   {r4-rX,lr}`<br /><br /> 其中 X 等于 (Code & 0x03) + 4 和 LR 将弹出，则代码 & 0x04，则|
|D8-DF||||32|`pop   {r4-rX,lr}`<br /><br /> 其中 X 等于 (Code & 0x03) + 8 和 LR 将弹出，则代码 & 0x04，则|
|E0-E7||||32|`vpop  {d8-dX}`<br /><br /> 其中 X 等于 (Code & 0x07) + 8|
|E8-EB|00-FF|||32|`addw  sp,sp,#X`<br /><br /> 其中 X 等于 (Code & 0x03FF) \* 4|
|EC-ED|00-FF|||16|`pop   {r0-r7,lr}`<br /><br /> LR 将弹出，如果 Code & 0x0100 和 r0-r7 弹出，如果 Code & 0x00FF 中设置了对应位|
|EE|00-0F|||16|Microsoft 专用|
|EE|10-FF|||16|可用|
|EF|00-0F|||32|`ldr   lr,[sp],#X`<br /><br /> 其中 X 等于 (Code & 0x000F) \* 4|
|EF|10-FF|||32|可用|
|F0-F4||||-|可用|
|F5|00-FF|||32|`vpop  {dS-dE}`<br /><br /> 其中 S 等于 (Code & 0x00F0) >> 4 且 E 等于 Code & 0x000F|
|F6|00-FF|||32|`vpop  {dS-dE}`<br /><br /> 其中，S 是 ((Code & 0x00F0) >> 4) + 16 且 E 等于 (Code & 0x000F) + 16|
|F7|00-FF|00-FF||16|`add   sp,sp,#X`<br /><br /> 其中 X 等于 (Code & 0x00FFFF) \* 4|
|F8|00-FF|00-FF|00-FF|16|`add   sp,sp,#X`<br /><br /> 其中 X 等于 (Code & 0x00FFFFFF) \* 4|
|F9|00-FF|00-FF||32|`add   sp,sp,#X`<br /><br /> 其中 X 等于 (Code & 0x00FFFF) \* 4|
|FA|00-FF|00-FF|00-FF|32|`add   sp,sp,#X`<br /><br /> 其中 X 等于 (Code & 0x00FFFFFF) \* 4|
|FB||||16|nop（16 位）|
|FC||||32|nop（32 位）|
|FD||||16|end + 尾声中的 16 位 nop|
|FE||||32|end + 尾声中的 32 位 nop|
|FF||||-|end|

这将显示为每个字节的十六进制值的范围中的展开代码*代码*，以及操作码大小*Opsize*和相应的原始指令解释。 空单元格指示较短的展开代码。 在具有涵盖多个字节的较大值的指令中，将首先存储最高有效位。 *Opsize*字段显示了与每个 thumb-2 操作关联的隐式操作码大小。 具有不同编码的表格中明显重复的项将用来区分不同的操作码大小。

展开代码经过专门设计，以便代码中的第一个字节可以指示代码的总大小（以字节为单位）以及指令流中相应操作码的大小。 若要计算序言或尾声的大小，请从序列的开头审核到结尾，然后使用查找表或类似方法来确定相应操作码的长度。

展开代码 0xFD 和 0xFE 等效于常规结束代码 0xFF，但在尾声用例（16 位或 32 位）中需要考虑一个额外的 nop 操作码。 对于序言，代码 0xFD、0xFE 和 0xFF 完全等效。 这考虑了常见的尾声结尾 `bx lr` 或 `b <tailcall-target>`，它们不具有等效的序言指令。 这样会增加展开序列可在序言和尾声之间共享的可能性。

在许多情况下，应当可以对序言和所有尾声使用相同的展开代码集。 但是，若要处理部分执行的序言和尾声的展开，则你可能必须拥有多个展开代码序列，这些序列因排列或行为而有所不同。 这就是每个尾声都具有其各自的指向展开数组的索引以显示开始执行的位置的原因。

### <a name="unwinding-partial-prologues-and-epilogues"></a>展开部分序言和尾声

最常见的展开情况是，异常在函数主体中出现，远离序言和所有的尾声。 在这种情况下，展开器将从索引 0 处开始执行展开数组中的代码，并且在检测到结束操作码之前将一直持续该操作。

如果在执行序言或尾声时出现异常，则仅构造部分堆栈帧，并且展开器必须准确确定已执行的操作，以便可以正确地撤消它。

例如，请考虑此序言和尾声序列：

```asm
0000:   push  {r0-r3}         ; 0x04
0002:   push  {r4-r9, lr}     ; 0xdd
0006:   mov   r7, sp          ; 0xc7
...
0140:   mov   sp, r7          ; 0xc7
0142:   pop   {r4-r9, lr}     ; 0xdd
0146:   add   sp, sp, #16     ; 0x04
0148:   bx    lr
```

在每个操作码的旁边是相应的展开代码，用于描述此操作。 序言展开代码的序列是尾声展开代码的镜像（不计入最终指令）。 这种情况很常见，并且也是始终将序言的展开代码假设为以与序言的执行顺序相反的顺序进行存储的原因。 这会为我们提供一组常规的展开代码：

```asm
0xc7, 0xdd, 0x04, 0xfd
```

0xFD 代码是序列末尾处的特殊代码，它表示尾声是一个长度超过序言的 16 位指令。 这样能够进一步共享展开代码。

在此示例中，如果在执行序言和尾声之间的函数主体时出现异常，则将从尾声用例（尾声代码中偏移量为 0 处）开始进行展开。 这对应于示例中的偏移量 0x140。 展开器将执行完全展开的序列，因为没有进行任何清理操作。 而如果在尾言代码开头后的某个指令处出现异常，则展开器可以通过跳过第一个展开代码成功进行展开。 给定操作码之间的一对一映射和展开代码，如果从指令*n*尾声中展开器应跳过第一个*n*展开代码。

类似的逻辑适用于序言的反向操作。 如果从序言中的偏移量 0 处展开，则无需执行任何操作。 如果从其中的某个指令展开，则展开序列应从结尾处的某个展开代码开始，因为序言展开代码是以反向顺序进行存储的。 一般情况下，如果从指令*n*在序言中，展开应处开始执行*n*展开代码的列表末尾的代码。

序言展开代码和尾声展开代码并不总是完全匹配。 在这种情况下，展开代码数组可能必须包含几个代码序列。 若要确定开始处理代码的偏移量，请使用此逻辑：

1. 如果从函数主体内部展开，则请在索引 0 处开始执行展开代码，并且继续该操作，直到到达结束操作码。

2. 如果从尾声内部展开，则请使用由尾声范围提供的特定于尾声的起始索引。 计算 PC 从尾声开始位置读取的字节数。 在展开代码中快进，直到处理完所有已执行的指令。 从该点开始执行展开序列。

3. 如果从序言内部展开，则请从展开代码中索引 0 处开始。 计算序列中的序言代码的长度，然后计算 PC 从序言结束位置读取的字节数。 在展开代码中快进，直到处理完所有未执行的指令。 从该点开始执行展开序列。

序言的展开代码必须始终位于数组的最前面。 在从主体内部展开的一般情况下，序言的展开代码也是用于展开的代码。 特定于尾声的代码序列应紧跟在序言代码序列之后。

### <a name="function-fragments"></a>函数片段

对于代码优化，将函数拆分为不连续的部分会很有用。 完成拆分后，每个函数片段都需要各自单独的 .pdata（也可能为 .xdata）记录。

假定函数序言位于函数的开头并且无法拆分，则函数片段存在以下四种情况：

- 仅限序言；其他片段中的所有尾声。

- 序言和一个或多个尾声；其他片段中的附加尾声。

- 不存在任何序言和尾声；其他片段中的序言以及一个或多个尾声。

- 仅限序言；其他片段中的序言和可能的附加尾声。

在第一种情况下，只须描述序言。 这通过照常描述序言并指定可以在精简的.pdata 格式*Ret*值为 3 以指示不存在尾声。 可以通过照常在索引 0 处提供序言展开代码，并指定尾声计数为 0，采用完整的 .xdata 格式来完成此操作。

第二种情况极类似于一个常规函数。 如果片段中只有一个尾声且位于该片段的末尾处，则可以使用精简的 .pdata 记录。 否则，必须使用完整的 .xdata 记录。 请记住，为尾声的开始位置指定的偏移量与片段的开始位置（而非函数的原始开始位置）相关。

第三和第四种情况分别为第一和第二种情况的变体，不同之处是它们不包含序言。 在这些情况下，假定在尾言开始位置之前存在代码且将其视为函数主体的一部分，这通常是通过撤消序言的效果进行展开的。 因此必须使用伪序言对这些情况进行编码，该伪序言描述了如何从函数主体内部展开，但在确定是否在片段的开始位置执行部分展开时，需要将其长度视为 0。 或者，可以使用与尾声相同的展开代码来描述该伪序言，因为它们可能执行等效操作。

在第三个和第四个的情况下，通过设置指定伪序言的存在*标志*精简的.pdata 记录为 2，或通过设置字段*F*为 1 的.xdata 标头中的标志。 在任一情况下，将忽略针对部分序言展开进行检查，并且将所有非尾声展开都视为完整的展开。

#### <a name="large-functions"></a>大函数

片段可用于描述大小超过 .xdata 标头中位字段设定的 512 KB 限制的函数。 若要描述非常大的函数，则只需将其分解为小于 512 KB 的片段。 应当调整每个片段，以便它不会将尾声拆分为多个部分。

仅函数的第一个片段包含序言；所有其他片段都被标记为不包含序言。 根据尾声的数目，每个片段可能包含零个或多个尾声。 请记住，片段中的每个尾声范围指定相对于该片段开头处（而非函数的开始位置）的起始偏移量。

如果片段没有序言和尾声，则它仍然需要自己的 .pdata（也可能是 .xdata）记录，以描述如何从函数主体内部展开。

#### <a name="shrink-wrapping"></a>紧缩套装

是一个更复杂的函数片段特例*紧缩*，方法用于将寄存器保存从开头到更高版本中的函数，以优化的简单情况，无需寄存器保存的函数。 可将它描述为用于分配堆栈空间但会保存最小寄存器集的外部区域，也可描述为用于保存并还原其他寄存器的内部区域。

```asm
ShrinkWrappedFunction
    push   {r4, lr}          ; A: save minimal non-volatiles
    sub    sp, sp, #0x100    ; A: allocate all stack space up front
    ...                      ; A:
    add    r0, sp, #0xE4     ; A: prepare to do the inner save
    stm    r0, {r5-r11}      ; A: save remaining non-volatiles
    ...                      ; B:
    add    r0, sp, #0xE4     ; B: prepare to do the inner restore
    ldm    r0, {r5-r11}      ; B: restore remaining non-volatiles
    ...                      ; C:
    pop    {r4, pc}          ; C:
```

通常，紧缩套装函数应当为在常规序言中保存的其他寄存器预分配空间，然后使用 `str` 或 `stm`（而不是 `push`）来执行寄存器保存。 这会保留函数原始序言中的所有堆栈指针操作。

必须将示例紧缩套装函数拆分为三个区域，并在注释中这些区域标记为 A、B 和 C。 第一个 A 区域通过额外非易失寄存器保存的末尾来覆盖函数的开始位置。 必须构造 .pdata 记录或 .xdata 记录，以将该片段描述为具有序言而不具有尾声。

中间的 B 区域获取其自身的 .pdata 记录或 .xdata 记录，记录将描述没有序言且没有尾声的片段。 但是，此区域的展开代码仍必须存在，因为它们被视为函数主体。 这些代码必须描述一个复合序言，它表示区域 A 序言中保存的原始寄存器以及在进入区域 B 之前保存的其他寄存器，如同它们由某个操作序列生成。

不可以将区域 B 保存的寄存器视为“内部序言”，因为为区域 B 描述的复合序言必须描述区域 A 的序言和其他保存的寄存器。 如果将片段 B 描述为具有序言，则展开代码还可以表示该序言的大小，并且无法以对仅用于保存其他寄存器的操作码进行一对一映射的方式来描述复合序言。

必须将保存的其他寄存器视为区域 A 的一部分，因为在它们完成之前，复合序言无法准确描述堆栈的状态。

最后一个 C 区域获取其自身的 .pdata 记录或 .xdata 记录，记录描述了没有序言但具有尾声的片段。

如果在进入区域 B 之前即已完成的堆栈操作可简化为一个指令，则还可以使用另一种方法：

```asm
ShrinkWrappedFunction
    push   {r4, lr}          ; A: save minimal non-volatile registers
    sub    sp, sp, #0xE0     ; A: allocate minimal stack space up front
    ...                      ; A:
    push   {r4-r9}           ; A: save remaining non-volatiles
    ...                      ; B:
    pop    {r4-r9}           ; B: restore remaining non-volatiles
    ...                      ; C:
    pop    {r4, pc}          ; C: restore non-volatile registers
```

此处的关键在于，在每个指令边界，堆栈与该区域中的展开代码是完全一致的。 如果在本例中展开发生在内部推送之前，则将它视为区域 A 的一部分，并且仅展开区域 A 序言。 如果展开发生在内部推送之后，它被视为区域 B，后者没有序言但具有描述内部推送和区域 A.类似的逻辑中原始序言的展开代码的一部分包含于内部弹出。

### <a name="encoding-optimizations"></a>编码优化

由于展开代码的丰富性以及利用数据的压缩和扩展形式的能力，存在很多机会来优化编码，从而进一步缩小空间。 随着这些技术的主动使用，使用展开代码描述函数和片段的净开销将变得非常小。

最重要的优化是，注意不要将用于展开的序言/尾声边界与编译器角度的逻辑序言/尾声边界混淆。 可收缩展开边界，使其更为紧密，以提高效率。 例如，在堆栈设置之后，序言可能包含用于执行其他验证检查的代码。 但是，所有堆栈操作都完成后，无需对后续操作进行编码，并且可从展开序言中删除堆栈操作以外的任何操作。

此相同的规则适用于函数长度。 如果函数中的尾声之后存在数据（例如，文本池），则不应将其作为函数长度的一部分而包括进来。 通过将函数收缩为仅为该函数一部分的代码，尾声位于最末尾位置的可能性将会非常高 并且可以使用精简的 .pdata 记录。

在序言中，将堆栈指针保存到另一个寄存器之后，通常无需记录任何进一步的操作码。 若要展开函数，则需要完成的第一个操作是从保存的寄存器中还原 SP，从而使进一步的操作不会对展开产生任何影响。

完全不必将单指令尾声编码为范围代码或展开代码。 如果展开发生在执行该指令之前，则将该展开假定为来自函数主体内部，并且只需执行序言展开代码即可。 如果展开发生在执行该单个指令之后，则按照定义它将发生在其他区域中。

出于与之前相同的原因：如果展开发生在执行该指令之前，则只需执行完整的序言展开即可。因此多指令尾声不必对尾声的首个指令进行编码。 如果展开发生在该指令之后，则仅需要考虑后续操作。

展开代码重用应该是主动的。 由每个尾声范围指定的索引都指向展开代码数组中的任意一个起点。 它无需指向前一个序列的开始位置；它可以指向中间位置。 此处的最佳方法是生成所需的代码序列，然后在已编码的序列池中扫描准确的字节匹配，并将任何的完全匹配用作重用的起点。

如果在忽略单指令尾声后没有剩余的尾声，则请考虑使用精简的 .pdata 格式；缺少某个尾声时更有可能出现这种情况。

## <a name="examples"></a>示例

在这些示例中，映像基位于 0x00400000。

### <a name="example-1-leaf-function-no-locals"></a>示例 1：Leaf 函数，没有局部变量

```asm
Prologue:
  004535F8: B430      push        {r4-r5}
Epilogue:
  00453656: BC30      pop         {r4-r5}
  00453658: 4770      bx          lr
```

.pdata（不可编辑，2 个字）：

- 字 0

   - *函数开始 RVA* = 的 0x000535F8 （= 0x004535F8 0x00400000）

- 字 1

   - *标志*= 1，指示规范的序言和尾声格式

   - *函数长度*= 0x31 （= 0x62/2）

   - *Ret* = 1，表示 16 位分支返回

   - *H* = 0 时，不，该值指示参数进行寻址

   - *R*= 0 和*Reg* = 1，表示 r4-r5 的推送/弹出

   - *L* = 0，表示没有 LR 保存/还原

   - *C* = 0，表示没有帧链

   - *堆栈调整*= 0，表示未进行堆栈调整

### <a name="example-2-nested-function-with-local-allocation"></a>示例 2：具有本地分配的嵌套的函数

```asm
Prologue:
  004533AC: B5F0      push        {r4-r7, lr}
  004533AE: B083      sub         sp, sp, #0xC
Epilogue:
  00453412: B003      add         sp, sp, #0xC
  00453414: BDF0      pop         {r4-r7, pc}
```

.pdata（不可编辑，2 个字）：

- 字 0

   - *函数开始 RVA* = 0x000533AC (= 0x004533AC-0x00400000)

- 字 1

   - *标志*= 1，指示规范的序言和尾声格式

   - *函数长度*= 0x35 （= 0x6A/2）

   - *Ret* = 0，表示 pop {pc} 返回

   - *H* = 0 时，不，该值指示参数进行寻址

   - *R*= 0 和*Reg* = 3，表示 r4-r7 的推送/弹出

   - *L* = 1，指示 LR 保存/还原

   - *C* = 0，表示没有帧链

   - *堆栈调整*= 的 3 （= 0x0C/4）

### <a name="example-3-nested-variadic-function"></a>示例 3：嵌套的 Variadic 函数

```asm
Prologue:
  00453988: B40F      push        {r0-r3}
  0045398A: B570      push        {r4-r6, lr}
Epilogue:
  004539D4: E8BD 4070 pop         {r4-r6}
  004539D8: F85D FB14 ldr         pc, [sp], #0x14
```

.pdata（不可编辑，2 个字）：

- 字 0

   - *函数开始 RVA* = 的 0x00053988 （= 0x00453988 0x00400000）

- 字 1

   - *标志*= 1，指示规范的序言和尾声格式

   - *函数长度*= 0x2A （= 0x54/2）

   - *Ret* = 0，表示 pop {pc}-样式的返回 （在这种情况下为 ldr pc，[sp]，#0x14 返回）

   - *H* = 1，指示参数进行寻址

   - *R*= 0 和*Reg* = 2，表示 r4-r6 的推送/弹出

   - *L* = 1，指示 LR 保存/还原

   - *C* = 0，表示没有帧链

   - *堆栈调整*= 0，表示未进行堆栈调整

### <a name="example-4-function-with-multiple-epilogues"></a>示例 4：具有多个尾声的函数

```asm
Prologue:
  004592F4: E92D 47F0 stmdb       sp!, {r4-r10, lr}
  004592F8: B086      sub         sp, sp, #0x18
Epilogues:
  00459316: B006      add         sp, sp, #0x18
  00459318: E8BD 87F0 ldm         sp!, {r4-r10, pc}
  ...
  0045943E: B006      add         sp, sp, #0x18
  00459440: E8BD 87F0 ldm         sp!, {r4-r10, pc}
  ...
  004595D4: B006      add         sp, sp, #0x18
  004595D6: E8BD 87F0 ldm         sp!, {r4-r10, pc}
  ...
  00459606: B006      add         sp, sp, #0x18
  00459608: E8BD 87F0 ldm         sp!, {r4-r10, pc}
  ...
  00459636: F028 FF0F bl          KeBugCheckEx     ; end of function
```

.pdata（不可编辑，2 个字）：

- 字 0

   - *函数开始 RVA* = 的 0x000592F4 （= 0x004592F4 0x00400000）

- 字 1

   - *标志*= 0，表示存在.xdata 记录 （因多个尾声而需要）

   - *.xdata address* - 0x00400000

.xdata（变量，6 个字）：

- 字 0

   - *函数长度*= 的 0x0001A3 （= 0x000346/2）

   - *Vers* = 0，表示 xdata 的第一个版本

   - *X* = 0，表示没有异常数据

   - *E* = 0，表示尾声范围的列表

   - *F* = 0，表示完整的函数描述，包括序言

   - *尾声计数*= 0x04，指示 4 个尾声范围

   - *代码字*= 0x01，表示展开代码的一个 32 位字

- 字 1-4，描述位于 4 个位置上的 4 个尾声范围。 每个范围都具有一组可与序言共享的、偏移量为 0x00 的、无条件的、指定条件 0x0E（始终）的常规展开代码。

- 展开代码，从字 5 处开始：（在序言/尾声之间进行共享）

   - 展开代码 0 = 0x06:sp: sp + = (6 << 2)

   - 展开代码 1 = 0xDE：弹出 {r4-r10, lr}

   - 展开代码 2 = 0xFF：末尾

### <a name="example-5-function-with-dynamic-stack-and-inner-epilogue"></a>示例 5:具有动态堆栈和内部尾声的函数

```asm
Prologue:
  00485A20: B40F      push        {r0-r3}
  00485A22: E92D 41F0 stmdb       sp!, {r4-r8, lr}
  00485A26: 466E      mov         r6, sp
  00485A28: 0934      lsrs        r4, r6, #4
  00485A2A: 0124      lsls        r4, r4, #4
  00485A2C: 46A5      mov         sp, r4
  00485A2E: F2AD 2D90 subw        sp, sp, #0x290
Epilogue:
  00485BAC: 46B5      mov         sp, r6
  00485BAE: E8BD 41F0 ldm         sp!, {r4-r8, lr}
  00485BB2: B004      add         sp, sp, #0x10
  00485BB4: 4770      bx          lr
  ...
  00485E2A: F7FF BE7D b           #0x485B28    ; end of function
```

.pdata（不可编辑，2 个字）：

- 字 0

   - *函数开始 RVA* = 的 0x00085A20 （= 0x00485A20 0x00400000）

- 字 1

   - *标志*= 0，表示存在.xdata 记录 （因多个尾声而需要）

   - *.xdata address* - 0x00400000

.xdata（变量，3 个字）：

- 字 0

   - *函数长度*= 的 0x0001A3 （= 0x000346/2）

   - *Vers* = 0，表示 xdata 的第一个版本

   - *X* = 0，表示没有异常数据

   - *E* = 0，表示尾声范围的列表

   - *F* = 0，表示完整的函数描述，包括序言

   - *尾声计数*= 0x001，表示 1 个尾声范围总共

   - *代码字*= 0x01，表示展开代码的一个 32 位字

- 字 1:尾声范围偏移量为 0xC6 （= 0x18C/2），从 0x00 处并且具有条件 0x0e （始终） 开始展开代码索引

- 展开代码，从字 2 处开始：（在序言/尾声之间进行共享）

   - 展开代码 0 = 0xC6：sp = r6

   - 展开代码 1 = 0xDC：弹出 {r4-r8, lr}

   - 展开代码 2 = 0x04:sp: sp + = (4 << 2)

   - 展开代码 3 = 0xFD：末尾，计为尾声的 16 位指令

### <a name="example-6-function-with-exception-handler"></a>示例 6:具有异常处理程序函数

```asm
Prologue:
  00488C1C: 0059 A7ED dc.w  0x0059A7ED
  00488C20: 005A 8ED0 dc.w  0x005A8ED0
FunctionStart:
  00488C24: B590      push        {r4, r7, lr}
  00488C26: B085      sub         sp, sp, #0x14
  00488C28: 466F      mov         r7, sp
Epilogue:
  00488C6C: 46BD      mov         sp, r7
  00488C6E: B005      add         sp, sp, #0x14
  00488C70: BD90      pop         {r4, r7, pc}
```

.pdata（不可编辑，2 个字）：

- 字 0

   - *函数开始 RVA* = 的 0x00088C24 （= 0x00488C24 0x00400000）

- 字 1

   - *标志*= 0，表示存在.xdata 记录 （因多个尾声而需要）

   - *.xdata address* - 0x00400000

.xdata（变量，5 个字）：

- 字 0

   - *函数长度*= 的 0x000027 （= 0x00004E/2）

   - *Vers* = 0，表示 xdata 的第一个版本

   - *X* = 1，表示存在异常数据

   - *E* = 1，表示单个尾声

   - *F* = 0，表示完整的函数描述，包括序言

   - *尾声计数*= 0x00，表示尾声展开代码从偏移量 0x00 开始

   - *代码字*= 0x02，表示展开代码的两个 32 位字

- 展开代码，从字 1 处开始：

   - 展开代码 0 = 0xC7：sp = r7

   - 展开代码 1 = 0x05:sp: sp + = (5 << 2）。

   - 展开代码 2 = 0xED/0x90：弹出 {r4, r7, lr}

   - 展开代码 4 = 0xFF：末尾

- 字 3 指定异常处理程序 = 0x0019A7ED (= 0x0059A7ED-0x00400000)

- 字 4 及之后的字为内联异常数据

### <a name="example-7-funclet"></a>示例 7:Funclet

```asm
Function:
  00488C72: B500      push        {lr}
  00488C74: B081      sub         sp, sp, #4
  00488C76: 3F20      subs        r7, #0x20
  00488C78: F117 0308 adds        r3, r7, #8
  00488C7C: 1D3A      adds        r2, r7, #4
  00488C7E: 1C39      adds        r1, r7, #0
  00488C80: F7FF FFAC bl          target
  00488C84: B001      add         sp, sp, #4
  00488C86: BD00      pop         {pc}
```

.pdata（不可编辑，2 个字）：

- 字 0

   - *函数开始 RVA* = 的 0x00088C72 （= 0x00488C72 0x00400000）

- 字 1

   - *标志*= 1，指示规范的序言和尾声格式

   - *函数长度*= 0x0B （= 0x16/2）

   - *Ret* = 0，表示 pop {pc} 返回

   - *H* = 0 时，不，该值指示参数进行寻址

   - *R*= 0 和*Reg* = 7，任何寄存器，该值指示未保存/还原

   - *L* = 1，指示 LR 保存/还原

   - *C* = 0，表示没有帧链

   - *堆栈调整*= 1，表示 1 × 4 字节堆栈调整

## <a name="see-also"></a>请参阅

[ARM ABI 约定概述](overview-of-arm-abi-conventions.md)<br/>
[Visual C++ ARM 迁移的常见问题](common-visual-cpp-arm-migration-issues.md)
