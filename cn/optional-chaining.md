# Optional Chaining

使用 Optional Chaining 可以让我们摆脱很多不必要的判断和取值，但是在使用的时候需要小心陷阱。

因为 Optional Chaining 是随时都可能提前返回 `nil` 的，所以使用 Optional Chaining 所得到的东西其实都是 Optional 的。比如有下面的一段代码：

    class Toy {
        let name: String
        init(name: String) {
            self.name = name
        }
    }

    class Pet {
        var toy: Toy?
    }

    class Child {
        var pet: Pet?
    }

在实际使用中，我们想要知道小明的宠物的玩具的名字的时候，可以通过下面的 Optional Chaining 拿到：

    let toyName = xiaoming.pet?.toy?.name

注意虽然我们最后访问的是 `name`，并且在 `Toy` 的定义中 `name` 是被定义为一个确定的 `String` 而非 Optional 的，但是我们拿到的 `toyName` 其实还是一个 `String?` 的类型。这是由于在 Optional Chaining 中我们在任意一个 `?.` 的时候都可能遇到 `nil` 而提前返回，这个时候当然就只能拿到 `nil` 了。

在实际的使用中，我们更多写的可能是使用 Optional Binding 来直接取值的这样的代码：

    if let toyName = xiaoming.pet?.toy?.name {
        // 太好了，小明既有宠物，而且宠物还正好有个玩具
    }

可能单独拿出来看会很清楚，但是只要稍微和其他特性结合一下，事情就会变得复杂起来。来看看下面的例子：
    
    extension Toy {
        func play() {
            //...
        }
    }
    
我们为 `Toy` 定义了一个扩展，以及一个玩玩具的 `play()` 方法。还是拿小明举例子，要是有玩具的话，就玩之：

    xiaoming.pet?.toy?.play()

除了小明也许我们还有小红小李小张等等..在这种时候我们会想要把这一串调用抽象出来，做一个闭包方便使用。传入一个 `Child` 对象，如果小朋友有宠物并且宠物有玩具的话，就去玩。于是很可能你会写出这样的代码：


<div class="ui error message">
  <div class="header">
  <i class="fa fa-bug"></i>
    这是错误代码
  </div>
  <pre><code>let playClosure = {(child: Child) -> () in child.pet?.toy?.play()}</code></pre>
</div>

你会发现这么表意清晰的代码居然无法编译！

问题在于对于 `play()` 的调用上。定义的时候我们没有写 `play()` 的返回，这表示这个方法返回 `Void` (或者写作一对小括号 `()`，它们的等价的)。但是正如上所说，经过 Optional Chaining 以后我们得到的是一个 Optional 的结果。也就是说，我们最后得到的应该是这样一个 closure：

    let playClosure = {(child: Child) -> ()? in child.pet?.toy?.play()}

虽然看起来挺奇怪的，但这就是事实，这样调用的返回将是一个 `()?` (或者写成 `Void?` 会更清楚一些)。使用的时候我们可以通过 Optional Binding 来判定方法是否调用成功：

    if let result: () = playClosure(xiaoming) {
        println("好开心~")
    } else {
        println("没有玩具可以玩 :(")
    }

