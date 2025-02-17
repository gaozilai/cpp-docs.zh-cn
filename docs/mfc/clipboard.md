---
title: 剪贴板
ms.date: 11/04/2016
helpviewer_keywords:
- cutting and copying data
- copying data
- Clipboard
- Clipboard, programming
- transferring data
ms.assetid: a71b2824-1f14-4914-8816-54578d73ad4e
ms.openlocfilehash: 5814b2fdfc7fbcaca00037cc64dd71aa27d65cc3
ms.sourcegitcommit: ecf274bcfe3a977c48745aaa243e5e731f1fdc5f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/04/2019
ms.locfileid: "66504614"
---
# <a name="clipboard"></a>剪贴板

此系列文章介绍如何在 MFC 应用程序中实现对 Windows 剪贴板的支持。 两种方式使用 Windows 剪贴板：

- 实现标准编辑菜单命令，例如剪切、 复制和粘贴。

- 实现统一数据传输通过拖放 (OLE)。

剪贴板是一个源和目标之间的数据传输的标准 Windows 方法。 它还可能在 OLE 操作非常有用。 使用 OLE 的问世，有两种剪贴板机制在 Windows 中。 标准 Windows 剪贴板 API 仍可用，但具有已附有 OLE 数据传输机制。 OLE 统一数据传输 (UDT) 支持剪切、 复制和粘贴剪贴板和拖放。

剪贴板是共享由整个 Windows 会话，因此它不具有句柄或自己的类的系统服务。 管理类的成员函数通过剪贴板[CWnd](../mfc/reference/cwnd-class.md)。

## <a name="what-do-you-want-to-know-more-about"></a>你想要了解更多信息

- [何时使用每一剪贴板机制](../mfc/clipboard-when-to-use-each-clipboard-mechanism.md)

- [使用传统的 Windows 剪贴板 API](../mfc/clipboard-using-the-windows-clipboard.md)

- [使用 OLE 剪贴板机制](../mfc/clipboard-using-the-ole-clipboard-mechanism.md)

- [复制和粘贴数据](../mfc/clipboard-copying-and-pasting-data.md)

- [添加其他格式](../mfc/clipboard-adding-other-formats.md)

- [Windows 剪贴板](/windows/desktop/dataxchg/clipboard)

- [实施拖放 (OLE)](../mfc/drag-and-drop-ole.md)

## <a name="see-also"></a>请参阅

[用户界面元素](../mfc/user-interface-elements-mfc.md)
