# Sequence

Swift 的 `for in` 可以用在所有实现了 `SequenceType` 的类型上，而为了实现 `SequenceType` 你首先需要实现一个 `GeneratorType`。比如一个实现了反向的 generator 和 sequence 可以这么写：

    // 先定义一个实现了 GeneratorType 这个 protocol 的类型
    // GeneratorType 需要指定一个 typealias Element 
    // 以及提供一个返回 Element? 的方法 next()
    class ReverseGenerator: GeneratorType {
        typealias Element = Int
        
        var counter: Element
        init<T>(array: [T]) {
            self.counter = array.count - 1
        }
        
        init(start: Int) {
            self.counter = start
        }
        
        func next() -> Element? {
            return self.counter < 0 ? nil : counter--
        }
    }
    
    // 然后我们来定义 SequenceType
    // 和 GeneratorType 很类似，不过换成指定一个 typealias Generator
    // 以及提供一个返回 Generator? 的方法 generate()
    struct ReverseSequence<T>: SequenceType {
        var array: [T]
        
        init (array: [T]) {
            self.array = array
        }
        
        typealias Generator = ReverseGenerator
        func generate() -> Generator {
            return ReverseGenerator(array: array)
        }
    }
    
    let arr = [0,1,2,3,4]

    // 每次对 SequenceType 使用 forin，其实就是调用 generate() 获取一个 Generator 的实例
    // 然后不断调用这个 Generator 的 next() 方法
    for i in ReverseSequence(array: arr) {
        println("Index \(i) is \(arr[i])")
    }

输出为

    Index 4 is 4
    Index 3 is 3
    Index 2 is 2
    Index 1 is 1
    Index 0 is 0

顺便你可以免费得到的收益是你可以使用像 `map` , `filter` 和 `reduce` 这些方法，因为它们都有对应 `SequenceType ` 的版本：

<div class="ui orange segment">
<code>func map&lt;S : SequenceType, T&gt;(source: S, transform: (S.Generator.Element) -&gt; T) -&gt; [T]</code>
<p></p>
<code>func filter&lt;S : SequenceType&gt;(source: S, includeElement: (S.Generator.Element) -&gt; Bool) -&gt; [S.Generator.Element]</code>
<p></p>
<code>func reduce&lt;S : SequenceType, U&gt;(sequence: S, initial: U, combine: (U, S.Generator.Element) -&gt; U) -&gt; U</code>
</div>
