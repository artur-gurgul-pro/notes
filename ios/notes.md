---
layout: default
title: Swift - notes
categories: swift
---

#### Autoclosure

Lazy evaluation of the function's arguments. Instead of eager calculation of values, the clousure is passed, and executed only when needed.

```swift
func test(_ closure: @autoclosure() -> Bool) {
    <#Code#>
}

test(8==9)
```

#### Destructuring Assignment

```swift
let (name, surname) = ("Artur", "Gurgul")
```

#### Checking if value is in range

```swift
let i = 101
if case 100...101 = i {
    <#Code#>
}

if (100...101).contains(i) {
    <#Code#>
}
```

#### Accessing tuple values

```swift
("value1", "value2").0
```

#### Pattern matching 

```swift
enum Example {
    case first(String)
    case secund(String)
}

let example: Example = .first("test")
```

```swift
switch example {
case .first(let value), .secund(let value):
    print(value)
}

if case let .first(value) = example {
    print(value)
}
```

_print odd numbers_

```swift
for i in 1...100 where i%2 != 0 {

} 
```

#### Blurable view in UIKit

```swift
extension Blurable where Self: UIView {
    func addBlur(_ alpha: CGFloat = 0.5) {
        let effect = UIBlurEffect(style: .prominent)
        let effectView = UIVisualEffectView(effect: effect)
        effectView.frame = self.bounds
        effectView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        effectView.alpha = alpha
        self.addSubview(effectView)
    }
}
```

```swift
extension BackgroundView: Blurable {}
```

## Difference between `@ObservedObject`, `@State`, and `@EnvironmentObject`

[https://www.hackingwithswift.com/quick-start/swiftui/whats-the-difference-between-observedobject-state-and-environmentobject](https://www.hackingwithswift.com/quick-start/swiftui/whats-the-difference-between-observedobject-state-and-environmentobject)

>- Use `@State` for simple properties that belong to a single view. They should usually be marked `private`.
>- Use `@ObservedObject` for complex properties that might belong to several views. Most times you’re using a reference type you should be using `@ObservedObject` for it.
>- Use `@StateObject` once for each observable object you use, in whichever part of your code is responsible for creating it.
>- Use `@EnvironmentObject` for properties that were created elsewhere in the app, such as shared data.

### Cold vs hot observables

From: Anton Moiseev's Book [“Angular Development with Typescript, Second Edition.”](https://www.manning.com/books/angular-development-with-typescript-second-edition) :

> **Hot and cold observables**
> 
> There are **two** types of **observables**: hot and cold. The main difference is that a **cold observable** **creates** a **data producer** for **each subscriber**, whereas a **hot observable creates** a **data producer first**, and **each subscriber** gets the **data** from **one producer**, **starting** from **the moment of** **subscription**.
> 
> Let’s compare watching a **movie** on **Netflix** to going into a **movie theater**. Think of yourself as an **observer**. Anyone who decides to watch Mission: Impossible on Netflix will get the entire movie, regardless of when they hit the play button. Netflix creates a new **producer** to stream a movie just for you. This is a **cold observable**.
> 
> If you go to a movie theater and the showtime is 4 p.m., the producer is created at 4 p.m., and the streaming begins. If some people (**subscribers**) are late to the show, they miss the beginning of the movie and can only watch it starting from the moment of arrival. This is a **hot observable**.
> 
> A **cold observable** starts producing data when some code invokes a **subscribe()** function on it. For example, your app may declare an observable providing a URL on the server to get certain products. The request will be made only when you subscribe to it. If another script makes the same request to the server, it’ll get the same set of data.
> 
> A **hot observable** produces data even if no subscribers are interested in the data. For example, an accelerometer in your smartphone produces data about the position of your device, even if no app subscribes to this data. A server can produce the latest stock prices even if no user is interested in this stock.


# Interesting snippets

<!-- https://swiftbysundell.com/articles/swiftui-views-versus-modifiers/ -->

# View modifier 

The notification we'll send when a shake gesture happens.

```swift
extension UIDevice {
    static let deviceDidShakeNotification = Notification
                                              .Name(rawValue: "deviceDidShakeNotification")
}
```

 Override the default behavior of shake gestures to send our notification instead.

```swift
extension UIWindow {
     open override func motionEnded(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        if motion == .motionShake {
            NotificationCenter
                .default
                .post(name: UIDevice.deviceDidShakeNotification, object: nil)
        }
     }
}
```

A view modifier that detects shaking and calls a function of our choosing.

```swift
struct DeviceShakeViewModifier: ViewModifier {
    let action: () -> Void

    func body(content: Content) -> some View {
        content
            .onAppear()
            .onReceive(NotificationCenter
                           .default
                           .publisher(for: UIDevice.deviceDidShakeNotification)) { _ in
                               action()
            }
    }
}
```

A View extension to make the modifier easier to use.

```swift
extension View {
    func onShake(perform action: @escaping () -> Void) -> some View {
        self.modifier(DeviceShakeViewModifier(action: action))
    }
}
```

An example view that responds to being shaken

```swift
struct ContentView: View {
    var body: some View {
        Text("Shake me!")
            .onShake {
                print("Device shaken!")
            }
    }
}
```

# Swizzling

```swift
extension UIViewController {
    @objc dynamic func newViewDidAppear(animated: Bool) {
        viewDidAppear(animated)
        print("View appeared")
    }
}

private let swizzling: Void = {
    let originalMethod = class_getInstanceMethod(UIViewController.self,
                                                 #selector(UIViewController.viewDidLoad))
    let swizzledMethod = class_getInstanceMethod(UIViewController.self,
                                                 #selector(UIViewController.newViewDidAppear))
    if let originalMethod = originalMethod, let swizzledMethod = swizzledMethod {
        method_exchangeImplementations(originalMethod, swizzledMethod)
    }
}()
```

# class vs static values

```swift
class Car {
    static var start: Int {
        return 100
    }

    class var stop: Int {
        return 0
    }
}
```

```swift
class Student: Person {
    // Not allowed
    // override static var start: Int {
    //    return 150
    // }

    override class var stop: Int {
        return 5
    }
}
```


# Always Publisher example

```swift
public struct Always<Output>: Publisher {
    public typealias Failure = Never
    public let output: Output

    public init(_ output: Output) {
        self.output = output
    }

    public func receive<S: Subscriber>(subscriber: S)
                where S.Input == Output, S.Failure == Failure {
        let subscription = Subscription(output: output, 
                                        subscriber: subscriber)
        subscriber.receive(subscription: subscription)
    }
}
```

```swift
private extension Always {
    final class Subscription<S: Subscriber> 
                where S.Input == Output, S.Failure == Failure {
        private let output: Output
        private var subscriber: S?

    init(output: Output, subscriber: S) {
        self.output = output
        self.subscriber = subscriber
    }
  }
}
```

```swift
extension Always.Subscription: Cancellable {
  func cancel() {
    subscriber = nil
  }
}
```

```swift
extension Always.Subscription: Subscription {
  func request(_ demand: Subscribers.Demand) {
    var demand = demand
    while let subscriber = subscriber, demand > 0 {
      demand -= 1
      demand += subscriber.receive(output)
    }
  }
}
```

# Buffered publisher

```swift
let publisher = PassthroughSubject<Int, Never>()
publisher.send(1)
let buffered = publisher.buffer(size: 4, prefetch: .keepFull, whenFull: .dropOldest)
```

# Combine publishers

```swift
 let publisher1 = [1,2].publisher
 let publisher2 = [3,4].publisher
```

Emits first and then second

```swift
 let combinedPublisher = Publishers
                             .Concatenate(prefix: publisher1.eraseToAnyPublisher(),
                                          suffix: publisher2.eraseToAnyPublisher())
```

Combine without ordering

```swift
let combinedPublisher = Publishers.Merge(publisher1.eraseToAnyPublisher(),
                                         publisher2.eraseToAnyPublisher())
```

**_Other operators worth to look at_**

- `Zip` - pair the emitted object, like a zip in the jacket
- [`CombineLatest`](https://developer.apple.com/documentation/combine/publisher/combinelatest%28_:%29) - When `publisher1` and `publisher2` emitted some event then the latest values from each are taken and reemitted. From now on each change from either publisher is passed down.  

#### Example of zip

```swift
let numbers = [1, 2, 3, 4].publisher
let twos = sequence(first: 2, 
                    next: {_ in 2}).publisher
numbers
	.zip(twos)
	.map { pow(Decimal($0), $1) }
	.sink(receiveValue: { p in
		print(p)
	}).store(in: &cancellables)
```

#### Cancelling a publisher

```swift
let timer = Timer
	.publish(every: 1.0, on: .main, in: .common)
	.autoconnect()
```

```swift
var counter = 0
subscriber = timer
	.map { _ in counter += 1 }
	.sink { _ in
		if counter >= 5 {
			timer.upstream.connect().cancel()
		}
	}
```

It will work similar to this

```swift
subscriber = timer.prefix(5)
```


#### Assign 

```swift
class Dog {
	var name: String = ""
}

let dog = Dog()
let publisher = Just("Snow")
publisher.assign(to:/.name, on: dog)
```

```swift
class MyModel: ObservableObject {
	@Published var lastUpdated: Date = Date()
	init() {
		Timer
			.publish(every: 1.0, on: .main, in: .common)
			.autoconnect()
			.assign(to: &$lastUpdated)
	}
}
```

```swift
class MyModel: ObservableObject {
	@Published var id: Int = 0
}

let model = MyModel()
Just(100).assign(to: &model.$id)
```

Here's an example of using the `@dynamicMemberLookup` attribute in Swift:

```swift
@dynamicMemberLookup
struct DynamicStruct {
    subscript(dynamicMember member: String) -> String {
        return "You accessed dynamic member '\(member)'"
    }
}

let dynamicStruct = DynamicStruct()
let result = dynamicStruct.someDynamicMember
print(result) // Output: "You accessed dynamic member 'someDynamicMember'"
```

Key Value Coding

```swift
class SomeClass: NSObject {
  @objc dynamic var name = "Name"
}
```

```swift
object.value(forKey: "name") as String
```

```swift
object.setValue("New name", forKey: "name")
```


### Create map

```Swift
extension Sequence {
    func dictionay<T>(keyPath: KeyPath<Element, T>) -> [T: Element] {
        var dictionary =  [T: Element]()
        
        for elemement in self {
            let key = elemement[keyPath: keyPath]
            dictionary[key] = elemement
        }
        
        return dictionary
    }
}
```

### Compact

```swift
extension Array {
  public func compact<T>() -> [T] where Element == Optional<T> {
    compactMap { $0 }
  }
}

```

### Swift - currying

[https://thoughtbot.com/blog/introduction-to-function-currying-in-swift](https://thoughtbot.com/blog/introduction-to-function-currying-in-swift)

```swift
func curry<A, B, C, D>(_ f: @escaping (A, B, C) -> D) -> (A) -> (B) -> (C) -> D {
    { a in { b in { c in f(a, b, c) } } }
}
    
func curry<A, B, C>(_ f: @escaping (A, B) -> C) -> (A) -> (B) -> C {
    { a in { b in f(a, b) } }
}

func uncurry<A, B, C>(_ f: @escaping (A) -> (B) -> C) -> (A, B) -> C {
    { f($0)($1) }
}

func uncurry<A, B, C, D>(_ f: @escaping (A) -> (B) -> (C) -> D) -> (A, B, C) -> D {
    { f($0)($1)($2) }
}

func currying<A, B, C>(_ a: A, _ f: @escaping (A, B) -> C) -> (B) -> C {
    { (curry(f))(a)($0) }
}

func currying<A, B, C, D>(_ a: A, _ f: @escaping (A, B, C) -> D) -> (B, C) -> D {
    { (curry(f))(a)($0)($1) }
}
```

#### Example of usage

```swift
func add(a: Int, b: Int, c: Int) -> Int {
    a + b + c
}
```

```swift
let adding = curry(add)
let adding5 = uncurry(adding(5))
```

or

```swift
let adding5 = currying(5, add)
```

then

```swift
print(adding5(6, 7))
```

## Check availability 

[https://www.avanderlee.com/swift/available-deprecated-renamed/](https://www.avanderlee.com/swift/available-deprecated-renamed/)

```swift
if #available(iOS 15, *) {
    print("This code only runs on iOS 15 and up")
} else {
    print("This code only runs on iOS 14 and lower")
}
```

```swift
guard #available(iOS 15, *) else {
    print("Returning if iOS 14 or lower")
    return
}

print("This code only runs on iOS 15 and up")
```

```swift
@available(iOS 14, *)
final class NewAppIntroduction {
    // ..
}
```

```swift
@available(iOS, deprecated: 12, obsoleted: 13, message: "We no longer show an app introduction on iOS 14 and up")
@available(*, unavailable, renamed: "launchOnboarding")
```

Mapping

```swift
func maping<T>(keyPath: KeyPath<Element, T>) -> [T: Element] {
	var dictionary =  [T: Element]()
	
	for elemement in self {
		let key = elemement[keyPath: keyPath]
		dictionary[key] = elemement
	}
	
	return dictionary
}
```

### Type aliases

```swift
public typealias Point<T: Numeric> = (x: T, y: T)
public typealias MyResult<T> = Result<T, Error>
```

async/await