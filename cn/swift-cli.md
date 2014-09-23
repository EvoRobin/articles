# Swift 命令行工具

Swift 的 REPL (Read-Eval-Print Loop) 环境可以让我们使用 Swift 进行简单的交互式编程。也就是说每输入一句语句就立即执行和输出。这在很多解释型的语言中是很常见的，非常适合用来对语言的特性进行学习。

要启动 REPL 环境，就要使用 Swift 的命令行工具，它是以 `xcrun` 命令的参数形式存在的。首先我们需要确认使用的 Xcode 版本是否是 6.1 或者以上，如果不是的话，可以在 Xcode 设置里 Locations 中的 Command Line Tools 一项中进行选择。然后我们就可以在命令行中输入 `xcrun swift` 来启动 REPL 环境了。

<div class="ui orange segment">
<p>在 Xcode 6.0 中是没有包含 OSX 10.10 的 SDK 的，而命令行执行 Swift REPL 是需要最新版的 OSX SDK 的。如果我们使用 Xcode 6.0 的版本的话，会出现 the SDK 'MacOSX10.9.sdk' does not support Swift 的错误。</p>
</div>

要指出的是，REPL 环境只是**表现得**像是即时的解释执行，但是其实质还是每次输入代码后进行编译再运行。这就限制了我们不太可能在 REPL 环境中做很复杂的事情。

另一个用法是直接将一个 `.swift` 文件作为命令行工具的输入，这样里面的代码也会被自动地编译和执行。我们甚至还可以在 `.swift` 文件最上面加上命令行工具的路径，然后将文件权限改为可执行，之后就可以直接执行这个 `.swift` 文件了：

    // hello.swift
    #!/usr/bin/env xcrun swift
    println("hello")

    // Terminal
    > chmod 755 hello.swift
    > ./hello.swift

    // 输出：
    hello

这些特性与其他的解释性语言表现得完全一样。

相对于直接用 `swift` 命令执行，Swift 命令行工具的另一个常用的地方是直接脱离 Xcode 环境进行编译和生成可执行的二进制文件。我们可以使用 `swiftc` 来进行编译，比如下面的例子：

    // MyClass.swift
    class MyClass {
        let name = "XiaoMing"
        func hello() {
            println("Hello \(name)")
        }
    }

    // main.swift
    let object = MyClass()
    object.hello()


    > xcrun swiftc MyClass.swift main.swift

将生成一个名叫 `main` 的可执行文件。运行之：

    > ./main
    // 输出：
    // Hello XiaoMing

利用这个方法，我们就可以用 Swift 写出一些命令行的程序了。

最后想说明的一个 Swift 命令行工具的使用案例是生成汇编级别的代码。有时候我们想要确认经过优化后的汇编代码实际上做了些什么。`swiftc` 提供了参数来生成 `asm` 级别的汇编代码：

    swiftc -O hello.swift > hello.asm

Swift 的命令行工具还有不少强大的功能，对此感兴趣的读者不妨使用 `xcrun swift --help` 以及 `xcrun swiftc --help` 来查看具体还有哪些参数可以使用。


