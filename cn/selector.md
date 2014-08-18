# Selector

`@selector` 是 objc 时代的一个关键字，它可以将一个方法转换并赋值给一个 `SEL` 类型，它的表现很类似一个动态的函数指针。在 objc 时 selector 非常常用，从设定 target-action，到自举询问是否响应某个方法，再到指定接受通知时需要调用的方法等等，都是由 selector 来负责的。在 objc 里生成一个 selector 的方法一般是这个样子的：

    -(void) callMe {
        //...
    }

    -(void) callMeWithParam:(id)obj {
        //...
    }

    SEL someMethod = @selector(callMe);
    SEL anotherMethod = @selector(callMeWithParam:);

    // 或者也可以使用 NSSelectorFromString
    // SEL someMethod = NSSelectorFromString(@"callMe");
    // SEL anotherMethod = NSSelectorFromString(@"callMeWithParam:");

一般为了方便，很多人会选择使用 `@selector`，但是如果要追求灵活的话，可能会更愿意使用 `NSSelectorFromString` 的版本 -- 因为我们可以在运行时动态生成字符串，从而调用到对应的方法。

在 Swift 中没有 `@selector` 了，因此我们要生成一个 selector 的话只能使用字符串。Swift 里对应原来 selector 的类型是一个叫做 `Selector` 的结构体，它提供了一个接受字符串的初始化方法。像上面的两个例子在 Swift 中等效的写法是：

    func callMe() {
        //...
    }

    func callMeWithParam(obj: AnyObject!) {
        //...
    }

    let someMethod = Selector("callMe")
    let anotherMethod = Selector("callMeWithParam:")

和 objc 时一样，记得在 `callMeWithParam` 后面加上冒号 (:)，这才是完整的方法名字。多个参数的方法名也和原来类似，大概会是这个样子：

    func turnByAngle(theAngle: Int, speed: Float) {
        //...
    }

    let method = Selector("turnByAngle:speed:")

另外，因为 `Selector` 类型实现了 `StringLiteralConvertible`，因此我们甚至可以不使用它的初始化方法，而直接用一个字符串进行赋值，就可以完成创建了。

最后需要注意的是，selector 其实是 objc runtime 的概念，如果这个你的 selector 对应的方法只在 Swift 中可见的话 (也就是说它是一个 Swift 中的 private 方法)，在调用这个 selector 时你会遇到一个 unrecognized selector 错误：

E> ## 这是错误代码
E>
E> ~~~~~~~~~~~~~~~ 
E> private func callMe() {
E>     //...
E> }
E>
E> NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector:"callMe", userInfo: nil, repeats: true)
E> ~~~~~~~~~~~~~~~ 

正确的做法是在 `private` 前面加上 `@objc` 或者 `dynamic` 关键字，这样运行时就能找到对应的方法了。

    dynamic private func callMe() {
        //...
    }
    
    NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector:"callMe", userInfo: nil, repeats: true)





