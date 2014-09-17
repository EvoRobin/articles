# 柯里化

Swift 里可以将方法进行[柯里化 (Currying)](http://en.wikipedia.org/wiki/Currying)，也就是把接受多个参数的方法变换成接受第一个参数的方法，并且返回接受余下的参数而且返回结果的新方法。举个例子，Swift 中我们可以这样写出多个括号的方法：

    func addTwoNumbers(a: Int)(num: Int) -> Int {
        return a + num
    }

然后通过只传入第一个括号内的参数进行调用，这样将返回另一个方法：

    let addToFour = addTwoNumbers(4)    // addToFour 是一个 Int -> Int
    let result = addToFour(num: 6)      // result = 10

或者

    func greaterThan(comparor: Int)(input : Int) -> Bool{
        return input > comparor;
    }
    
    let greaterThan10 = greaterThan(10);
    
    greaterThan10(input : 13)    // 结果是 true
    greaterThan10(input : 9)     // 结果是 false

柯里化是一种量产类似方法的好办法，可以通过柯里化一个方法模板来避免写出很多重复代码，也方便了今后维护。

举一个实际应用时候的例子，在 [Selector](http://swifter.tips/selector/) 一节中，我们提到了在 Swift 中 `Selector` 只能使用字符串在生成。这面临一个很严重的问题，就是难以重构，并且无法在编译期间进行检查，其实这是十分危险的行为。但是 target-action 又是 Cocoa 中如此重要的一种设计模式，无论如何我们都想安全地使用的话，应该怎么办呢？一种可能的解决方式就是利用方法的柯里化。Ole Begemann 在[这篇帖子](http://oleb.net/blog/2014/07/swift-instance-methods-curried-functions/?utm_campaign=iOS_Dev_Weekly_Issue_157&utm_medium=email&utm_source=iOS%2BDev%2BWeekly)里提到了一种很好封装，这为我们如何借助柯里化，安全地改造和利用 target-action 提供了不少思路。


    protocol TargetAction {
        func performAction()
    }

    struct TargetActionWrapper<T: AnyObject>: 
                                TargetAction {
        weak var target: T?
        let action: (T) -> () -> ()
        
        func performAction() -> () {
            if let t = target {
                action(t)()
            }
        }
    }

    enum ControlEvent {
        case TouchUpInside
        case ValueChanged
        // ...
    }


    class Control {
        var actions = [ControlEvent: TargetAction]()
        
        func setTarget<T: AnyObject>(target: T, 
                       action: (T) -> () -> (), 
                    controlEvent: ControlEvent) {

            actions[controlEvent] = TargetActionWrapper(
                target: target, action: action)
        }
        
        func removeTargetForControlEvent(controlEvent: ControlEvent) {
            actions[controlEvent] = nil
        }
        
        func performActionForControlEvent(controlEvent: ControlEvent) {
            actions[controlEvent]?.performAction()
        }
    }

