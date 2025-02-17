---
title: 工具栏编辑器 (C++)
ms.date: 02/14/2019
f1_keywords:
- vc.editors.toolbar.F1
- vc.editors.toolbar
- vc.editors.newtoolbarresource
helpviewer_keywords:
- resource editors [C++], Toolbar editor
- editors, toolbars
- toolbars [C++], editing
- Toolbar editor
- toolbars [C++], creating
- Toolbar editor [C++], creating new toolbars
- Insert Resource
- bitmaps [C++], converting to toolbars
- Toolbar editor [C++], converting bitmaps
- toolbars [C++], converting bitmaps
- New Toolbar Resource dialog box [C++]
- buttons [C++], custom toolbars
- toolbar buttons [C++], editing
- buttons
- toolbar buttons [C++], creating
- Toolbar editor [C++], creating buttons
- toolbar buttons [C++], button image
- toolbar buttons [C++], creating
- toolbar buttons (in Toolbar editor)
- toolbar buttons [C++], moving
- Toolbar editor [C++], moving buttons
- Toolbar editor [C++], copying buttons
- toolbars [C++], copying buttons
- toolbar buttons [C++], copying
- toolbar buttons [C++], deleting
- Toolbar editor [C++], deleting buttons
- Toolbar editor [C++], spacing toolbar buttons
- toolbar buttons [C++], space between buttons
- toolbar controls [MFC], command ID
- toolbar buttons [C++], setting properties
- Toolbar editor [C++], toolbar button properties
- command IDs, toolbar buttons
- size, toolbar buttons
- toolbar buttons [C++], setting properties
- Toolbar editor [C++], toolbar button properties
- status bars [C++], active toolbar button text
- command IDs, toolbar buttons
- width, toolbar buttons
- tool tips [C++], adding to toolbar buttons
- "\n in tool tip"
- toolbar buttons [C++], tool tips
- buttons [C++], tool tips
- Toolbar editor [C++], creating tool tips
ms.assetid: aa9f0adf-60f6-4f79-ab05-bc330f15ec43
ms.openlocfilehash: f0faa93cd4ea1fdc2fad90a5d4d47f2feeef65e6
ms.sourcegitcommit: ecf274bcfe3a977c48745aaa243e5e731f1fdc5f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/04/2019
ms.locfileid: "66504228"
---
# <a name="toolbar-editor-c"></a>工具栏编辑器 (C++)

**工具栏编辑器**，可创建工具栏资源并将位图转换为工具栏资源。 **工具栏编辑器**使用的图形化显示来显示工具栏和按钮，非常类似于如何将介绍在完成的应用程序中。

**工具栏编辑器**窗口将显示两个按钮图像，与相同视图**图像编辑器**窗口。 拆分栏分隔两个窗格，您可以从端拆分栏拖动到另一端来更改窗格的相对大小。 活动窗格将显示选择边框和两个视图的图像的上方是使用者工具栏。

![Toolbar Editor](../mfc/media/vctoolbareditor.gif "vcToolbarEditor")<br/>
**工具栏编辑器**

**工具栏编辑器**类似于**图像编辑器**在功能和菜单项中，图形工具和位图网格两者之间是相同的。 中没有菜单命令**图像**菜单之间切换**工具栏编辑器**并**的图像编辑器**。 有关使用的详细信息**图形**工具栏中，**颜色**面板中，或**图像**菜单中，请参阅[的图像编辑器](../windows/image-editor-for-icons.md)。

可以创建一个新的工具栏中C++通过转换位图的项目。 从位图图将转换为工具栏按钮图像。 通常，位图包含在单个位图中，使用每个按钮的一个映像的多个按钮映像。 默认为 16 像素宽和图像的高度，映像可以是任何大小。 可以指定中的按钮图像的大小**新建工具栏资源**对话框中选择时**工具栏编辑器**从**图像**菜单中的同时**图像编辑器**。

**新建工具栏资源**对话框可以指定的宽度和高度为工具栏资源中要添加的按钮的C++项目。 默认值为 16 × 15 像素。

用于创建工具栏的位图具有最大宽度为 2048，因此，如果您设置**按钮宽度**到*512*，只能有四个按钮。 如果将宽度设置为*513*，只能有三个按钮。

**新建工具栏资源**对话框具有以下属性：

|属性|描述|
|---|---------------|
|**按钮宽度**|提供空间以输入你要从位图资源转换为工具栏资源的工具栏按钮的宽度。|
|**按钮高度**|提供空间以输入你要从位图资源转换为工具栏资源的工具栏按钮的高度。|

> [!NOTE]
> 图像裁剪为的宽度和高度指定，和颜色调整为使用标准工具栏颜色 （16 种颜色）。

默认情况下，新的或空白按钮显示工具栏的右侧。 您可以在编辑前移动此按钮。 当创建一个新按钮时，另一个空白按钮将出现编辑按钮右侧。 当您保存一个工具栏时，不会保存空白的按钮。

工具栏按钮具有以下属性：

|属性|描述|
|--------------|-----------------|
|**ID**|定义按钮的 ID。 下拉列表提供了常见**ID**名称。|
|**Width**|设置按钮的宽度。 建议使用 16 个像素。|
|**Height**|设置按钮的高度。 一个按钮的高度更改工具栏上的所有按钮的高度。 建议将 15 个像素。|
|**提示**|定义在状态栏中显示的消息。 添加 *\n* ，并将添加一个名称**工具提示**向工具栏按钮。 有关详细信息，请参阅[创建工具提示](../windows/creating-a-tool-tip-for-a-toolbar-button.md)。|

**宽度**并**高度**应用于所有按钮。 用于创建工具栏的位图具有最大宽度为 2048，因此，如果将按钮宽度设置为*512*只能有四个按钮，如果将宽度设置为*513*，只能有三个按钮。

## <a name="how-to"></a>操作说明

**工具栏编辑器**，可以：

### <a name="to-create-new-toolbars"></a>若要创建新工具栏

1. 在中**资源视图**，右键单击你 *.rc*文件，然后选择**添加资源**。 如果您有一个现有工具栏您 *.rc*文件中，可以右键单击**工具栏**文件夹，然后选择**插入工具栏**。

1. 在中**添加资源**对话框中，选择**工具栏**中**资源类型**列表，然后选择**新建**。

   如果一个加号 ( **+** ) 的旁边将出现**工具栏**资源类型，这意味着工具栏模板都可用。 选择加号以展开模板列表中的，选择一个模板，然后选择**新建**。

### <a name="to-convert-bitmaps-to-toolbar-resources"></a>若要将位图转换为工具栏资源

1. 打开现有的位图资源中[的图像编辑器](../windows/image-editor-for-icons.md)。 如果位图不是在你 *.rc*文件中，右键单击 *.rc*文件，然后选择**导入**，然后导航到想要添加到该位图在 *.rc*文件，然后选择**打开**。

1. 转到菜单**图像** > **工具栏编辑器**。

   **新建工具栏资源**对话框随即出现。 您可以更改的宽度和高度与位图匹配的图标图像。 随后显示的工具栏图像**工具栏编辑器**。

1. 若要完成转换，请更改该命令**ID**的按钮使用[属性窗口](/visualstudio/ide/reference/properties-window)。 键入新*ID*或选择**ID**从下拉列表。

   > [!TIP]
   > **属性**窗口中包含的图钉按钮的标题栏中，选择此项启用或禁用**自动隐藏**窗口。 若要循环通过所有工具栏按钮属性，而无需重新打开各个属性窗口，关闭**自动隐藏**关闭因此**属性**窗口保持不变。

   此外可以通过更改新的工具栏上的按钮的命令 Id[属性窗口](/visualstudio/ide/reference/properties-window)。

### <a name="to-manage-toolbar-buttons"></a>若要管理工具栏按钮

#### <a name="to-create-a-new-toolbar-button"></a>若要创建新的工具栏按钮

1. 在中[资源视图](how-to-create-a-resource-script-file.md#create-resources)展开资源文件夹 (例如， *Project1.rc*)。

1. 展开**工具栏**文件夹，然后选择工具栏上，若要编辑，然后执行下列操作之一：

   - 将 ID 分配给工具栏右侧的空白按钮。 您可以通过编辑来实现**ID**属性中的[属性窗口](/visualstudio/ide/reference/properties-window)。 例如，你可能想要提供的工具栏按钮和菜单选项相同的 ID。 在这种情况下，使用下拉列表框选择**ID**的菜单选项。

   - 选择空白按钮中的工具栏右侧**工具栏视图**窗格并开始绘制。 分配一个默认按钮命令 ID (ID_BUTTON\<n >)。

#### <a name="to-add-an-image-to-a-toolbar-as-a-button"></a>若要将图像添加到工具栏中，为的按钮

1. 在中[资源视图](how-to-create-a-resource-script-file.md#create-resources)，通过双击它打开工具栏。

1. 接下来，打开你想要添加到您的工具栏图像。

   > [!NOTE]
   > 如果您在 Visual Studio 中打开该映像，将在中打开**的图像编辑器**。 此外可以在其他图形程序中打开的图像。

1. 转到菜单**编辑** > **副本**。

1. 选择其选项卡顶部的源窗口中切换到您的工具栏。

1. 转到菜单**编辑** > **粘贴**。

   图像将显示在工具栏上，为新按钮。

#### <a name="to-move-a-toolbar-button"></a>移动工具栏按钮

在中**工具栏视图**窗格中，将你想要将移动到其新位置上工具栏按钮。

- 若要从工具栏中复制按钮，按下**Ctrl**密钥并在**工具栏视图**窗格中，将按钮拖到其新位置的工具栏上或某个位置上另一个工具栏。

- 若要删除的工具栏按钮，选择工具栏上的按钮，并将其拖出工具栏。

- 要插入或删除上一个工具栏按钮之间的空间，或者将它们拖离或向另一个在工具栏上。

|操作|步骤|
|------|------|
|若要不后跟一个空格的按钮前插入空格|将按钮拖动到右或向下直到与下一步按钮重叠大约一半。|
|要插入的按钮后跟一个空格前的空间并且要保留尾随空格|拖动按钮，直到右侧或底部边缘刚好接触下一步按钮，或只是重叠。|
|若要前插入空格跟一个空格的按钮并关闭该以下空间|将按钮拖动到右或向下直到与下一步按钮重叠大约一半。|
|若要删除在工具栏按钮之间留一个空格|一侧的空间的空间向另一侧上的按钮拖动按钮，直到它与重叠大约一半的下一步按钮。|

> [!NOTE]
> 如果没有离开所拖动的按钮的一侧空间并拖动按钮与相邻的按钮，多个中间**工具栏编辑器**插入在相反一侧所拖动的按钮的一个空格。

#### <a name="to-change-the-properties-of-a-toolbar-button"></a>若要更改工具栏按钮的属性

1. 在C++项目中，选择工具栏上的按钮。

1. 键入中的新 ID **ID**属性中的[属性窗口](/visualstudio/ide/reference/properties-window)，或使用下拉列表选择一个新**ID**。

#### <a name="to-create-a-tool-tip-for-a-toolbar-button"></a>若要创建的工具栏按钮的工具提示

1. 选择工具栏上的按钮。

1. 在中[属性窗口](/visualstudio/ide/reference/properties-window)，在**提示**字段中添加按钮进行状态栏并在该消息后的说明，添加`\n`和工具提示名称。

例如，若要查看的工具提示**打印**按钮**写字板**:

1. 打开**写字板**。

1. 鼠标指针悬停**打印**工具栏按钮，注意单词`Print`现在浮动的鼠标指针下。

1. 查看底部的状态栏**写字板**窗口，会发现它现在显示的文本`Prints the active document`。

`Print` 为工具提示名称和`Prints the active document`是按钮的状态栏的说明。

如果你想使用此效果**工具栏编辑器**，请设置**提示**属性设置为`Prints the active document\nPrint`。

## <a name="requirements"></a>要求

MFC 或 ATL

## <a name="see-also"></a>请参阅

[资源编辑器](../windows/resource-editors.md)
[菜单和其他资源](/windows/desktop/menurc/resources)<br/>
[工具栏按钮属性](../windows/toolbar-button-properties.md)<br/>
