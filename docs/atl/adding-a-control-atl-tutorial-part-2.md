---
title: 添加控件（ATL 教程，第 2 部分）
ms.custom: get-started-article
ms.date: 09/26/2018
ms.assetid: c9575a75-1064-41f1-9697-7aada560c669
ms.openlocfilehash: 53f38d63a44328bf014f04635a24989a875ddf1e
ms.sourcegitcommit: 6cf0c67acce633b07ff31b56cebd5de3218fd733
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/24/2019
ms.locfileid: "67344330"
---
# <a name="adding-a-control-atl-tutorial-part-2"></a>添加控件（ATL 教程，第 2 部分）

在此步骤中，将控件添加到你的项目、 生成它，并在网页上对其进行测试。

## <a name="procedures"></a>过程

### <a name="to-add-an-object-to-an-atl-project"></a>若要将对象添加到 ATL 项目

1. 在“解决方案资源管理器”  中，右键单击“`Polygon`”项目。

1. 指向**外**快捷菜单，然后单击**新项**子菜单中。

    “添加新项”  对话框随即出现。 在左侧的树状结构中列出了不同的对象类别。

1. 单击**ATL**文件夹。

1. 从右侧的模板列表中，选择**ATL 控件**。 单击 **添加**。 **ATL 控件**向导将打开，并且您可以配置该控件。

1. 类型`PolyCtl`作为短名称，然后注意自动完成其他字段。 不要单击**完成**尚未，因为必须进行一些更多的更改。

**ATL 控件**向导的**名称**页包含以下字段：

|字段|内容|
|-----------|--------------|
|**短名称**|输入控件的名称。|
|**类**|C++创建实现控件的类名称。|
|**.h 文件**|创建用于包含的定义文件C++类。|
|**.cpp 文件**|创建要包含的实现的文件C++类。|
|**CoClass**|此控件的组件类的名称。|
|**Interface**|该控件将在其实现其自定义方法和属性的接口的名称。|
|**Type**|关于控件的说明。|
|**编程 ID**|可用于查找控件的 CLSID 可读名称。|

您会发现多个附加设置必须在更改**ATL 控件**向导。

### <a name="to-enable-support-for-rich-error-information-and-connection-points"></a>若要启用对丰富的错误信息和连接点

1. 单击**选项**以打开**选项**页。

1. 选择**连接点**复选框。 此选项创建 IDL 文件中对传出接口的支持。

此外可以添加接口来扩展控件的功能。

### <a name="to-extend-the-controls-functionality"></a>若要扩展控件的功能

1. 单击**接口**以打开**接口**页。

1. 选择`IProvideClassInfo2`然后单击**向上**箭头以将其移动到**支持**列表。

1. 选择`ISpecifyPropertyPages`然后单击**向上**箭头以将其移动到**支持**列表。

此外可以使控件*可插入*，这意味着它是可嵌入到支持嵌入的对象，如 Excel 或 Word 的应用程序。

### <a name="to-make-the-control-insertable"></a>若要使控件可插入

1. 单击**外观**以打开**外观**页。

1. 选择**Insertable**复选框。

由对象公开的多边形将具有一种纯色填充颜色，因此您必须添加`Fill Color`常见属性。

### <a name="to-add-a-fill-color-stock-property-and-create-the-control"></a>若要添加的填充颜色常用属性和创建控件

1. 单击**常用属性**以打开**常用属性**页。

1. 下**不支持**，向下的滚动可能常用属性列表。 选择`Fill Color`然后单击**向上**箭头以将其移动到**支持**列表。

1. 选择“完成”  。

在该向导创建控件时，将发生多个代码更改和文件相加。 会创建以下文件：

|文件|描述|
|----------|-----------------|
|PolyCtl.h|包含了大部分的实现的C++类`CPolyCtl`。|
|PolyCtl.cpp|包含的其余部分`CPolyCtl`。|
|PolyCtl.rgs|包含用来注册该控件的注册表脚本的文本文件。|
|PolyCtl.htm|包含对新创建的控件的引用的网页。|

向导还可以将以下代码更改：

- 添加`#include`语句包括 ATL 的 stdafx.h 和 stdafx.cpp 文件所需的支持控件文件。

- 更改 polygon.idl 使其包含新控件的详细信息。

- 将新控件添加到 Polygon.cpp 中的对象映射。

现在可以生成要在操作中看到它的控件。

## <a name="building-and-testing-the-control"></a>生成和测试控件

### <a name="to-build-and-test-the-control"></a>生成并测试控件

1. 上**构建**菜单上，单击**生成多边形**。

    一旦控件生成完成，右键单击中的 PolyCtl.htm**解决方案资源管理器**，然后选择**用浏览器查看**。 显示包含该控件的 HTML 网页。 应看到与标题"ATL 8.0 测试页以获取对象 PolyCtl"，并在控件文本 PolyCtl 的页面。

> [!NOTE]
> 如果控件不可见，知道某些浏览器，需要运行 ActiveX 控件的设置调整。 请参阅有关如何启用 ActiveX 控件的浏览器的文档。

> [!NOTE]
> 在完成本教程中，如果收到错误消息，无法创建 DLL 文件，关闭 PolyCtl.htm 文件和 ActiveX 控件测试容器并重新生成解决方案。 如果仍无法创建 DLL，重新启动计算机，或如果使用终端服务注销。

接下来，您将向控件添加自定义属性。

[返回到步骤 1](../atl/creating-the-project-atl-tutorial-part-1.md) &#124; [转到步骤 3](../atl/adding-a-property-to-the-control-atl-tutorial-part-3.md)

## <a name="see-also"></a>请参阅

[教程](../atl/active-template-library-atl-tutorial.md)
