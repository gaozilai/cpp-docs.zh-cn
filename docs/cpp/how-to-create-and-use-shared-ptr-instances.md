---
title: 如何：创建和使用 shared_ptr 实例
ms.custom: how-to
ms.date: 05/22/2019
ms.topic: conceptual
ms.assetid: 7d6ebb73-fa0d-4b0b-a528-bf05de96518e
ms.openlocfilehash: ac6db74122383ef8adb0f208860a6f6fba02dcc7
ms.sourcegitcommit: 8adabe177d557c74566c13145196c11cef5d10d4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/10/2019
ms.locfileid: "66821684"
---
# <a name="how-to-create-and-use-sharedptr-instances"></a>如何：创建和使用 shared_ptr 实例

`shared_ptr` 类型是 C++ 标准库中的一个智能指针，是为多个所有者可能必须管理对象在内存中的生命周期的方案设计的。 在您初始化一个 `shared_ptr` 之后，您可复制它，按值将其传入函数参数，然后将其分配给其他 `shared_ptr` 实例。 所有实例均指向同一个对象，并共享对一个“控制块”（每当新的 `shared_ptr` 添加、超出范围或重置时增加和减少引用计数）的访问权限。 当引用计数达到零时，控制块将删除内存资源和自身。

下图显示了指向一个内存位置的几个 `shared_ptr` 实例。

![共享指针图表](../cpp/media/shared_ptr.png "Shared pointer diagram")

## <a name="example-setup"></a>示例设置

以下示例均假设你已经包含所需标头并声明所需类型，如下所示：

```cpp
// shared_ptr-examples.cpp
// The following examples assume these declarations:
#include <algorithm>
#include <iostream>
#include <memory>
#include <string>
#include <vector>

struct MediaAsset
{
    virtual ~MediaAsset() = default; // make it polymorphic
};

struct Song : public MediaAsset
{
    std::wstring artist;
    std::wstring title;
    Song(const std::wstring& artist_, const std::wstring& title_) :
        artist{ artist_ }, title{ title_ } {}
};

struct Photo : public MediaAsset
{
    std::wstring date;
    std::wstring location;
    std::wstring subject;
    Photo(
        const std::wstring& date_,
        const std::wstring& location_,
        const std::wstring& subject_) :
        date{ date_ }, location{ location_ }, subject{ subject_ } {}
};

using namespace std;

int main()
{
    // The examples go here, in order:
    // Example 1
    // Example 2
    // Example 3
    // Example 4
    // Example 6
}
```

## <a name="example-1"></a>示例 1

如有可能，第一次创建内存资源时，请使用 [make_shared](../standard-library/memory-functions.md#make_shared) 函数创建 `shared_ptr`。 `make_shared` 异常安全。 它使用同一调用为控制块和资源分配内存，这会减少构造开销。 如果不使用 `make_shared`，则必须先使用显式 `new` 表达式来创建对象，然后才能将其传递到 `shared_ptr` 构造函数。 以下示例演示了同时声明和初始化 `shared_ptr` 和新对象的各种方式。

[!code-cpp[stl_smart_pointers#1](../cpp/codesnippet/CPP/how-to-create-and-use-shared-ptr-instances_1.cpp)]

## <a name="example-2"></a>示例 2

以下示例演示如何声明和初始化对其他 `shared_ptr` 已分配的对象具有共享所有权的 `shared_ptr` 实例。 假设 `sp2` 是已初始化的 `shared_ptr`。

[!code-cpp[stl_smart_pointers#2](../cpp/codesnippet/CPP/how-to-create-and-use-shared-ptr-instances_2.cpp)]

## <a name="example-3"></a>示例 3

在你使用复制元素的算法时，`shared_ptr` 在 C++ 标准库容器中很有用。 您可以将元素包装在 `shared_ptr` 中，然后将其复制到其他容器中（请记住，只要您需要，基础内存就会一直有效）。 以下示例演示如何在向量中对 `replace_copy_if` 实例使用 `shared_ptr` 算法。

[!code-cpp[stl_smart_pointers#4](../cpp/codesnippet/CPP/how-to-create-and-use-shared-ptr-instances_3.cpp)]

## <a name="example-4"></a>示例 4

您可以使用 `dynamic_pointer_cast`、`static_pointer_cast` 和 `const_pointer_cast` 来转换 `shared_ptr`。 这些函数类似于 `dynamic_cast`、`static_cast` 和 `const_cast` 运算符。 以下示例演示如何测试基类的 `shared_ptr` 向量中每个元素的派生类型，然后复制元素并显示有关它们的信息。

[!code-cpp[stl_smart_pointers#5](../cpp/codesnippet/CPP/how-to-create-and-use-shared-ptr-instances_4.cpp)]

## <a name="example-5"></a>示例 5

您可以通过下列方式将 `shared_ptr` 传递给其他函数：

- 按值传递 `shared_ptr`。 这将调用复制构造函数，增加引用计数，并使被调用方成为所有者。 此操作的开销很小，但此操作的开销可能很大，具体取决于要传递多少 `shared_ptr` 对象。 当调用方和被调用方之间的（隐式或显式）代码协定要求被调用方是所有者时，请使用此选项。

- 按引用或常量引用传递 `shared_ptr`。 在这种情况下，引用计数不会增加，并且只要调用方不超出范围，被调用方就可以访问指针。 或者，被调用方可以决定基于引用创建一个 `shared_ptr`，并成为一个共享所有者。 当调用方并不知道被调用方，或当您由于性能原因必须传递一个 `shared_ptr` 且希望避免复制操作时，请使用此选项。

- 传递基础指针或对基础对象的引用。 这使被调用方能够使用对象，但不会使其能够共享所有权或延长生存期。 如果被调用方通过原始指针创建一个 `shared_ptr`，则新的 `shared_ptr` 独立于原始的，并且不会控制基础资源。 当调用方和被调用方之间的协定明确指定调用方保留 `shared_ptr` 生存期的所有权时，请使用此选项。

- 在确定如何传递 `shared_ptr` 时，确定被调用方是否必须共享基础资源的所有权。 “所有者”是只要它需要就可以使基础资源一直有效的对象或函数。 如果调用方必须保证被调用方可以将指针的生命延长到其（函数）生存期以外，则请使用第一个选项。 如果您不关心被调用方是否延长生存期，则按引用传递并让被调用方复制或不复制它。

- 如果必须为帮助器函数提供对基础指针的访问权限，并且你知道帮助器函数将使用该指针并且将在调用函数返回前返回，则该函数不必共享基础指针的所有权。 它只需在调用方的 `shared_ptr` 的生存期内访问指针即可。 在这种情况下，按引用传递 `shared_ptr` 或传递原始指针或对基础对象的引用是安全的。 通过此方式传递将提供一个小的性能好处，并且还有助于您表达编程意图。

- 有时，例如，在一个 `std::vector<shared_ptr<T>>` 中，您可能必须将每个 `shared_ptr` 传递给 lambda 表达式主体或命名函数对象。 如果 lambda 或函数没有存储指针，则将按引用传递 `shared_ptr` 以避免调用每个元素的复制构造函数。

## <a name="example-6"></a>示例 6

以下示例演示 `shared_ptr` 如何重载各种比较运算符以在 `shared_ptr` 实例所有的内存上实现指针比较。

[!code-cpp[stl_smart_pointers#3](../cpp/codesnippet/CPP/how-to-create-and-use-shared-ptr-instances_6.cpp)]

## <a name="see-also"></a>请参阅

[智能指针（现代 C++）](../cpp/smart-pointers-modern-cpp.md)
