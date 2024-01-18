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

Now that we have a struct and a method defined, we need to create a constructor
function that will return a new instance of our `SimpleClass` struct. This method 
will be called by Godot whenever it needs to create a new instance of your object.

```go
// NewSimpleClass is a constructor that we can pass to godot.
func NewSimpleClass() godot.Class {
	return &SimpleClass{}
}
```

Now that we have a constructor function, we can register the function with
Godot, so it knows how to create a new instance of your struct and call its
methods. We can register the class by calling the `godot.AutoRegister` method 
in our `init()` function, which is a special Go function that will be executed
when our shared library is loaded, and passing our constructor function.

```go
func init() {
	// AutoRegister will register the given class constructor with Godot.
	godot.AutoRegister(NewSimpleClass)
}
```

The `godot.AutoRegister` function works by calling your constructor and inspecting
your Godot struct with reflection for all public receiver methods and struct
fields. It will then register the constructor and your struct's methods and fields
with Godot through Godot's GDNative C API.

Now we can compile our project into a shared library:

**Linux**    
`go build -v -buildmode=c-shared -o libgodot.so <your_go_library>.go`    

**Mac OS X**    
`go build -v -buildmode=c-shared -o libgodot.dylib <your_go_library>.go`    

This will create a shared library object that you can use in Godot! To learn how
to set up your library in Godot, refer to the section below.

# How do I use native scripts from the editor?

First, copy your `.so`, `.dylib`, and/or `.dll` library that you compiled into
your project folder.

Create a new `GDNativeLibrary` resource by clicking the new icon in the inspector.
A `GDNativeLibrary` resource is a platform-agnostic abstraction of our native library. 
With it, it allows us to specify different shared library object files for different
platforms, such as `.so` for Linux platforms, `.dylib` for Mac, and `.dll` for Windows.
![](images/tutorial01.png)

Select `GDNativeLibrary` from the list of resource types.
![](images/tutorial02.png)

Select the folder icon next to the platform that you want to support in the inspector.
For Linux platforms, you'll want to select a `.so` file, `.dylib` for Mac, and `.dll` for
Windows. You can add each shared library file for each platform you want to support.

![](images/tutorial03.png)

Select your shared library file from your project directory.

![](images/tutorial04.png)

Click the save icon and then click "Save As.."

![](images/tutorial05.png)    
![](images/tutorial06.png)    

Save your GDNativeLibrary resource to your project directory. This file simply
contains the paths to the shared library objects for each platform you want to
support.

![](images/tutorial07.png)    

Now create a new node that you want to attach your Go library to.

![](images/tutorial08.png)    
![](images/tutorial09.png)    

Click the add script icon to attach your Go library to the node.

![](images/tutorial10.png)    

Select "NativeScript" as the script language, and enter the name of the struct 
that you registered in your Go library that you would like to be attached to this
node. You should also select "Built-in Script", so this setting is built in to
the scene.

![](images/tutorial11.png)    

With your node selected, you can now attach our GDNativeLibrary resource that we
created earlier to this node.

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
