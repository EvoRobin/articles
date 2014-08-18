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
