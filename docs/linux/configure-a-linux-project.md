---
title: 在 Visual Studio 中配置 C++ Linux 项目
ms.date: 06/07/2019
ms.assetid: 4d7c6adf-54b9-4b23-bd23-5de0c825b768
ms.openlocfilehash: 5acd9edeef8f09f86c394c39939d8408821dd691
ms.sourcegitcommit: 8adabe177d557c74566c13145196c11cef5d10d4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/10/2019
ms.locfileid: "66821565"
---
# <a name="configure-a-linux-project"></a>配置 Linux 项目

::: moniker range="vs-2015"

Linux 支持在 Visual Studio 2017 及更高版本中提供。

::: moniker-end

本主题介绍如何配置 C++ Linux 项目，如[在 Visual Studio 中新建 C++ Linux 项目](create-a-new-linux-project.md)中所述。 对于 CMake Linux 项目，请参阅[配置 Linux CMake 项目](cmake-linux-project.md)。 

可以将 Linux 项目配置为以物理 Linux 计算机、虚拟机或[适用于 Linux 的 Windows 子系统](/windows/wsl/about) (WSL) 为目标。 

::: moniker range="vs-2019"

Visual Studio 2019 版本 16.1： 

- 当以 WSL 为目标时，可以避免在以远程 Linux 系统为目标时生成和 IntelliSense 所需的复制操作。

- 可以指定用于生成和调试的独立 Linux 目标。

::: moniker-end

## <a name="general-settings"></a>常规设置

若要查看配置选项，请选择“项目”>“属性”菜单，或在“解决方案资源管理器”中右键单击相应项目，然后从上下文菜单中选择“属性”    。 显示“常规”设置  。

![常规配置](media/settings_general.png)

默认情况下，可执行文件 (.out) 是使用该工具生成的。 若要生成静态或动态库，或使用现有生成文件，请使用“配置类型”设置  。

有关属性页中设置的详细信息，请参阅 [Linux 项目属性页参考](prop-pages-linux.md)。

## <a name="remote-settings"></a>远程设置

要更改有关远程 Linux 计算机的设置，请配置[常规](prop-pages/general-linux.md)下显示的远程设置。

- 要指定远程目标 Linux 计算机，请使用“远程生成计算机”条目  。 这可以让你选择之前创建的连接之一。 若要创建新条目，请参阅[连接到远程 Linux 计算机](connect-to-your-remote-linux-computer.md)部分。

   ![生成计算机](media/remote-build-machine-vs2019.png)

   ::: moniker range="vs-2019"

   Visual Studio 2019 版本 16.1：  若要以适用于 Linux 的 Windows 子系统为目标，请单击向下箭头找到“平台工具集”，然后选择“WSL_1_0”   。 其他远程选项将消失，默认 WSL shell 的路径将显示在其位置：

   ![WSL 生成计算机](media/wsl-remote-vs2019.png)

   如果进行了并行 WSL 安装，则可在此处指定其他路径。 有关管理多个发行版的详细信息，请参阅[管理和配置适用于 Linux 的 Windows 子系统](/windows/wsl/wsl-config#set-a-default-distribution)。

   可在“配置属性”>“调试”页上指定用于调试的其他目标   。

   ::: moniker-end

- “远程生成根目录”确定在远程 Linux 计算机上生成项目的根位置  。 除非更改，否则该位置默认为 **~/projects**。

- “远程生成项目目录”是在远程 Linux 计算机上生成此特定项目的位置  。 该位置默认为 **$(RemoteRootDir)/$(ProjectName)** ，它将扩展到以当前项目命名的目录，在上面设置的根目录下。

> [!NOTE]
> 要更改默认的 C 和 C++ 编译器，或者用于生成项目的链接器和存档程序，请使用“C/C++”>“常规”部分和“链接器”>“常规”部分中的相应条目   。 例如，可以指定某个版本的 GCC 或 Clang。 有关详细信息，请参阅 [C/C+ + 属性 (Linux C++)](prop-pages/c-cpp-linux.md) 和[链接器属性 (Linux C++)](prop-pages/linker-linux.md)。

## <a name="copy-sources-remote-systems-only"></a>复制源（仅限远程系统）

::: moniker range="vs-2019"

在以 WSL 为目标时，本部分不适用。

::: moniker-end

在远程系统上进行生成时，开发电脑上的源文件将复制到 Linux 计算机并在该计算机上进行编译。 默认情况下，Visual Studio 项目中的所有源文件都将复制到上述设置中设置的位置。 但是，也可以将其他源文件添加到此列表，或完全关闭复制源文件，这是生成文件项目的默认值。

- “**要复制的源**”确定将哪些源文件复制到远程计算机。 在默认情况下， **\@(SourcesToCopyRemotely)** 默认为项目中的所有源代码文件，但不包含任何资产/资源文件，如映像。

- 可以打开和关闭“**复制源**”，以启用和禁用将源文件复制到远程计算机操作。

- “**要复制的其他源**”允许添加将复制到远程系统的其他源文件。 可以指定以分号分隔的列表，也可以使用 **:=** 语法指定要使用的本地和远程名称：

`C:\Projects\ConsoleApplication1\MyFile.cpp:=~/projects/ConsoleApplication1/ADifferentName.cpp;C:\Projects\ConsoleApplication1\MyFile2.cpp:=~/projects/ConsoleApplication1/ADifferentName2.cpp;`

## <a name="build-events"></a>生成事件

由于所有编译都是在远程计算机（或 WSL）上进行的，因此，已将数个其他生成事件添加到“项目属性”中的“生成事件”部分。 它们是“远程预生成事件”、“远程预链接事件”和“远程后期生成事件”，将在此过程中的各个步骤之前或之后在远程计算机上发生    。

![生成事件](media/settings_buildevents.png)

## <a name="remote_intellisense"></a> 远程系统上标头的 IntelliSense

::: moniker range="vs-2019"

在以 WSL 为目标时，本部分不适用。

::: moniker-end

在连接管理器中添加新连接时，Visual Studio 会自动为远程系统上的编译器检测包含目录  。 随后 Visual Studio 压缩这些文件压缩，并将其复制到本地 Windows 计算机上的目录中。 此后，每当在 Visual Studio 或 CMake 项目中使用该连接时，都会使用这些目录中的标头来提供 IntelliSense。

此功能取决于 Linux 计算机是否安装了 zip。 可使用此 apt-get 命令安装 zip：

```cmd
apt install zip
```

若要管理标头缓存，导航到“工具”>“选项”，依次选择“跨平台”>“连接管理器”>“远程标头 IntelliSense 管理器”  。 若要在更改 Linux 计算机后更新标头缓存，请选择远程连接，然后选择“更新”  。 选择“删除”，来删除标头但不删除连接本身  。 选择“浏览”，打开“文件资源管理器”中的本地目录   。 将此文件夹视为只读文件夹。 要下载在 Visual Studio 2017 版本 15.3 之前创建的现有连接的标头，请选择该连接，然后选择“下载”  。

::: moniker range="vs-2017"

![远程标头 IntelliSense](media/remote-header-intellisense.png)

::: moniker-end

::: moniker range="vs-2019"

![远程标头 IntelliSense](media/connection-manager-vs2019.png)

可以启用日志记录以帮助解决问题：

![远程日志记录](media/remote-logging-vs2019.png)

::: moniker-end

## <a name="see-also"></a>请参阅

[设置编译器和生成属性](../build/working-with-project-properties.md)<br/>
[C++ 常规属性 (Linux C++)](../linux/prop-pages/general-linux.md)<br/>
[VC++ 目录 (Linux C++)](../linux/prop-pages/directories-linux.md)<br/>
[复制源项目属性 (Linux C++)](../linux/prop-pages/copy-sources-project.md)<br/>
[生成事件属性 (Linux C++)](../linux/prop-pages/build-events-linux.md)
