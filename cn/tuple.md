# 多元组 (Tuple)

多元组 (Tuple) 是我们的新朋友，多尝试使用这个新特性吧，会让生活轻松不少～

比如交换输入，普通程序员亘古以来可能都是这么写的

    func swapMe<T>(inout a: T, inout b: T) {
        let temp = a
        a = b
        b = a
    }

但是要是使用多元组的话，我们可以不使用额外空间就完成交换，一下子就达到了文艺程序员的写法

    func swapMe<T>(inout a: T, inout b: T) {
        (a,b) = (b,a)
    }

另外一个挺常用的地方是错误错误处理。objc 时代我们已经习惯了在需要错误处理的时候先做一个 `NSError` 的指针，然后将地址传到方法里等返回：

    NSError *error = nil;
    BOOL success = [[NSFileManager defaultManager] 
                            moveItemAtPath:@"/path/to/target"
                                    toPath:@"/path/to/destination"
                                     error:&error];
    if (!success) {
        NSLog(@"%@", error);
    }

现在如果我们新写库的时候可以考虑直接返回一个带有 `NSError` 的多元组，而不是去填充地址了：

    func doSomethingMightCauseError() -> (Bool, NSError?) {
        //...做某些操作，成功结果放在 ok 中
        if ok {
            return (true, nil)
        } else {
            return (false, NSError(domain:"SomeErrorDomain", code:1, userInfo: nil))
        }
    }

然后使用的时候，对比之前的做法，现在就非常简单了：

    let (success, maybeError) = doSomethingMightCauseError()
    if let error = maybeError {
        // 发生了错误
    }


<div class="ui orange segment">
<p>一个有趣但是不被注意的事实，其实在 Swift 中任何东西都是放在多元组里的。</p>
<p>不相信？试试看输出这个吧</p>
<pre><code>
  var num = 42
  println(num)
  println(num.0.0.0.0.0.0.0.0.0.0)
 </code></pre>
</div>


