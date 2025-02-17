---
title: 标准转换
ms.date: 11/19/2018
helpviewer_keywords:
- standard conversions, categories of
- L-values [C++]
- conversions, standard
ms.assetid: ce7ac8d3-5c99-4674-8229-0672de05528d
ms.openlocfilehash: aee100bdc7e8ba6dd7d06c6bca9ed39c09cf2d97
ms.sourcegitcommit: 0ab61bc3d2b6cfbd52a16c6ab2b97a8ea1864f12
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "62267270"
---
# <a name="standard-conversions"></a>标准转换

C++ 语言定义其基础类型之间的转换。 它还定义指针、引用和指向成员的指针派生类型的转换。 这些转换称为*标准转换*。

本节讨论下列标准转换：

- 整型提升

- 整型转换

- 浮点转换

- 浮点转换和整型转换

- 算术转换

- 指针转换

- 引用转换

- 指向成员的指针转换

    > [!NOTE]
    >  用户定义的类型可指定其自己的转换。 中介绍了用户定义类型的转换[构造函数](../cpp/constructors-cpp.md)并[转换](../cpp/user-defined-type-conversions-cpp.md)。

以下代码将导致转换（本例中为整型提升）：

```cpp
long  long_num1, long_num2;
int   int_num;

// int_num promoted to type long prior to assignment.
long_num1 = int_num;

// int_num promoted to type long prior to multiplication.
long_num2 = int_num * long_num2;
```

仅当转换生成引用类型时，其结果才为左值。 例如，用户定义的转换声明为`operator int&()`返回引用，并且是左值。 但是，转换声明为`operator int()`返回一个对象，并不是左值。

## <a name="integral-promotions"></a>整型提升

整数类型的对象可以转换为另一个更宽的整数类型（即，可表示更大的一组值的类型）。 这种扩展类型的转换称为“整型提升”。 利用整型提升，您可以在可使用其他整数类型的任何位置将以下项用于表达式：

- 对象、 文本和类型的常量**char**和**short int**

- 枚举类型

- **int**位域

- 枚举数

C++ 提升是“值保留”。 即，提升后的值一定与提升前的值相同。 在值保留提升中，较短的整数类型的对象 (如位域或类型的对象**char**) 提升到类型**int**如果**int**可以表示完整原始类型的范围。 如果**int**不能表示完整的一系列值，则该对象提升到类型**无符号的 int**。尽管此策略与 ANSI C 中使用的相同，但值保留转换不保留对象的“符号”。

值保留提升和保留符号的提升通常会生成相同的结果。 但是，如果提升的对象是以下项之一，它们可能生成不同的结果：

- 一个操作数 **/** ， `%`， `/=`， `%=`， **<** ， **\< =** ， **>** ，或 **>=**

   这些运算符依赖于用于确定结果的符号。 因此，当值保留和符号保留提升应用于这些操作数时，它们将生成不同的结果。

- 左的操作数 **>>** 或 **>>=**

   当执行移位运算时，这些运算符会区别对待有符号的数量和无符号的数量。 对于有符号的数量，将数量右移会导致符号位传播到空出的位位置。 对于无符号的数量，空出的位位置将由零填充。

- 重载函数的自变量，或重载运算符的操作数（取决于该操作数的用于自变量匹配的类型的符号）。 (请参阅[重载运算符](../cpp/operator-overloading.md)的更多有关定义重载运算符。)

## <a name="integral-conversions"></a>整型转换

整型转换在整型之间执行。 整型类型包括**char**， **int**，并**长**(和**短**，**签名**，和**无符号**这些类型的版本)。

**有为无符号**

有符号整数类型的对象可以转换为对应的无符号类型。 当这些转换发生时，实际位模式不会更改；但是，数据的解释会更改。 考虑此代码：

```cpp
#include <iostream>

using namespace std;
int main()
{
    short  i = -3;
    unsigned short u;

    cout << (u = i) << "\n";
}
// Output: 65533
```

在前面的示例中，**短签名**， `i`、 定义和初始化为一个负数。 表达式`(u = i)`会导致`i`可转换为**unsigned short**到赋值前`u`。

**未签名的签名**

无符号整数类型的对象可以转换为对应的有符号类型。 但是，如果无符号对象的值在有符号类型表示的范围之外，则此类转换可能导致数据错误解释，如以下示例所示：

```cpp
#include <iostream>

using namespace std;
int main()
{
short  i;
unsigned short u = 65533;

cout << (i = u) << "\n";
}
//Output: -3
```

在前面的示例中，`u`是**unsigned short**整数对象，必须转换为有符号的数量来计算该表达式`(i = u)`。 由于其值不能以正确表示**短签名**，如所示，数据被错误解释。

## <a name="floating-point-conversions"></a>浮点转换

浮动类型的对象可以安全地转换为更精确的浮点类型，也就是说，转换不会导致基数丢失。 例如，转换从**float**到**double**或从**double**到**长双精度型**是安全的并且值保持不变。

如果浮点类型的对象位于低精度类型可表示的范围中，则还可转换为该类型。 (请参阅[浮点限制](../cpp/floating-limits.md)浮点类型的范围。)如果原始值不能精确地表示，则可将其转换为下一更高或更低的可表示值。 如果此类值不存在，则结果是不确定的。 请看下面的示例：

```cpp
cout << (float)1E300 << endl;
```

按类型可表示的最大值**float**为 3.402823466E38-比 1E300 小得多的数字。 因此，该数字转换为无穷大，并且结果是"inf"。

## <a name="conversions-between-integral-and-floating-point-types"></a>整型和浮点型之间的转换

某些表达式可能导致浮点型的对象转换为整型，反之亦然。 当整型对象转换为浮点型且无法正确表示原始值时，结果要么是下一个较大的可表示值，要么是下一个较小的可表示值。

当浮点型的对象转换为整型时，小数部分将被截断。 转换过程中不会进行舍入。 截断意味着，数字 1.3 将转换为 1，-1.3 被转换为-1。

## <a name="arithmetic-conversions"></a>算术转换

很多二元运算符 (中所述[使用二进制运算符的表达式](../cpp/expressions-with-binary-operators.md)) 会导致操作数转换并产生的结果相同的方式。 这些运算符导致转换的方式称为“常用算术转换”。 不同本机类型的操作数的算术转换按下表所示的方式执行。 Typedef 类型的行为方式基于其基础本机类型。

### <a name="conditions-for-type-conversion"></a>类型转换的条件

|满足的条件|转换|
|--------------------|----------------|
|其中一个操作数的类型是**长双精度型**。|另一个操作数将转换为类型**长双精度型**。|
|上述不满足条件，并且其中一个操作数的类型是**double**。|另一个操作数将转换为类型**double**。|
|上述不满足的条件，并且其中一个操作数的类型是**float**。|另一个操作数将转换为类型**float**。|
|未满足上述条件（没有任何一个操作数属于浮动类型）。|整型提升按以下方式对操作数执行：<br /><br />-如果其中一个操作数的类型为**无符号长**，另一个操作数将转换为类型**无符号长**。<br />-如果未满足上述条件，并且如果其中一个操作数的类型**长**和其他类型的**无符号的 int**，这两个操作数都转换为键入**无符号长**。<br />-如果不满足上述两个条件，并且如果其中一个操作数的类型**长**，另一个操作数将转换为类型**长**。<br />-如果不满足上述三个条件，并且如果其中一个操作数的类型**无符号的 int**，另一个操作数将转换为类型**无符号的 int**。<br />-如果没有上述条件为真，将两个操作数转换为类型**int**。|

以下代码演示了上表中所述的转换规则：

```cpp
double dVal;
float fVal;
int iVal;
unsigned long ulVal;

int main() {
   // iVal converted to unsigned long
   // result of multiplication converted to double
   dVal = iVal * ulVal;

   // ulVal converted to float
   // result of addition converted to double
   dVal = ulVal + fVal;
}
```

上面的示例中的第一个语句显示了两个整数类型 `iVal` 和 `ulVal` 的相乘。 满足的条件是两个操作数属于浮动类型和一个操作数是类型**无符号的 int**。因此，另一个操作数， `iVal`，将转换为类型**无符号的 int**。结果将分配给 `dVal`。 满足的条件是一个操作数的类型是**双**; 因此，**无符号的 int**相乘的结果将转换为类型**double**。

前面的示例中的第二个语句显示了添加**float**整型类型，并`fVal`和`ulVal`。 `ulVal`变量转换为类型**float** （表中的第三个条件）。 相加的结果将转换为类型**双**（第二个表中的条件） 和分配给`dVal`。

## <a name="pointer-conversions"></a>指针转换

在赋值、初始化、比较和其他表达式中，可以转换指针。

### <a name="pointer-to-classes"></a>指向类的指针

在两种情况下，指向类的指针可转换为指向基类的指针。

第一种情况是指定的基类可访问且转换是明确的。 (请参阅[多个基类](../cpp/multiple-base-classes.md)有关不明确的基类引用的详细信息。)

基类是否可访问取决于派生中使用的继承的类型。 考虑下图中阐释的继承。

![显示基本的继承关系图&#45;类可访问性](../cpp/media/vc38xa1.gif "继承关系图显示基&#45;类可访问性") <br/>
阐明基类可访问性的继承关系图

下表显示针对该图阐释的情况的基类可访问性。

### <a name="base-class-accessibility"></a>基类可访问性

|函数的类型|派生|从<br /><br /> B * 到 A\*法律？|
|----------------------|----------------|-------------------------------------------|
|外部（非类范围）函数|Private|否|
||Protected|否|
||Public|是|
|B 成员函数（在 B 范围内）|Private|是|
||Protected|是|
||Public|是|
|C 成员函数（在 C 范围内）|Private|否|
||Protected|是|
||Public|是|

第二种情况是，在您使用显式类型转换时，指向类的指针可转换为指向基类的指针。 (请参阅[显式类型转换运算符](explicit-type-conversion-operator-parens.md)显式类型转换有关的详细信息。)

此类转换的结果是指向完全由基类描述的对象部分（即“子对象”）的指针。

以下代码定义了两个类（即 `A` 和 `B`），其中 `B` 派生自 `A`。 (继承的详细信息，请参阅[派生的类](../cpp/inheritance-cpp.md)。)然后定义 `bObject`、类型 `B` 的对象和两个指向该对象的指针（`pA` 和 `pB`）。

```cpp
// C2039 expected
class A
{
public:
    int AComponent;
    int AMemberFunc();
};

class B : public A
{
public:
    int BComponent;
    int BMemberFunc();
};
int main()
{
   B bObject;
   A *pA = &bObject;
   B *pB = &bObject;

   pA->AMemberFunc();   // OK in class A
   pB->AMemberFunc();   // OK: inherited from class A
   pA->BMemberFunc();   // Error: not in class A
}
```

指针 `pA` 的类型为 `A *`，它可解释为“指向类型 `A` 的对象的指针”。 成员`bObject``(`如`BComponent`并`BMemberFunc`) 是唯一的以键入`B`也就是无法通过`pA`。 `pA` 指针只允许访问类 `A` 中定义的对象的那些特性（成员函数和数据）。

### <a name="pointer-to-function"></a>指向函数的指针

指向一个函数的指针可以转换为类型`void *`，如果类型`void *`足够大以容纳该指针。

### <a name="pointer-to-void"></a>指向 void 的指针

指向类型**void**可以转换为指针的显式类型强制转换为任何其他类型，但仅 (不同于在 C 中)。 指向任何类型的指针可以隐式转换为指向类型的指针**void**。指向不完整类型的对象的指针可以转换为指向**void** （隐式） 和 back （显式）。 此类转换的结果与原始指针的值相等。 对象被视为是不完整的（如果已声明对象），但未提供足够多的可用信息，无法确定其大小或基类。

指向不是任何对象的指针**const**或**易失性**可以隐式转换为类型的指针`void *`。

### <a name="const-and-volatile-pointers"></a>固定和可变指针

C++不会提供一个标准转换**const**或**易失性**类型设置为不是一种**const**或**易失性**。 但是，任何类型的转换都可以用显式类型强制转换指定（包括不安全的转换）。

> [!NOTE]
>  指向成员的 C++ 指针（指向静态成员的指针除外）与常规指针不同，二者具有不同的标准转换。 指向静态成员的指针是普通指针，且与普通指针具有相同的转换。

### <a name="null-pointer-conversions"></a>null 指针转换

计算结果为零的整数常量表达式，或到某个指针类型的此类表达式强制转换，将转换为称为“null 指针”的指针。 此指针与指向任何有效对象或函数的指针比较的结果肯定不会相等（指向基对象的指针除外，此类指针可以有相同的偏移量并且仍指向不同的对象）。

在 C++ 11 [nullptr](../cpp/nullptr.md) 类型应为首选到 C 样式 null 指针。

### <a name="pointer-expression-conversions"></a>指针表达式转换

带数组类型的所有表达式都可以转换为同一类型的指针。 转换的结果是指向第一个数组元素的指针。 下面的示例演示了这样的转换：

```cpp
char szPath[_MAX_PATH]; // Array of type char.
char *pszPath = szPath; // Equals &szPath[0].
```

生成返回特定类型的函数的表达式将转换为指向返回该类型的函数的指针，以下情况除外：

- 表达式用作 address-of 运算符的操作数 ( **&** )。

- 表达式用作到 function-call 运算符的操作数。

## <a name="reference-conversions"></a>引用转换

对类的引用可在以下情况下转换为对基类的引用：

- 访问指定的基类。

- 转换是明确的。 (请参阅[多个基类](../cpp/multiple-base-classes.md)有关不明确的基类引用的详细信息。)

转换的结果为指向表示基类的子对象的指针。

## <a name="pointer-to-member"></a>指向成员的指针

指向可在赋值、初始化、比较和其他语句中转换的类成员的指针。 本节描述以下指向成员的指针转换：

## <a name="pointer-to-base-class-member"></a>指向基类成员的指针

当满足以下条件时，指向基类的成员的指针可以转换为指向派生自基类的类的成员的指针：

- 从指向派生类的指针到基类指针的反向转换可以访问。

- 派生类并非以虚拟方式从基类继承。

当左操作数是指向成员的指针时，右操作数必须是 pointer-to-member 类型或计算结果为 0 的常量表达式。 此赋值仅在以下情况下有效：

- 右操作数是指向与左操作数相同的类的成员的指针。

- 左操作数是指向以公共但不明确的方式派生自右操作数的类的成员的指针。

## <a name="integral-constant-conversions"></a>整数常量转换

计算结果为零的整数常量表达式将转换为名为“null 指针”的指针。 此指针与指向任何有效对象或函数的指针比较的结果肯定不会相等（指向基对象的指针除外，此类指针可以有相同的偏移量并且仍指向不同的对象）。

以下代码演示了指向类 `i` 中的成员 `A` 的指针的定义。 指针 `pai` 将初始化为 0，因此是 null 指针。

```cpp
class A
{
public:
int i;
};

int A::*pai = 0;

int main()
{
}
```

## <a name="see-also"></a>请参阅

[C++ 语言参考](../cpp/cpp-language-reference.md)