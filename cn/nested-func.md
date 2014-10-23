# 方法嵌套

方法终于成为了[一等公民](http://en.wikipedia.org/wiki/First-class_citizen)，也就是说，我们可以将方法当作变量或者参数来使用了。更进一步地，我们甚至可以在一个方法中定义新的方法，这给代码结构层次和访问级别的控制带来的新的选择。

想想看有多少次我们因为一个方法主体内容过长，而不得不将它重构为好几个小的功能块的方法，然后在原来的主体方法中去调用这些小方法。这些具体负责一个个小功能块的方法也许一辈子就被调用这么一次，但是却不得不存在于整个类型的作用域中。虽然我们会将它们标记为私有方法，但是事实上它们所承担的任务往往和这个类型没有直接关系，而只是会在这个类型中的某个方法中被用到。更甚至这些小方法也可能有些复杂，我们还想进一步将它们分成更小的模块，我们很可能也只有将它们放到和其他方法平级的地方。这样一来，本来应该是进深的结构，却被整个展平了，导致之后在对代码的理解和维护上都很成问题。在 Swift 中，我们对于这种情况有了很好的应对，我们可以在方法中定义其他方法，也就是说让方法嵌套起来。

举个例子，我们在写一个网络请求的类 `Request` 时，可能面临着将请求的参数编码到 url 里的任务。因为输入的参数可能包括单个的值，字典，或者是数组，因此为了结构漂亮和保持方法短小，我们可能将情况分开，写出这样的代码：

    func appendQuery(var url: String, 
                         key: String, 
                       value: AnyObject) -> String {

        if let dictionary = value as? [String: AnyObject] {
            return appendQueryDictionary(url, key, dictionary)
        } else if let array = value as? [AnyObject] {
            return appendQueryArray(url, key, array)
        } else {
            return appendQuerySingle(url, key, value)
        }
    }

    func appendQueryDictionary(var url: String, 
                                   key: String, 
                                 value: [String: AnyObject]) -> String {
        //...
        return result
    }

    func appendQueryArray(var url: String, 
                              key: String, 
                            value: [AnyObject]) -> String {
        //...
        return result
    }

    func appendQuerySingle(var url: String, 
                               key: String, 
                             value: AnyObject) -> String {
        //...
        return result
    }

事实上后三个方法都只会在第一个方法中被调用，它们其实和 `Request` 没有直接的关系，所以将它们放到 `appendQuery` 中去会是一个更好的组织形式：

    func appendQuery(var url: String, 
                         key: String, 
                       value: AnyObject) -> String {

        func appendQueryDictionary(var url: String, 
                                       key: String, 
                                     value: [String: AnyObject]) -> String {
            //...
            return result
        }

        func appendQueryArray(var url: String, 
                                  key: String, 
                                value: [AnyObject]) -> String {
            //...
            return result
        }

        func appendQuerySingle(var url: String, 
                                   key: String, 
                                 value: AnyObject) -> String {
            //...
            return result
        }

        if let dictionary = value as? [String: AnyObject] {
            return appendQueryDictionary(url, key, dictionary)
        } else if let array = value as? [AnyObject] {
            return appendQueryArray(url, key, array)
        } else {
            return appendQuerySingle(url, key, value)
        }
    }

另一个重要的考虑是虽然 Swift 提供了 `public`，`internal` 和 `private` 三种访问权限，但是有些方法我们完全不希望在其他地方被直接使用。最常见的例子就是在方法的模板中：我们一方面希望灵活地提供一个模板来让使用者可以通过模板定制他们想要的方法，但另一方面又不希望暴露太多实现细节，或者甚至是让使用者可以直接调用到模板。一个最简单的例子就是在[参数修饰](#func-params)一节中提到过的类似这样的代码：

    func makeIncrementor(addNumber: Int) -> ((inout Int) -> ()) {
        func incrementor(inout variable: Int) -> () {
            variable += addNumber;
        }
        return incrementor;
    }

