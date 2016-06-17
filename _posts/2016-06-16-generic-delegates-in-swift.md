---
title:  "Generic Delegates in Swift"
date:   2016-06-16 21:18:00
description: "AssociatedType, Thunks & Type Erasure"
---

Recently, I stumbled over a problem as I wanted to integrate a delegate with generic function definitions. I created a protocol with an `associatedType` to make it generic. The protocol _(simplified)_ looked basically like this:

```swift
protocol StackDelegate  {
    associatedtype T
    func didPushElement(element: T, stack: Stack<T>)
}
```

I simply wanted to add the delegate property to my receiving object,

```swift
struct Stack<T> {
    var delegate : StackDelegate?
    ...
}
```

but then I found myself fighting with the following error:

```
Protocol 'StackDelegate' can only be used as a generic
constraint because it has Self or associated type
requirements
```

It turns out that when using a generic protocol in this way, the compiler can't infer a concrete type during compilation. I found a great article that explains this problem in depth: [Generic Protocols & Their Shortcomings](http://krakendev.io/blog/generic-protocols-and-their-shortcomings).

One way to solve this issue is to use a small helper struct called _[thunk](https://en.wikipedia.org/wiki/Thunk)_, that acts as bridge between the delegate and the receiving object:

```swift
struct StackDelegateThunk<T> : StackDelegate {
   private let _didPushElement : (T, Stack<T>) -> Void

   init<P:StackDelegate where P.T == T>(_ delegate: P) {
       _didPushElement = delegate.didPushElement
   }

   func didPushElement(element: T, stack: Stack<T>) {
       _didPushElement(element, stack)
   }
}
```

It forwards the calls from the receiving object to the delegate. _&bdquo;Using this, we can effectively erase our abstract generic protocol types in favor of another concrete, more fully-fledged type. This is often referred to as [type erasure](https://en.wikipedia.org/wiki/Type_erasure).&rdquo;_ <sup>[1](#footnote1)</sup>

Having the struct in place, the delegate declaration can replaced with the following lines:

```swift
struct Stack<T> {
    private(set) var delegate : StackDelegateThunk<T>?

    mutating func setDelegate<P:StackDelegate where P.T == T>(delegate: P) {
        self.delegate = StackDelegateThunk<T>(delegate)
    }
    ...
}
```

That's basically it. A playground with a full example can be found [here](https://github.com/mkoehnke/Playgrounds/blob/master/Generic-Delegates.playground/Contents.swift).

<a name="footnote1"></a><sup>[1]</sup> [Generic Protocols & Their Shortcomings](http://krakendev.io/blog/generic-protocols-and-their-shortcomings)
