---
title: 在 Visual Studio 中配置 Linux CMake 项目
description: 如何在 Visual Studio 中配置、编辑和编译 Linux CMake 项目
ms.date: 06/07/2019
ms.assetid: f8707b32-f90d-494d-ae0b-1d44425fdc25
ms.openlocfilehash: e0a4abb7fe62880af12277d5c5c474d6ec4e0202
ms.sourcegitcommit: 8adabe177d557c74566c13145196c11cef5d10d4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/10/2019
ms.locfileid: "66821670"
---
# <a name="configure-a-linux-cmake-project"></a>配置 Linux CMake 项目

::: moniker range="vs-2015"

Linux 支持在 Visual Studio 2017 及更高版本中提供。

::: moniker-end

打开包含 CMake 项目的文件夹时，Visual Studio 会使用 CMake 生成的元数据来自动配置 IntelliSense 和生成。 本地配置和调试设置存储在 JSON 文件中，可选择与使用 Visual Studio 的其他人共享这些文件。 

Visual Studio 不会修改 CMakeLists.txt 文件或原始 CMake 缓存，因此处理同一个项目的其他人可继续使用他们已在使用的任何工具。

有关 Visual Studio 中的 CMake 支持的常规信息，请参阅 [CMake Tools for Visual Studio](../build/cmake-projects-in-visual-studio.md)。 在继续此处的操作前，请先阅读相关内容。

## <a name="before-you-begin"></a>在开始之前

首先，请确保已安装“使用 C++ 的 Linux 开发”工作负载，包括 CMake 组件  。 请参阅[在 Visual Studio 中安装 C++ Linux 工作负载](download-install-and-setup-the-linux-development-workload.md)。 

在 Linux 系统上，请确保已安装以下项： 

- gcc
- gdb
- rsync
- zip 

::: moniker range="vs-2019"

Linux 对 CMake 项目的支持要求在目标计算机上安装最新版本的 CMake。 发行版的默认包管理器提供的版本通常不够新，不足以支持 Visual Studio 所需的所有功能。 Visual Studio 2019 会检测 Linux 系统上是否安装了最新版本的 CMake。 如果未找到，Visual Studio 将在编辑器窗格顶部显示一个信息栏，用于安装此最新版本。

::: moniker-end

::: moniker range="vs-2017"

使用 Visual Studio 中的 CMake 支持需要 CMake 3.8 中引入的服务器模式支持。 对于 Microsoft 提供的 CMake 变体，请在 [https://github.com/Microsoft/CMake/releases](https://github.com/Microsoft/CMake/releases) 中下载最新的预生成二进制文件。

二进制文件将安装到 `~/.vs/cmake`。 部署二进制文件后，项目将自动再生成。 请注意，如果由 `CMakeSettings.json` 中的 `cmakeExecutable` 字段指定的 CMake 无效（不存在或是不受支持的版本）且预生成二进制文件存在，则 Visual Studio 将忽略 `cmakeExecutable` 并使用预生成二进制文件。

:::moniker-end

## <a name="open-a-folder"></a>打开文件夹

首先，依次选择主菜单中的“文件”   > “打开”   > “文件夹”  ，或在命令行中键入“`devenv.exe <foldername>`”。 打开的文件夹中应该有一个 CMakeLists.txt 文件，以及你的源代码。
以下示例显示了一个简单的 CMakeLists.txt 文件和 .cpp 文件：

```cpp
// hello.cpp

#include <iostream>

int main(int argc, char* argv[])
{
    std::cout << "Hello from Linux CMake" << std::endl;
}
```

CMakeLists.txt：

```cmd
project (hello-cmake)
add_executable(hello-cmake hello.cpp)
```

## <a name="choose-a-linux-target"></a>选择 Linux 目标

只要打开文件夹，Visual Studio 就会分析 CMakeLists.txt 文件，并指定“x86 调试”  以定目标到 Windows。 要以远程 Linux 系统为目标，请将项目设置更改为“Linux 调试”或“Linux 发布”   。 

::: moniker range="vs-2019"

要以适用于 Linux 的 Windows 子系统为目标，如果使用 GCC，请选择“WSL 调试”或“WSL 发布”；如果使用 Clang/LLVM 工具集，则选择 Clang 变体   。 

Visual Studio 2019 版本 16.1：在以 WSL 为目标时，无需复制或源或标头，因为 Linux 上的编译器可以直接访问源文件所在的 Windows 文件系统，并且 Visual Studio 同样可以直接访问 Linux 头文件  。

::: moniker-end

对于远程目标，Visual Studio 默认选择“工具” > “选项” > “跨平台” > “连接管理器”下列表中的第一个远程系统     。 如果未找到任何远程连接，那么系统会提示你进行创建。 有关详细信息，请参阅[连接到远程 Linux 计算机](connect-to-your-remote-linux-computer.md)。

如果指定远程 Linux 目标，则会将源复制到远程系统。

选择目标后，CMake 会在 Linux 系统上自动运行，以为项目生成 CMake 缓存。 

![在 Linux 上生成 CMake 缓存](media/cmake-linux-1.png "Generate the CMake cache on Linux")

为了向远程 Linux 系统上的标头提供 IntelliSense 支持，Visual Studio 会自动将这些标头从 Linux 计算机复制到本地 Windows 计算机上的目录中。 有关详细信息，请参阅[远程标头的 IntelliSense](configure-a-linux-project.md#remote_intellisense)。

## <a name="debug-the-project"></a>调试项目

若要在指定调试目标系统上调试代码，请设置断点，并在项目设置旁边的工具栏菜单中选择“CMake 目标”作为启动项，再选择工具栏中的“&#x23f5; 开始”，或按 F5  。

若要自定义程序的命令行参数，请右键单击解决方案资源管理器中的可执行文件，并选择“调试和启动设置”   。 这会打开或创建 launch.vs.json 配置文件，其中包含程序信息。 若要指定其他参数，请以 `args` JSON 数组形式添加它们。 有关详细信息，请参阅 [C++ 的“打开文件夹”项目](../build/configure-cmake-debugging-sessions.md)和[配置 CMake 调试会话](../build/open-folder-projects-cpp.md)。

## <a name="configure-cmake-settings-for-linux"></a>配置适用于 Linux 的 CMake 设置

CMake Linux 项目中的 CMakeSettings.json 文件可指定在[自定义 CMake 设置](../build/customize-cmake-settings.md)中列出的所有属性，以及控制远程 Linux 计算机上的生成设置的其他属性。 

::: moniker range="vs-2019"

若要在 Visual Studio 2019 中更改默认 CMake 设置，请在主工具栏中打开“配置”下拉列表，然后选择“管理配置”   。 

![CMake 管理配置](../build/media/vs2019-cmake-manage-configurations.png "CMake 配置下拉列表")

此操作将打开“CMake 设置编辑器”，可使用它来编辑根项目文件夹中的 `CMakeSettings.json` 文件  。 也可以通过单击编辑器中的“编辑 JSON”按钮来直接打开文件  。 有关详细信息，请参阅[自定义 CMake 设置](../build/customize-cmake-settings.md)。

::: moniker-end

::: moniker range="vs-2017"

若要在 Visual Studio 2017 中更改默认 CMake 设置，请从主菜单依次选择“CMake”|“更改 CMake 设置”|“CMakeLists.txt”，或右键单击“解决方案资源管理器”中的“CMakeSettings.txt”，然后选择“更改 CMake 设置”    。 然后，visual Studio 会在根项目文件夹中创建一个新的 `CMakeSettings.json` 文件。 可使用“CMake 设置”编辑器打开文件或直接修改该文件  。 有关详细信息，请参阅[自定义 CMake 设置](../build/customize-cmake-settings.md)。

::: moniker-end

以下示例显示了基于上述代码示例的 Linux 调试的默认配置：

```json
{
      "name": "Linux-Debug",
      "generator": "Unix Makefiles",
      "remoteMachineName": "${defaultRemoteMachineName}",
      "configurationType": "Debug",
      "remoteCMakeListsRoot": "/var/tmp/src/${workspaceHash}/${name}",
      "cmakeExecutable": "/usr/local/bin/cmake",
      "buildRoot": "${env.LOCALAPPDATA}\\CMakeBuilds\\${workspaceHash}\\build\\${name}",
      "installRoot": "${env.LOCALAPPDATA}\\CMakeBuilds\\${workspaceHash}\\install\\${name}",
      "remoteBuildRoot": "/var/tmp/build/${workspaceHash}/build/${name}",
      "remoteInstallRoot": "/var/tmp/build/${workspaceHash}/install/${name}",
      "remoteCopySources": true,
      "remoteCopySourcesOutputVerbosity": "Normal",
      "remoteCopySourcesConcurrentCopies": "10",
      "remoteCopySourcesMethod": "rsync",
      "remoteCopySourcesExclusionList": [".vs", ".git"],
      "rsyncCommandArgs" : "-t --delete --delete-excluded",
      "remoteCopyBuildOutput" : "false",
      "cmakeCommandArgs": "",
      "buildCommandArgs": "",
      "ctestCommandArgs": "",
      "inheritEnvironments": [ "linux-x64" ]
}
```
下表概述了这些设置：

|设置|说明|
|-----------|-----------------|
|`name`|可按需设置此值。|
|`remoteMachineName`|指定了目标远程系统（以防存在多个系统）。 此字段启用了 IntelliSense，可帮助你选择正确的系统。|
|`remoteCMakeListsRoot`|指定将项目源复制到远程系统上的何处。|
|`remoteBuildRoot`|指定远程系统上生成“生成输出”的位置。 还会在本地将该输出复制到由 `buildRoot` 指定的位置。|
|`remoteInstallRoot` 和 `installRoot`| 与 `remoteBuildRoot` 和 `buildRoot` 类似，只不过在执行 CMake 安装时应用它们。|
|`remoteCopySources`|指定是否将本地源复制到远程计算机。 如果具有大量文件且已自行同步源，则可能要将其设置为 false。|
|`remoteCopyOutputVerbosity`| 指定复制步骤的详细级别，以防需要诊断错误。|
|`remoteCopySourcesConcurrentCopies`| 指定要生成多少进程来执行复制。|
|`remoteCopySourcesMethod`| 可以为 `rsync` 或 `sftp`。|
|`remoteCopySourcesExclusionList`| 指定不想复制到远程计算机的文件。|
|`rsyncCommandArgs`|控制复制的 rsync 方法。|
|`remoteCopyBuildOutput`| 控制是否要将远程生成输出复制到本地生成文件夹。|

可使用这些可选设置进行更多控制：

```json
{
      "remotePrebuildCommand": "",
      "remotePreGenerateCommand": "",
      "remotePostbuildCommand": "",
}
```

这些选项可用于生成前后以及 CMake 生成前在 Linux 系统中运行命令。 值可以是远程系统上的任何有效命令。 输出通过管道传递回 Visual Studio。

::: moniker range="vs-2019"

在 Visual Studio 2019 中，可在“CMake 设置编辑器”中编辑所有这些设置  。

::: moniker-end

## <a name="see-also"></a>请参阅

[使用项目属性](../build/working-with-project-properties.md)<br/>
[Visual Studio 中的 CMake 项目](../build/cmake-projects-in-visual-studio.md)<br/>
[连接到远程 Linux 计算机](connect-to-your-remote-linux-computer.md)<br/>
[自定义 CMake 设置](../build/customize-cmake-settings.md)<br/>
[配置 CMake 调试会话](../build/configure-cmake-debugging-sessions.md)<br/>
[部署、运行和调试 Linux 项目](deploy-run-and-debug-your-linux-project.md)<br/>
[CMake 预定义配置引用](../build/cmake-predefined-configuration-reference.md)<br/>
