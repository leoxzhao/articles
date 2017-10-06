# Migrate to Swift 4

New Xcode 9 is out. 
iOS developers need to do the thing that they knew and love --- migrating existing code into Swift 4.
As always, Swift 4 introduced some new features.
What you care the most at the time probably isn't how to use those new features, 
but how to fix the breaking changes that prevent your Swift 3 code from compiling.
You don't want tickling with Xcode when a hot fix needs to be uploaded into App Store ASAP.

You can get a detailed list of changes and updates of the Swift language in [here](https://swift.org/blog/swift-4-0-released/).
Things like "SE-0168 Multi-Line String Literals" are less relevant right now.
You can pick it up when you write new code.

Here are the things that matter for the migration.

## Use Xcode 8.x to migrate your project to Swift 3

If your code is still using Swift 2.x version, you need to download Xcode 8.x and use that to migrate your project into Swift 3 first. Xcode 9 does not support migration from 2.x to 3.x.
The good thing of late migration is that all of your external dependences are likely already migrated into Swift 3.x. You just need to find out the newer version of your Cocoapods or Carthage modules with Swift 3 support, then modify Podfile (for Cocoapods), or Cartfile (for Carthage), then run `pod install` or `cathrage update`.

## Use Swift 3 for external modules

Unlike previous version, Xcode 9 supports both Swift 3 and Swift 4 in the same project.

A lot of Xcode projects use either [Cocoapods](https://cocoapods.org/), 
or [Carthage](https://github.com/Carthage/Carthage) package management.

For Cocoapods, this makes migration easier. 
For example, if you are using Cocoapods in your project, it is likely that one of the pods still has not been made compatible with Swift 4 yet.
you can for now set those Cocoapods targets to use Swift 3.2, and use Swift 4 for main project.

For example, by the time I was doing it, ReactiveCocoa isn't Swift 4 compatible yet. I added this code at the end of Podfile:

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    if target.name == "ReactiveCocoa" || target.name == "Kanna"
      target.build_configurations.each do |config|
            config.build_settings['SWIFT_VERSION'] = '3.2'
      end
    end
  end
end
```

## Get Started

After you have done above tasks. You can now open Xcode project or workspace.
Then select the following command:

View -> Navigators -> Show Issue Nevigator ⌘5

Xcode code shows "Conversion to Swift 4 is available".

Click that will trigger the "Select targets to convert" dialog. Note that you need uncheck those Cocoapods modules
that you specifically specified to use Swift 3.x to avoid converting them.

## Stricker Cloure Checking in Swift 4

From Swift 3 to Swift 4, there are two types of changes, the language itself, and Apple's Swift libraries and frameworks. Xcode can do a remarkable good job of converting naming changes. But there are still some manual work.

For example, Xcode can automatically change `NSFontAttributeName` to `NSAttributedStringKey.font`, `NSForegroundColorAttributeName` to `NSAttributedStringKey.foregroundColor`. But it can not make interface changes.

In Swift 2.x, you probably have code like this.
```swift
    dispatch_async(dispatch_get_main_queue()) { _ in
        button.setTitle("Done", for: .normal)
    }
```

After being converted into Swift 3, it becomes like this:
```swift
    DispatchQueue.main.async { _ in
        button.setTitle("Done", for: .normal)
    }
```

While Swift 3 can compile this piece of code, Swift 4 can not.
Xcode reports an error: `Contextual closure type '() -> Void' expects 0 arguments, but 1 was used in closure body`.
Swift 4 has a stricker check than Swift 3 to closure. You need to delete `_ in` to pass.


Another example of stricker checker is:

```swift
class Dog {
    var name:String
    init(_ name: String) {
        self.name = name
    }
    func play(trick: (_ name: String, _ time: Float)->()) {
        trick(name, 10.0)
    }
}

// somewhere in your project
    let bob = Dog("Bob")
    bob.play { name in      // fine in Swift 3
        print("\(name) is doing a trick.")
    }
```

Note that the closure only declared `name`. Swift 3 is ok with not listing time parameters. But Swift 4 will throw an error: `Contextual closure type '(String, Float) -> ()' expects 2 arguments, but 1 was used in closure body`. So, in Swift 4, you must give exact amount of arguments, no more, no less.

## [SE-0169 Improve Interaction Between private Declarations and Extensions](https://github.com/apple/swift-evolution/blob/master/proposals/0169-improve-interaction-between-private-declarations-and-extensions.md)

A short version is, replace all `fileprivate` with `private`, it should just work.
There are cases that class B might access private members of class A defined in the same file.
In such cases, you need to switch back to fileprivate, and refactor the code to reduce access to private members later.

For example, for this file, dog.swift, you can simply replace `fileprivate` with `private`.

```swift
class Dog {
    var name: String
    fileprivate var sick: Bool    // this fileprivate can be replaced with private.
}

extension Dog: Walkable {
    func walk() {
        if (!sick) {
            print("\(name) is walking.")
        } else {
            print("\(name) is sick right now.")
        }
    }
}
```

But for this kind of usage, you need to keep fileprivate. 
Because Vet is another class, and therefore can not access private members of Dog.

```swift
protocol Walkable {
    func walk()
}

class Dog {
    var name: String
    fileprivate var sick: Bool    // this can not.
    
    init(name: String) {
        self.name = name
        self.sick = false
    }
}

class Vet {
    func diagnose(dog: Dog) {
        if (dog.sick) {
            print("\(dog.name) is sick.")
        }
    }
}
```

# [SE-0160 Limiting @objc inference](https://github.com/apple/swift-evolution/blob/master/proposals/0160-objc-inference.md)

Congratulations, you have made it this far. This is what I considered the most complicated part of the Swift 4 migration.
I knew this day will come, but just didn't expect it can be this quick.
Apple is really obsessed with optimization for a good reason.
New iPhone 8 has smaller battery than its predecessor, but can manage to achieve the same or exceed the battery life of iPhone 7.
Software optimization is not just an engineering practice. It is money.
If the phone can finish a task with less CPU cycle, that directly translates to smaller battery, less charging, and better user experience.

Now, let's get into the technical detail.

Objective C is a dynamic language. It has a runtime, small, but a runtime nonetheless.

Check this C code:

```c
#include <stdio.h>

void printGreetingMessage(char *name) {
    printf("Hello %s", name);
}

int main(int argc, const char * argv[]) {
    printGreetingMessage("Leo");  // <--
    return 0;
}
```

Which function `printGreetingMessage("Leo")` needs to call is known during compile time.
When this code is executed, computer simply pushs the parameters into stack, and then jump to where function `printGreetingMessage` is located.

Now, check this Objective C code:

```objc
#import <Foundation/Foundation.h>

@interface Dog: NSObject

-(void)bark;

@end

@implementation Dog

-(void)bark {
    NSLog(@"Bark, bark.");
}

@end


int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Dog *dog = [[Dog alloc] init];
        [dog bark];  // <--
    }
    return 0;
}
```

Objective C generates a message table for Dog. Each method of Dog has an entry in that table.
Then [dog bark] get executed, computer first needs to go to that table, and find a entry that matches `-(void)bark`, and then call it.
If you knew C++ virtual function table, this is worth. As the count of the methods for `Dog` grows, so does the table.
The search will be slower and slower. Apple did quite some optimization on it. But there is only so much you can do.
Part of the reason of introducing Swift is for performance.

If a class is defined and used purely in Swift, then there is no such penalty.
But Swift, as a language for macOS and iOS development, 
unavoidably needs to interact with Objective C code (most of the Apple's frameworks and some external libraries).
To maintain interoperability, in real world project, most Swift classes also need to maintain a message table, and behave the same as their Objective C counterparts. Well, not any more in Swift 4.

In Swift 4, without `@objc`, a member function, a member variable, a property, etc. will only available in Swift.
So accessing those only takes constant time.

Xcode 9 can automatically add `@objc`. The problem is that Xcode 9 can only infer from code. For KVO access, for example, **User Defined Runtime Attributes** in Interface Builder, Xcode can not handle it. Also, @IBAction, @IBInspectable and @IBOutlet imply `@objc`.
So, you need to select Swift 4 @objc Inference -> Minimize Inference, and then try to test the app to cover all possible UI or KVO scenarios.
After the conversion, Xcode automatically set **Swift 3 @objc Inference** to On. If a member function got hit without being marked with `@objc`, a log warning will immediately generate.

```
....swift:31:5: implicit Objective-C entrypoint -[<the method:>] is deprecated and will be removed in Swift 4; add explicit '@objc' to the declaration to emit the Objective-C entrypoint in Swift 4 and suppress this message
Current stack trace:
...
```

Keep doing the testing, until there is no such error exists in the log output.
After that, turn **Swift 3 @objc Inference** to Default. Now you can have an app that is more performant.

## Summary

It is kind of a pain that each new version of Swift introduces some break changes.
You need to touch a lot of file to just to migrate to newer version.
The more code you touch, the more issues you might introduce.
So, do it earlier, be prepared.
On the bright side, the changes make sense, and make Swift a better language.
I only wish Swift 4 would be a stable version, 
and no more dramatical change in the coming versions so we can at least have a certain period of peace time when we can focus on our products instead of chasing Xcode each year.



