---
title:  "Instantiate ViewControllers from Storyboard"
date:   2016-01-24 21:18:00
description: "A Better Way"
---

Once in a while I need to instantiate a ViewController from a Storyboard. Of course, I could just go ahead and use this method here:

```swift
let vc = UIStoryboard(name: "Main", bundle: nil)
	 .instantiateViewControllerWithIdentifier("ViewController")
	 as? ViewController
```

But, there are a few things that I don't like about this approach:

- I hate hardcoded strings all over my source code.
- This can be a mess when renaming / refactoring.
- I would like to have at least a little bit of type safety.
- It's [uuugly](http://youtu.be/EvtSzLSHVYc).

<p></p>

What I like to use instead is an _Enum_ containing all Storyboard names and a helper method for instantiating their ViewControllers:

```swift
enum Storyboard : String {
    case Main = "Main"
    // Add Storyboard names as needed.

    func getViewController<T : UIViewController>(identifier: String? = nil) -> T {
        let storyboardID = identifier ?? (NSStringFromClass(T.self) as NSString).pathExtension
        guard let vc = UIStoryboard(name: rawValue, bundle: nil).instantiateViewControllerWithIdentifier(storyboardID) as? T else {
            preconditionFailure("ViewController with ID '\(storyboardID)' doesn't exist in Storyboard '\(rawValue)'.")
        }
        return vc
    }
}
```

But before one can use any of this, the ViewController that is supposed to be instantiated, must have a _Storyboard ID_. This property can be changed in the _Identity Inspector_:

![My helpful screenshot]({{ site.baseurl }}assets/images/storyboard-id.png)

If the __Storyboard ID matches the ViewController class name__, you can instantiate with the following line:

```swift
let vc : ViewController = Storyboard.Main.getViewController()
```

If that's not the case, a different Storyboard ID can be passed as parameter.
