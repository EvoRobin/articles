# @autoclosure 和 ??

Apple 为了推广和介绍 Swift，破天荒地为这门语言开设了一个[博客](https://developer.apple.com/swift/blog/)(当然我觉着是因为 Swift 坑太多需要一个地方来集中解释)。其中[有一篇](https://developer.apple.com/swift/blog/?id=4)提到了一个叫做 `@autoclosure` 的关键词。

`@autoclosure` 可以说是 Apple 的一个非常神奇的创造，因为这更多地是像在 “hack” 这门语言。简单说，`@autoclosure` 做的事情就是把一句表达式自动地**封装**成一个闭包 (closure)。这样有时候在语法上看起来就会非常漂亮。

比如我们有一个方法接受一个闭包，当闭包执行的结果为 `true` 的时候进行打印：

    func logIfTrue(predicate: () -> Bool) {
        if predicate() {
            println("True")
        }
    }

在调用的时候，我们需要写这样的代码

    logIfTrue({return 2 > 1})

当然，在 Swift 中对闭包的用法有做一些简化，在这种情况下我们可以省略掉 `return`，写成：

    logIfTrue({2 > 1})

还可以更近一步，因为这个闭包是最后一个参数，所以可以使用尾随闭包 (trailing closure) 的方式把大括号拿出来，然后省略括号，变成：

    logIfTrue{2 > 1}

但是不管那种方式，要么是书写起来十分麻烦，要么是表达上不太清晰，看起来都让人生气。于是 `@autoclosure` 登场了。我们可以改换方法参数，在前面加上 `@autoclosure` 关键字：

    func logIfTrue(predicate: @autoclosure () -> Bool) {
        if predicate() {
            println("True")
        }
    }

这时候我们就可以直接写：

    logIfTrue(2 > 1)

来进行调用了，Swift 将会吧 `2 > 1` 这个表达式自动转换为 `() -> Bool`。这样我们就得到了一个写法简单，表意清楚的式子。

在 Swift 中，有一个非常有用的操作符，可以用来快速地对 `nil` 进行条件判断，那就是 `??`。这个操作符可以判断输入并在当左侧的值是非 `nil` 的 Optional 值时返回其 value，当左侧是 `nil` 时返回右侧的值，比如：

    var level : Int?
    var startLevel = 1

    var currentLevel = level ?? startLevel

在这个例子中我们没有设置过 `level`，因此最后 `startLevel` 被赋值给了 `currentLevel`。如果我们充满好奇心地点进 `??` 的定义，可以看到 `??` 有两种版本：

    func ??<T>(optional: T?, defaultValue: @autoclosure () -> T?) -> T?
    func ??<T>(optional: T?, defaultValue: @autoclosure () -> T) -> T

在这里我们的输入满足的是后者，虽然表面上看 `startLevel` 只是一个 `Int`，但是其实在使用时它被自动封装成了一个 `() -> Int`，有了这个提示，我们不妨来猜测一下 `??` 的实现吧：

    func ??<T>(optional: T?, defaultValue: @autoclosure () -> T) -> T {
        switch optional {
            case .Some(let value):
                return value
            case .None:
                return defaultValue()
            }
    }

可能你会有疑问，为什么这里要使用 `autoclosure`，直接接受 `T` 作为参数并返回不行么？这正是 `autoclosure` 的一个最值得称赞的地方。如果我们直接使用 `T`，那么就意味着在 `??` 操作符真正取值之前，我们就必须准备好一个默认值，这个默认值的准备和计算是会消耗性能的。但是其实要是 `optional` 不是 `nil` 的话，我们是完全不需要这个默认值，而会直接返回 `optional` 解包后的值。这样一来，默认值就白白准备了，这样的开销是完全可以避免的，方法就是将默认值的计算推迟到 `optional` 判定为 `nil` 之后。

就这样，我们可以巧妙地绕过条件判断和强制转换，以很优雅的写法处理对 `Optional` 及默认值的取值了。最后要提一句的是，`@autoclosure` 并不支持带有输入参数的写法，也就是说只有形如 `() -> T` 的参数才能使用这个特性进行简化。另外因为调用者往往很容易忽视 `@autoclosure` 这个特性，所以在写接受 `@autoclosure` 的方法时还请特别小心，如果在容易产生歧义或者误解的时候，还是使用完整的闭包写法会比较好。

<div class="ui orange segment">
	<div class="header">
		<i class="fa fa-pencil"></i>
	    练习
	</div>
<p>在 Swift 中，其实 <code>&&</code> 和 <code>||</code> 这两个操作符里也用到了 <code>@autoclosure</code>。作为练习，不妨打开 Playground，试试看怎么实现这两个操作符吧？</p>
</div>