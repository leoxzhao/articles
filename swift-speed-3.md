# How Fast is Swift, and Why (Part 3)

In part 1, we knew that in a very simple case, Swift could be 9 times faster than Objective C.
That is a huge improvement.

In part 2, we knew that C function can be optimized at compile time.
The compiler can replace some function calls with values.
C++ has such capability known as **template metaprogramming**.
But for most real world use cases, Swift's speed is on a par with C.

Now, it is time to see if Swift can keep its speed in complex cases.

Here we have a Swift class `Calculator`, and a few classes both inherit from it, and change `add` behavior slightly.
I knew "generous" does not mean giving inflated addition result.
But I will not even use those new classes. So, no harm is done.

```swift
class Calculator: NSObject {
    func add(_ a: Int, with b: Int) -> Int {
        return a + b
    }
}

class GenerousCalculator: Calculator {
    override func add(_ a: Int, with b: Int) -> Int {
        return a + b + 1
    }
}

class MoreGenerousCalculator: Calculator {
    override func add(_ a: Int, with b: Int) -> Int {
        return a + b + 2
    }
}

class SuperGenerousCalculator: Calculator {
    override func add(_ a: Int, with b: Int) -> Int {
        return a + b + 100
    }
}
```

Here is the function to measure the time it takes to do a billion additions.

```swift
func measureSwiftCalls(_ calculator: Calculator, _ times: Int) {
    var sum = 0
    let startTime = CACurrentMediaTime()
    for _ in 0..<times {
        sum = calculator.add(sum, with: 2)
    }
    let endTime = CACurrentMediaTime()
    print("Sum: \(sum)")
    print("Native Swift: \(String(format: "%.8f", endTime - startTime)) s")
}

measureSwiftCalls(Calculator(), 1000000000)
```

Now, this method reports 1.27093250 seconds.
That is 3 times of that it took before adding those three classes.
It was 0.42621721 seconds.
Unexpected, eh?

So, using Swift, just by adding a few classes will have an significant impact on performance, at least for this case.
What was going on?

Before we dig deeper into this, let's first clarify some concept of programming.
Developers need to use programming languages --- Swift, Java, C#, Objective C, C++, Go --- to write code for computers to do things.
It is as if computers can understand those languages. It can not, nor does it care.
CPU has its own instruction set.
Even assembly language, which is as close as it can get to CPU's instruction set, is still a higher language for people to express their intention to computers.
A developer may feel a good level of control over computers via code.
And it was correct in early years when CPU was simple and slow, and compilers were more of a translation layer in between human-readable code to computer-executable instructions.

Today's situation is much complex then it was.
We need more expressive languages to boost productivity.
More design patterns, programming methodologies are invented and utilized.
To bridge the gap between more expressive languages to low level instruction sets, compilers become much more sophisticated.

CPU does not simply take instructions and run them one by one.
There are all kind of underlying techniques to speed up the execution time, like **out-of-order execution**.
X86 CPUs have one layer sitting between instructions and CPU hardware, **microcode**.

Having the mental model of one-to-one mapping between code to the action CPU takes helps us offload the burden while coding.
But when performance matters, you should know there are a lot of stones to flip.

Ok, time to check some assembly to find what Swift compiler did behind the scene.

![ARM Assembly](img/swift_override_effect.png)

As usual, Swift chose to inline those functions (even `add` functions are marked as override.)
But this time, since there are 4 different `add` functions, Swift need to find the right code block to run each time.
The red line depicts the execution flow.

So, for each loop, the CPU needs to do compare-branch-branch, therefore the slowness.

In this case, we are lucky. Since we are using `Calculator.add`, it is the first one. A call to `SuperGenerousCalculator.add` takes longer time to execute, and the result data shows that.

We can tell which function to call for sure since we know it is same object, it should always call `Calculator.add`.
There is no need to check each time.
But as powerful as it is, Swift compiler has its limit.

There is a new tool called **Protocol-oriented programming**.
It promotes using `protocol` and `struct`.
It also make certain things simple, as well as avoid performance penalties associated with classes.

