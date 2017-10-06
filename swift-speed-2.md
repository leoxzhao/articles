# How Fast is Swift, and Why (Part 2)

In part 1, we knew that in a very simple case, Swift could be 9 times faster than Objective C.
That is a huge improvement. But is it as fast as it can be?
How does it compare to the gold standard of performance --- C?

Let's first measure how fast a C function can do the same task.
Luckily, you can mix Objective C and C code together.
No bridging, no nothing.
(You can use C++ in the same way just by renaming .m file to .mm)

```objc
long add(long a, long b) {
    return a + b;
}

@implementation OCCalculator
- (void)measureTimeOfC {
    CFTimeInterval start = CACurrentMediaTime();
    NSInteger sum = 0;
    for (int i = 0; i < 1000000000; i++) {
        sum = add(sum, 1);
    }
    CFTimeInterval endTime = CACurrentMediaTime();
    NSLog(@"Sum: %ld", sum);
    NSLog(@"C Total: %.12f s", endTime - start);
}
@end
```

Before I show you the numbers, make a guess?
Is the result comparable to Swift?
Is it faster, slower? By how much?

`C Total: 0.000000041677 s`

That is 10 million times faster than Swift.
That can not be right!

After checked the assembly generated, it turns out that the compiler calculated the result, and directly put it into the code.
As we dig deep into the performance, more compiler dark magics surface.
Even if I put `sum = add(sum, i);` in there, compiler can still calculate the value of `sum` during compile time.
A C++ developer can do the same thing using **template metaprogramming**.
But that requires much more skill, and generates code that is much harder to understand.

I found that to put compiler off, I need to use this `sum = add(sum, i % 2);`.
This time, I got the following result.

`C Total: 0.43306300 s`

This speed is very close to Swift's speed, a tad slower since there is a `%` operation in there. 

Now, can you say Swift is as fast as C? Not so fast.
The kind of optimization mentioned before is only done in C, not Swift.

So far, we have only shown the best part of Swift.
In part 3, I am going to check different scenarios to see how Swift behaves.
