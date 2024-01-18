[![Build Status](https://travis-ci.org/ShadowApex/godot-go.svg?branch=master)](https://travis-ci.org/ShadowApex/godot-go)
[![Go Report Card](https://goreportcard.com/badge/github.com/shadowapex/godot-go)](https://goreportcard.com/report/github.com/shadowapex/godot-go)
[![GoDoc](https://godoc.org/github.com/ShadowApex/goquery?status.png)](https://godoc.org/github.com/ShadowApex/godot-go/godot)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/ShadowApex/godot-go/master/LICENSE)

![Godot-Go logo](/logo.png)

Godot-Go
========
Go语言绑定[Godot引擎](https://godotengine.org/)'s [GDNative API](https://github.com/GodotNativeTools/godot_headers).

**注意：** 这些绑定目前仍在开发中。不是所有的设计， 实施或文档是最终的。欢迎提出意见/建议（参见 GitHub 问题）。 API 可能会发生变化。

用法
-----
Godot 引擎可以使用 GDNative 模块与用 Go 编写的代码进行交互。
它的工作原理是使用 Go 将代码编译为共享库 (`.so`, `.dylib`, `.dll`),
Godot引擎可以加载它来调用Go方法。

godot-go库提供了一个名为`godot. AutoRegister`的方法。
它将允许你向Godot暴露一个遵循`godot.Class`接口的Go结构体加载共享库时使用的引擎。
它还提供了对所有的可用的Godot引擎类及其方法，因此您可以调用Godot-Go代码中的函数。

# 设置
`go get github.com/shadowapex/godot-go/godot`

# 构建
当您准备将代码构建为可以导入Godot的动态库时，请根据您的平台使用下面的说明。

## Linux
`go build -v -buildmode=c-shared -o libgodot.so <your_go_library>.go`    

## Mac OS X
`go build -v -buildmode=c-shared -o libgodot.dylib <your_go_library>.go`    

# 教程
要编写可在Godot中使用的Go共享库，您首先需要创建一个`.go`文件将作为库的入口点。
这个文件*必须*将`package main`与`main()`和`init()`函数一起作为包的名称定义的。
`main()`函数将是空的，但需要编译它作为一个共享库:

```go
package main

func init() {
}

func main() {
}
```

设置好这些之后，我们可以定义一个我们想要可用的新结构体Godot。
在我们的结构体中，我们可以嵌入任何可用的Godot类之一，以便它实现`godot.Class`接口。
请注意，不支持嵌入多个Godot结构体。

```go
// SimpleClass is a simple go struct that can be attached to a Godot Node2D object.
type SimpleClass struct {
	godot.Node2D
}
```

一旦我们定义了结构体，我们现在可以将方法接收器附加到我们的结构体。
附加到这个结构体的所有方法都可以从Godot调用，前提是它们获取和/或返回内置或godot类型。
让我们继续定义一个方法`X_ready`方法的接收器，当我们的节点进入时，Godot将调用它加载场景。

```go
// X_ready is called as soon as the node enters the scene.
func (h *SimpleClass) X_ready() {
	godot.Log.Warning("Hello World!")
}
```

现在我们定义了一个结构体和一个方法，我们需要创建一个构造函数这个函数将返回一个`SimpleClass`结构体的新实例。
这个Godot方法将在需要创建对象的新实例时调用。

```go
// NewSimpleClass is a constructor that we can pass to godot.
func NewSimpleClass() godot.Class {
	return &SimpleClass{}
}
```

现在我们有了一个构造函数，我们可以注册这个函数Godot，所以它知道如何创建结构体的新实例并调用它方法。
我们可以通过调用 `godot.AutoRegister` 来注册这个类的方法在我们的`init()`函数中，
这是一个将被执行的特殊的Go函数，当我们的共享库加载并传递我们的构造函数时。

```go
func init() {
	// AutoRegister will register the given class constructor with Godot.
	godot.AutoRegister(NewSimpleClass)
}
```

`godot.AutoRegister`函数通过调用构造函数并进行检查来工作你的Godot结构体具有所有公共接收器方法和结构体的反射字段。
然后，它将注册构造函数以及结构体的方法和字段通过Godot的GDNative C API使用Godot。

现在我们可以将项目编译为共享库了:

**Linux**    
`go build -v -buildmode=c-shared -o libgodot.so <your_go_library>.go`    

**Mac OS X**    
`go build -v -buildmode=c-shared -o libgodot.dylib <your_go_library>.go`    

这将创建一个可以在Godot中使用的共享库对象!
学习如何要在Godot中设置库，请参阅下面的部分。

# 如何使用编辑器中的原生脚本?

首先，复制你的`.so`, `.dylib`和/或编译成的`.dll`库到你的项目文件夹。

通过点击inspector中的新图标创建一个新的`GDNativeLibrary`资源。
`GDNativeLibrary`资源是我们本地库的平台无关抽象。
使用它，它允许我们为不同的共享库对象文件指定不同的文件平台，
例如在Linux平台是`.so`。在Mac上是`.dylib`，在Windows上是`.dll`。
![](images/tutorial01.png)

从资源类型列表中选择`gdnativellibrary`。
![](images/tutorial02.png)

在检查器中选择你想要支持的平台旁边的文件夹图标。
对于Linux平台，你需要选择一个`.so`文件，`.dylib`用于Mac， `.dll`用于Windows。
您可以为想要支持的每个平台添加每个共享库文件。

![](images/tutorial03.png)

从项目目录中选择共享库文件。

![](images/tutorial04.png)

点击保存图标，然后点击“另存为..”

![](images/tutorial05.png)    
![](images/tutorial06.png)    

将您的GDNativeLibrary资源保存到您的项目目录。
这个文件很简单包含您想要的每个平台的共享库对象的路径支持。

![](images/tutorial07.png)    

现在创建一个要将Go库附加到其上的新节点。

![](images/tutorial08.png)    
![](images/tutorial09.png)    

单击add script图标将Go库附加到节点。

![](images/tutorial10.png)    

选择"NativeScript"作为脚本语言，并输入结构体的名称你在Go库中注册的，你想要附加到这个节点。
你还应该选择"Built-in Script"（内置脚本），这样这个设置就内置到场景。

![](images/tutorial11.png)    

选择好你的节点后，你现在可以连接我们的GDNativeLibrary资源在此节点之前创建。

![](images/tutorial12.png)    
![](images/tutorial13.png)    

Attributions
------------
The Go gopher was designed by [Renee French](http://reneefrench.blogspot.com/).

The logo used above was based on the image by [Takuya Ueda](https://twitter.com/tenntenn). 
Licensed under the Creative Commons 3.0 Attributions license. 
Available unmodified from: <https://github.com/golang-samples/gopher-vector>

The logo used above was also based on the image by [Andrea Calabró](https://commons.wikimedia.org/wiki/File:Godot_logo.svg)
Licensed under the Creative Commons Attribution License version 3.0 [CC-BY 3.0](https://creativecommons.org/licenses/by/3.0/legalcode)

License
-------
MIT - <https://opensource.org/licenses/MIT>  

Links
-----
GitHub repository - <https://github.com/shadowapex/godot-go>  
GDNative repository - <https://github.com/GodotNativeTools/godot_headers>  

Godot Engine - <https://godotengine.org>  
Go programming language - <https://golang.org>  
