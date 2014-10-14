# 下标

下标相信大家都很熟悉了，在绝大多数语言中使用下标来读写类似数组或者是字典这样的数据结构的做法，似乎已经是业界标准。在 Swift 中，`Array` 和 `Dictionary` 当然也实现了下标读写：

    var arr = [1,2,3]
    arr[2]            // 3
    arr[2] = 4        // arr = [1,2,4]

    var dic = ["cat":"meow", "goat":"mie"]
    dic["cat"]          // {Some "meow"}
    dic["cat"] = "miao" // dic = ["cat":"miao", "goat":"mie"]

数组的话没有什么好多说的，但是字典需要注意，我们通过下标访问得到的结果是一个 `Optional` 的值。这是很容易理解的，因为你不能限制下标访问时的输入值，对于数组来说如果越界了只好直接给你脸色让你崩掉，但是对于字典，查询不到是很正常的一件事情。对此，在 Swift 中我们有更好的处理方式，那就是返回 `nil` 告诉你没有要找的东西。

作为一门代表了先进生产力的语言，Swift 是允许我们自定义下标的。这不仅包含了对自己写的类型进行下标自定义，也包括了对那些已经支持下标访问的类型 (没错就是 `Array` 和 `Dictionay`) 进行扩展。我们重点来看看向已有类型添加下标访问的情况吧，比如说 `Array`。很容易就可以在 Swift 的定义文件 (在 Xcode 中通过 Cmd + 单击任意一个 Swift 内建的类型或者函数就可以访问到) 里，找到 `Array` 已经支持的下标访问类型：

    subscript (index: Int) -> T
    subscript (subRange: Range<Int>) -> Slice<T>

共有两种，它们分别接受单个 `Int` 类型的序号和一个表明范围的 `Range<Int>`，作为对应，返回值也分别是单个元素和一组对应输入返回的元素。

于是我们发现一个挺郁闷的问题，那就是我们很难一次性取出某几个特定位置的元素，比如在一个数组内，我想取出 index 为 0, 2, 3 的元素的时候，现有的体系就会比较吃力。我们很可能会要去枚举数组，然后在循环里判断是否是我们想要的位置。其实这里有更好的做法，比如说可以实现一个接受数组作为下标输入的读取方法：

    extension Array {
        subscript(input: [Int]) -> Slice<T> {
            get {
                var result = Slice<T>()
                for i in input {
                    assert(i < self.count, "Index out of range")
                    result.append(self[i])
                }
                return result
            }
            
            set {
                for (index,i) in enumerate(input) {
                    assert(i < self.count, "Index out of range")
                    self[i] = newValue[index]
                }
            }
        }
    }

这样，我们的 `Array` 的灵活性就大大增强了：

    var arr = [1,2,3,4,5]
    arr[[0,2,3]]            //[1,3,4]
    arr[[0,2,3]] = [-1,-3,-4]
    arr                     //[-1,2,-3,-4,5]

<div class="ui orange segment">
    <div class="header">
        <i class="fa fa-pencil"></i>
        练习
    </div>
<p>虽然我们在这里实现了下标为数组的版本，但是我并不推荐使用这样的形式。如果知道参数列表的读者也许会想为什么在这里我们不使用看起来更优雅的参数列表的方式，也就是 <code>subscript(input: Int...)</code> 的形式。不论从易用性还是可读性上来说，参数列表的形式会更好。但是存在一个问题，那就是在只有一个输入参数的时候参数列表会导致和现有的定义冲突，有兴趣的读者不妨试试看。当然，我们完全可以使用至少两个参数的的参数列表形式来避免这个冲突，即定义形如 <code>subscript(first: Int, second: Int, others: Int...)</code> 的下标方法，我想这作为练习留给读者进行尝试会更好。</p>
</div>