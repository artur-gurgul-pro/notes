---
layout: default
title: Threading
categories: swift
---

Chunk of jobs that can be performed on the separate thread are arranged into:

- `Closures` or `Functions`/`Methods` and send to `DispatchQueue`
-  Classes and send as objects to `OperationQueue`. The queue can perform `Closure` as well

#### Kinds of jobs

* Computation intensive jobs: When the thread uses entire processing capability of `CPU`. The reasonable maximum number of the threads is the number of CPU cores.
* I/O intensive jobs: In that case we can trigger more threads than we have CPU cores. An optimal of threads is `Threads` = `Cores` / (1-`Blocking Factor`). 


# [GCD](https://github.com/apple/swift-corelibs-libdispatch) (Grand Central Dispatch)

Creating queue. Note without parameter `attributes` we will get serial queue

```swift
let queue = DispatchQueue(label: "important.job",
                          qos: .default,
                          attributes: .concurrent)
```

#### Examples of sending jobs to queues:

Send to the queue and proceed with the execution in the current context
```swift
queue.async {
    <#Code of the job#>
}
```

Same as above, but on main thread where UI operations should be done

```swift
DispatchQueue.main.async {
     <#Code of the job#>
}
```

Perform on the global queue

```swift
DispatchQueue.global(qos: .background).async {
    <#Code of the job#>
}
```

Send block that will be executed when all jobs sent at this point are completed. Stop current thread will the queue is done with the sent block.

```swift
queue.sync(flags: [.barrier]) {
    <#Code of the job#>
}
```

Perform job in the time interval 

```swift
queue.schedule(after: .init(.now()), interval: .seconds(3)) {
    <#Code of the job#>
}
```

Getting the label of the queue

```swift
extension DispatchQueue {
    static var label: String {
        return String(cString: __dispatch_queue_get_label(nil), 
                      encoding: .utf8) ?? ""
    }
}
```

### `DispatchGroup`

```swift
extension Sequence {
    public func threadedMap<T>(_ mapper: @escaping (Element) -> T) -> [T] {
        var output = [T]()
        let group = DispatchGroup()
        let queue = DispatchQueue(label: "queue-for--map-result", qos: .background)
        
        for obj in self {
            queue.async(group:group) {
                output.append(mapper(obj))
            }
        }
        
        group.wait()
        return output
    }
}
```

### Mutex

Performing max 3 tasks at the time

```swift
let semaphore = DispatchSemaphore(value: 3)
                
for _ in 0..<15 {
    queue.async(qos: .background) {
        semaphore.wait()
        <#Code of the job#>
        semaphore.signal()
   }
}
```

### Locking and unlocking the thread

**NSLock**

Example of usage. The concurrent queue becomes sort of serial. One job at the time, but execution in random order.

```swift
let lock = NSLock()
for _ in 1...6 {
    queue.async {
        lock.lock()
        <#Code of the job#>
        lock.unlock()
    }
}
```

**Mutex**

Same thing using `pthread` API

```swift
var mutex = pthread_mutex_t()
pthread_mutex_init(&mutex, nil)
for i in 1...6 {	
    queue.async {
        pthread_mutex_lock(&mutex)
        <#Code of the job#>
        pthread_mutex_unlock(&mutex)
    }
}
```

When you are done with the lock, you need to release it

```swift
pthread_mutex_destroy(&mutex)
```

# `NSOperationQueue`

```swift
let queue = OperationQueue()
queue.name = "Queue Name"
queue.maxConcurrentOperationCount = 1
```

```swift
class MyVeryImportantOperation: Operation {
    override func main() {
        if isCancelled { return }
        // Some chunk of time consuming task
        if isCancelled { return }
        // Some another chunk of time consuming task
        // and so on...
    }
}
```

Sending jobs to the queue

```swift
let myOperation = MyVeryExpensiveOperation()
queue.addOperation(myOperation)
```

```swift
queue.addOperation {
    <#Code of the job#>
}
```


Notes `NSOperationQueue` fit better when requires canceling, suspending a block and/or  dependency management

# NSThread

```swift
let thread = Thread {
    <#code#>
}   
```

```swift
let thread = Thread(target: self,
                    selector: #selector(jobMethod:),
                    object: argumentObject)
```

```swift
thread.start()
```

Async method

```swift
func doStuff() async {
	<#code#>
}
```

# Span a thread in `NSObject` context

```swift
perform(#selector(job), on: .main, with: nil, waitUntilDone: false)
```

# async/await

Available from `Swift 5.5`. More info at [docs.swift.org](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency/)

#### Await the task group

```swift
let movies = await withTaskGroup(of: Movie.self) { group in
     var movies = [Movie]()
     movies.reserveCapacity(2)
     for no in 1...2 {
          group.addTask {
               return await apiClient.download(movie: no)
          }
     }
     for await movie in group {
	     movies.append(movie)
     }
     return movies
}
```

**Order of execution**

```swift
print("Before task group")
await withTaskGroup(of: Void.self) { group in
    for item in list {
        group.addTask {
            await doSomething()
            print("Task completed")
        }
    }
    print("For loop completed")
}

print("After task group")
```

```plain
 1 => print("Before task group")
 2 => print("For loop completed")
 3 => print("Task completed")
 4 => print("Task completed")
 5 => print("Task completed")
 6 => print("After task group")
```
#### `Actor`s

- Do not support inheritance

```swift
actor TestActor {
    var property = 1
    func update(no: Int) {
        sleep(UInt32.random(in: 1...3))
        property = no
    }
}
```

```swift
let actor = TestActor()
await actor.update(no: 5)
```

#### Call synchronously  and parallel

```swift
let firstMovie  = await apiClient.download(movie: 1)
let secondMovie = await apiClient.download(movie: 2)
let movies = [firstMovie, secondMovie]
```

```swift
async let firstMovie  = apiClient.download(movie: 1)
async let secondMovie = apiClient.download(movie: 2)
let photos = await [firstMovie, secondMovie]
```
#### Call the main thread

```swift
let t = Task { @MainActor in
    print ("update UI")
    return 5
}
await print(t.value)
```

#### Call `async` function from a regular method

```swift 
func job(no: Int) async -> Int {
    sleep(UInt32.random(in: 1...3))
    return no
}
```

```swift
let result = Task {
    await job(no: 5)
}
Task {
    await print(result.value)
}
```
# pthread

This is an object that will be sent to the thread

```swift
class ThreadParameter {
    let paramater = 1
}
```

This is a function that will be spanned on the background thread

```swift
func threadedFunction(pointer: UnsafeMutableRawPointer) -> UnsafeMutableRawPointer? {
    var threadParameter = pointer.load(as: ThreadParameter.self)
    <#Code of the job#>
    return nil
}
```

Creating  a `pthread`

```swift
var myThread: pthread_t? = nil
let threadParameter = ThreadParameter()
var pThreadParameter = UnsafeMutablePointer<ThreadParameter>.allocate(capacity:1)
pThreadParameter.pointee = threadParameter
        
let result = pthread_create(&myThread, nil, threadedFunction, pThreadParameter)
```

waiting for finishing the thread

```swift
if result == 0, let myThread {
    pthread_join(myThread,nil)
}
```

# Span threads using `Combine` framework

**`subscribe(on:)`**

Quotes from: [https://trycombine.com/posts/subscribe-on-receive-on/](https://trycombine.com/posts/subscribe-on-receive-on/)

> `subscribe(on:)` sets the scheduler on which you’d like the current subscription to be “managed” on. This operator sets the scheduler to use for creating the subscription, cancelling, and requesting input.
>
> ... `subscribe(on:)` sets the scheduler to subscribe the upstream on.
> 
> A side effect of subscribing on the given scheduler is that `subscribe(on:)` also changes the scheduler for its downstream ....

Subscription is done once and calling second time the `subscribe(on: )` have no effect.

`subscribe(on: DispatchQueue.global(qos: .background))` queues on this call &rarr;  `func request(_ demand: Subscribers.Demand)`

**`receive(on:)`**

> The correct operator to use to change the scheduler where downstream output is delivered is `receive(on:)`.
>
> On other words `receive(on:)` sets the scheduler where the downstream receives output on.
>
> You can use `receive(on:)` similarly to `subscribe(on:)`. You can use multiple `receive(on:)` operators and that will always change the downstream scheduler

# Usage of `RunLoop`

<!-- Diffrence between queue and runloop https://www.avanderlee.com/combine/runloop-main-vs-dispatchqueue-main/ -->

Doncumentation on [RunLoop](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)

> The purpose of a run loop is to keep your thread busy when there is work to do and put your thread to sleep when there is non. - Apple

# Measure a performance time 

```swift
let startTime = CFAbsoluteTimeGetCurrent()
<#Code of the job#>
let endTime = CFAbsoluteTimeGetCurrent()
print("Duration:\(endTime - startTime) seconds")
```

#### Stop thread for some time

```swift
sleep(2)
```

```swift
try await Task.sleep(until: .now + .seconds(2), clock: .continuous)
```
