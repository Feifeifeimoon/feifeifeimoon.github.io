---
title: (译) 教程：泛型入门 (Go 官方教程)
abbrlink: e82052d2
date: 2022-04-12 22:15:16
updated: 2022-04-12 22:15:16
tags:
---
## 译者序

本文翻译自 **Golang** 官方教程 [**Tutorial: Getting started with generics**](https://go.dev/doc/tutorial/generics)。主要是关于 **Golang** 泛型的简单教程。
**由于译者水平有限，本文不免存在遗漏或错误之处。如有疑问，请查阅原文。**
以下是译文。
<!-- more -->

---


本教程介绍 **Go** 中泛型的基础知识。通过泛型，你可以声明和使用编写为与调用代码提供的任何一组类型一起使用的函数或类型。
在本教程中，你将声明两个简单的非泛型函数，然后在单个泛型函数中实现相同的逻辑。
你将逐步完成以下部分：

1. 为您的代码创建一个文件夹📁。
2. 添加非泛型（**non-generic**）函数。
3. 添加一个泛型函数来处理多种类型。
4. 调用泛型函数时删除类型参数（**type arguments**）。
5. 声明类型约束（**Declare a type constraint**）。


{% note info %}
注意：有关其他教程，请参阅[教程](https://go.dev/doc/tutorial/)。
注意：如果您愿意，可以使用[“Go dev 分支”模式下的 Go Playground](https://go.dev/play/?v=gotip)来编辑和运行您的程序。
{% endnote %}


## 先决条件

- **Go 1.18** 或更高版本的安装。有关安装说明，请参阅[安装**Go**](https://go.dev/doc/install)。
- 编辑代码的工具。任何文本编辑器都可以正常工作。
- 一个命令终端。**Go** 在 **Linux** 和 **Mac** 上的任何终端以及 **Windows** 中的 **PowerShell** 或 **cmd** 上都能很好地工作。

## 为你的代码创建一个文件夹📁

首先，为你要编写的代码创建一个文件夹。

1. 打开命令提示符并切换到你的主目录。

    - Linux or Mac：

      ```Bash
      $ cd
      ```

    - Windows：

      ```PowerShell
      C:\> cd %HOMEPATH%
      ```

   本教程的其余部分将显示 `$` 作为提示符。你使用的命令也可以在 **Windows** 上运行。

2. 在命令提示符下，为你的代码创建一个名为 **generics** 的目录。

   ```Bash
   $ mkdir generics
   $ cd generics
   ```

3. 创建一个模块来保存您的代码。
   运行` go mod init `命令，为其提供新代码的模块路径。

   ```Bash
   $ go mod init example/generics
   go: creating new go.mod: module example/generics
   
   ```

   > 对于生产代码，你需要指定一个更符合您自己需求的模块路径。有关更多信息，请务必查看 [Managing dependencies](https://go.dev/doc/modules/managing-dependencies)。

接下来，你将添加一些简单的代码来处理 **map**。

## 添加非泛型函数

在这一步中，你将添加两个函数，每个函数将 **map** 中的值相加并返回总数。

你需要声明两个函数而不是一个，因为你要处理两种不同类型的 **map** ：一种存储 **int64** 类型的值，另一种存储 **float64** 类型的值。

### 编写代码

1. 使用您的文本编辑器，在 **generics** 目录中创建一个名为 `main.go `的文件。你将在这个文件中编写你的 **Go** 代码。

2. 进入 `main.go`，在文件顶部，写入以下包声明。

   ```Go
   package main
   ```

   一个独立程序（与库相反）始终位于 `main` 包中。

3. 在包声明下方，写入以下两个函数声明。

   ```Go
   // SumInts adds together the values of m.
   func SumInts(m map[string]int64) int64 {
       var s int64
       for _, v := range m {
           s += v
       }
       return s
   }
   
   // SumFloats adds together the values of m.
   func SumFloats(m map[string]float64) float64 {
       var s float64
       for _, v := range m {
           s += v
       }
       return s
   }
   
   ```

   在代码中：

    - 声明两个函数来将 Map 中的的值相加并返回总数
        - `SumInts` 针对 **string → int64** 类型的 **map**
        - `SumFloats` 针对 **string → floats** 类型的 **map**

4. 在 `main.go` 顶部的包声明下方，粘贴以下的 `main` 函数来初始化两个**map**，并且用作参数来调用在上一步中声明的函数。

   ```Go
   func main() {
       // Initialize a map for the integer values
       ints := map[string]int64{
           "first":  34,
           "second": 12,
       }
   
       // Initialize a map for the float values
       floats := map[string]float64{
           "first":  35.98,
           "second": 26.99,
       }
   
       fmt.Printf("Non-Generic Sums: %v and %v\n",
           SumInts(ints),
           SumFloats(floats))
   }
   ```

   在代码中：

    - 初始化了一个 **value** 为 **int64** 类型的 **map** 和一个 **value** 为 **float64** 类型的 **map**，每个 **map** 中有两条数据
    - 调用你之前定义的两个函数来计算每个 **map** 的所有值的总和。
    - 打印结果

5. 在 `main.go` 顶部附近，就在包声明的下方，导入您需要支持你刚刚编写的代码的包。 第一行代码应如下所示：

   ```Go
   package main
   
   import "fmt"
   ```

6. 保存 `main.go`



### 运行代码

在 `main.go` 的目录中的命令行中，运行代码。

```Bash
$ go run .
Non-Generic Sums: 46 and 62.97
```


通过泛型，可以只编写一个函数来替代两个函数。接下来，你将编写一个泛型函数针对无论值是 **integer** 或者 **float** 类型的 **map** 。

## 添加一个泛型函数来处理多种类型。

在这个部分，你将添加一个泛型函数，这个函数可以接收包含 **integer** 或 **float** 的 **map**，从而有效地将刚刚编写的两个函数替换为一个函数。

要支持任一类型的值，这个函数需要一种方法来声明它支持的类型。另一方面，在调用代码时也需要一种方法来指定它是使用 **integer** 类型的 **map** 还是 **float** 类型的 **map**。

为了支持这一点，你编写的函数在其普通函数参数之外还声明类型参数（***type parameters***）。这些类型参数（***type parameters***）使这个函数成为泛型函数，使其能够处理不同类型的参数。你将使用类型参数（ ***type arguments***）和普通函数参数来调用该函数。

每个类型参数都有一个类型约束（***type constraint***），它充当类型参数的一种元类型。每个类型约束（***type constraint***）指定调用代码可用于相应类型参数的允许类型参数（***type parameters***）。

虽然类型参数的约束通常表示一组类型，但在编译时，类型参数代表单一类型——调用代码作为类型参数提供的类型。如果类型参数的约束不允许类型参数的类型，则代码将无法编译。

请记住，类型参数必须支持泛型代码对其执行的所有操作。例如，如果你的函数代码尝试对约束包括数字类型的类型参数执行`string`操作（例如索引），则代码将无法编译。

在即将编写的代码中，你将使用允许整数或浮点类型的 **constraint**。

### 编写代码

1. 在您之前添加的两个函数下方，粘贴以下泛型函数。

   ```Go
   // SumIntsOrFloats sums the values of map m. It supports both int64 and float64
   // as types for map values.
   func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
       var s V
       for _, v := range m {
           s += v
       }
       return s
   }
   ```

   在代码中：

    - 声明一个 `SumIntsOrFloats` 函数，该函数具有两个类型参数（type parameters ）（方括号内的[]），**K** 和 **V，**以及一个使用类型参数 **m **的 **map[K]V** 类型的参数。该函数返回一个类型为 **V** 的值。
    - 为 **K** 类型参数指定**comparable**的类型约束（**constraint**）。专门针对此类情况，在 **Go** 中预先声明了**comparable**的约束。它允许任何类型的值可以用作比较运算符 `==` 和 `!=` 的操作数。**Go** 要求**map**的**key**是**comparable**的。因此，必须将 **K** 声明为**comparable**，这样您就可以使用 **K** 作为**map**变量中的**key**。它还确保调用方的代码中**map**的**key**键使用允许的类型（**comparable**类型）。
    - **V** 类型参数指定一个约束是两种类型的联合：`int64 ｜float64`。使用 `|`符号指定两种类型的联合，这意味着这个约束（**constraint**）允许其中任何一种类型。编译器将允许任一类型作为调用代码中的参数。
    - 指定 **m** 参数的类型为 `map[K]V`，其中 **K** 和 **V** 是已为类型参数指定的类型

2. 在 `main.go` 中，在您已有的代码下方，粘贴以下代码。

   ```Go
   fmt.Printf("Generic Sums: %v and %v\n",
       SumIntsOrFloats[string, int64](ints),
       SumIntsOrFloats[string, float64](floats))
   ```

   在代码中：

    - 调用刚刚声明的泛型函数，传递创建的两个 **map**。
    - 指定类型参数 - 方括号中的类型名称 - 以明确应该替换您正在调用的函数中的类型参数的类型。
    - 打印函数返回的总和。

### 运行代码

在 `main.go` 的目录中的命令行中，运行代码。

```Go
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
```


为了运行代码，在每次调用中，编译器会将类型参数替换为该调用中指定的具体类型。

在调用你编写的泛型函数时，你指定了类型参数，告诉编译器使用什么类型来代替函数的类型参数。正如你将在下一节中看到的，在许多情况下，您可以省略这些类型参数，因为编译器可以推断它们。

## 调用泛型函数时删除类型参数（**type arguments**）

在本节中，你将添加泛型函数调用的修改版本，进行一些小改动以简化调用代码。你将删除在这种情况下不需要的类型参数。

当 **Go** 编译器可以推断你要使用的类型时，你可以在调用代码中省略类型参数。编译器从函数参数的类型推断类型参数。

请注意，这并不总是可能的。例如，如果你需要调用没有参数的泛型函数，则需要在函数调用中包含类型参数。

### 编写代码

- 在 `main.go` 中，在已有的代码下方，粘贴以下代码。

  ```Go
  fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
      SumIntsOrFloats(ints),
      SumIntsOrFloats(floats))
  ```

  在代码中：

- 调用泛型函数，省略类型参数。

### 运行代码

在 `main.go` 的目录中的命令行中，运行代码。

```Bash
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97

```

接下来，你将通过将整数和浮点数的并集捕获到可以重用的类型约束中来进一步简化函数，例如从其他代码中。

## 声明类型约束（**Declare a type constraint**）

在最后一部分中，你将把之前定义的约束（***constraint***）移动到它自己的 `interface`中，以便可以在多个地方重用它。
可以像声明`interface`一样声明类型约束 （***type*** ***constraint***）。约束允许任何实现这个`interface`的类型。例如，如果声明了具有三个方法的类型约束接口（***type constraint interface***），然后在泛型函数中将其与类型参数一起使用，则用于调用该函数的类型参数必须具有所有这些方法。
正如你将在本节中看到的那样，约束接口也可以引用特定类的型。

### 编写代码

1. 在 `main` 上方，紧跟 `import` 语句之后，粘贴以下代码以声明类型约束。

   ```Go
   type Number interface {
       int64 | float64
   }
   ```

   在代码中：

- 声明 `Number` 接口类型以用作类型约束（***type*** ***constraint***）。
- 在 `interface`内声明 `int64` 和 `float64` 的联合。
  本质上，你将联合从函数声明移动到新的类型约束中。这样，当你想将类型参数约束为 `int64` 或 `float64` 时，你可以使用此 `Number` 类型约束而不是写出 `int64 | float64`。（译者加：意思是如果有同样参数的泛型函数可以直接重用这个 `interface` 不用再写一遍`int64 | float64` ）。

2. 在已有的函数下方，粘贴以下通用 `SumNumbers` 函数。

   ```Go
   // SumNumbers sums the values of map m. It supports both integers
   // and floats as map values.
   func SumNumbers[K comparable, V Number](m map[K]V) V {
       var s V
       for _, v := range m {
           s += v
       }
       return s
   }
   ```

   在代码中：

- 声明一个与之前声明的泛型函数具有相同逻辑的泛型函数，但使用新的`inteface` 而不是联合作为类型约束。

3. 在` main.go` 中，在已有的代码下方，粘贴以下代码。

   ```Go
   fmt.Printf("Generic Sums with Constraint: %v and %v\n",
       SumNumbers(ints),
       SumNumbers(floats))
   ```

   在代码中：

- 使用每个**Map**调用 `SumNumbers`，打印每个 **Map** 中 **Value** 的总和。
  与上一节一样，在调用泛型函数时省略了类型参数（方括号中的类型名称）。 **Go** 编译器可以从其他参数推断类型参数。



### 运行代码

在 `main.go` 的目录中的命令行中，运行代码。

```Bash
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
Generic Sums with Constraint: 46 and 62.97
```


## 结论

做得很好！你刚刚向自己介绍了 **Go** 中的泛型。

建议的下一个主题：

- [**Go Tour**](https://tour.golang.org/welcome/1)是对 **Go** 基础知识的逐步介绍。
- 你将在[**Effective Go**](https://go.dev/doc/effective_go)和[**How to write Go code**](https://go.dev/doc/code)中找到有用的 **Go** 最佳实践。

## 完整的代码

你可以在 [Go playground](https://go.dev/play/p/apNmfVwogK0?v=gotip)上运行这个程序。只需单击“**Run**”按钮。

```Go
package main

import "fmt"

type Number interface {
    int64 | float64
}

func main() {
    // Initialize a map for the integer values
    ints := map[string]int64{
        "first": 34,
        "second": 12,
    }

    // Initialize a map for the float values
    floats := map[string]float64{
        "first": 35.98,
        "second": 26.99,
    }

    fmt.Printf("Non-Generic Sums: %v and %v\n",
        SumInts(ints),
        SumFloats(floats))

    fmt.Printf("Generic Sums: %v and %v\n",
        SumIntsOrFloats[string, int64](ints),
        SumIntsOrFloats[string, float64](floats))

    fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
        SumIntsOrFloats(ints),
        SumIntsOrFloats(floats))

    fmt.Printf("Generic Sums with Constraint: %v and %v\n",
        SumNumbers(ints),
        SumNumbers(floats))
}

// SumInts adds together the values of m.
func SumInts(m map[string]int64) int64 {
    var s int64
    for _, v := range m {
        s += v
    }
    return s
}

// SumFloats adds together the values of m.
func SumFloats(m map[string]float64) float64 {
    var s float64
    for _, v := range m {
        s += v
    }
    return s
}

// SumIntsOrFloats sums the values of map m. It supports both floats and integers
// as map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}

// SumNumbers sums the values of map m. Its supports both integers
// and floats as map values.
func SumNumbers[K comparable, V Number](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

