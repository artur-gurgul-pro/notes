---
layout: default
title: iOS Architecture Patterns
categories: swift
---
### Remove story board dependency

1. Remove `Main.storyboard` file
1. Remove storyboard reference from `Info.plist` &rarr; In Scene Configuration find `Storyboard Name` and delete it
1. Go to build settings and remove `UIKit MainStoryboard File Base Name` field
1. Create a window in Scene Delegate

```swift
func scene(_ scene: UIScene,
           willConnectTo session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {
    
    guard let scene = (scene as? UIWindowScene) else { return }
    
    let window = UIWindow(windowScene: scene)
    window.rootViewController = ViewController()
    window.makeKeyAndVisible()
    self.window = window
}
```

## Dependency injection

<!-- https://www.swiftbysundell.com/tips/testing-code-that-uses-static-apis/ -->

[Explanation of the code under this link](https://www.avanderlee.com/swift/dependency-injection/)


```swift
public protocol InjectionKey {
    associatedtype Value
    static var currentValue: Self.Value { get set }
}
```

```swift
struct InjectedValues {
    private static var current = InjectedValues()
    
    static subscript<K>(key: K.Type) -> K.Value where K : InjectionKey {
        get { key.currentValue }
        set { key.currentValue = newValue }
    }
    
    static subscript<T>(_ keyPath: WritableKeyPath<InjectedValues, T>) -> T {
        get { current[keyPath: keyPath] }
        set { current[keyPath: keyPath] = newValue }
    }
}
```

```swift
@propertyWrapper
struct Injected<T> {
    private let keyPath: WritableKeyPath<InjectedValues, T>
    var wrappedValue: T {
        get { InjectedValues[keyPath] }
        set { InjectedValues[keyPath] = newValue }
    }
    
    init(_ keyPath: WritableKeyPath<InjectedValues, T>) {
        self.keyPath = keyPath
    }
}
```

#### Define dependency

```swift
private struct UsersRepositoryKey: InjectionKey {
    static var currentValue: AnyUsersRepository = UsersRepository()
}

extension InjectedValues {
    var usersRepository: AnyUsersRepository {
        get { Self[UsersRepositoryKey.self] }
        set { Self[UsersRepositoryKey.self] = newValue }
    }
}

protocol AnyUsersRepository {
    func getUsers(_ result: @escaping (Result<[User], Error>)->Void)
}

class UsersRepository: AnyUsersRepository {
    func getUsers(_ result: @escaping (Result<[User], Error>)->Void) {
        <#Implementation#>
    }
}
```

```swift
@Injected(\.usersRepository) var usersRepository: AnyUsersRepository
```

```swift
InjectedValues[\.usersRepository] = MockedUsersRepository()
```

## Model-View-Controller

#### Clasic version

- `View` and `Model` are linked together, so reusability is reduced.
- Note: Views in iOS apps are quite often reusable.

<p>
 {% svg ../svgs/classic-mvc.svg class="center-image" %}
</p>

#### Apple version

<p>
 {% svg ../svgs/apple-mvc.svg class="center-image" %}
</p>

**Model** responsibilities: 

- Business logic
- Accessing and manipulating data
- Persistence
- Communication/Networking 
- Parsing
- Extensions and helper classes
- Communication with models

Note: The `Model` must not communicate directly with the `View`. The `Controller` is the link between those

**View** responsibilities:

- Animations, drawings (`UIView`,  `CoreAnimation`, `CoreGraphics`)
- Show data that controller sends
- Might receive user input

**Controller** responsibilities:

-  Exchange data between `View` and `Model`
- Receive user actions and interruptions or signals from the outside the app
- Handles the view life cycle

#### Advantages

- Simple and usually less code
- Fast development for simple apps

#### Disadvantages

- Controllers coupled views
- Massive `ViewController`s

#### Communication between components

- [Delegation](https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/TargetAction.html) pattern
- [Target-Action](https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/TargetAction.html) pattern
- [Observer](https://developer.apple.com/documentation/foundation/nsnotificationcenter) pattern with `NSNotificationCenter`
- [Observer](https://developer.apple.com/documentation/swift/using-key-value-observing-in-swift) pattern with `KVO`

## Model-View-Presenter

In this design pattern View is implemented with classes `UIView` and `UIViewController`. The `UIViewController` has less responsibilities which are limited to:

- Routing/Coordination
- Navigation
- Passing informations via a delegation pattern

#### View

```swift
class ExampleController: UIViewController {
	private let exampleView = ExampleView()
	override func loadView() {
		super.loadView()
		setup()
	}
	private func setup() {
		let presenter = ExamplePresenter(exampleView)
		exampleView.presenter = presenter
		exampleView.setupView()
		self.view = exampleView
	}
}
```


#### Presenter

```swift
protocol ExampleViewDelegate {
	func updateView()
}
class ExamplePresenter {
	private weak var exampleView: ExampleViewDelegate?
	init(_ exampleView: ExampleViewDelegate) {
		self.exampleView = exampleView
	}
}
```

#### Advantages 

- Easier to test business logic
- Better separation of responsibilities

#### Disadvantages 

- Usually not a better choice for smaller projects
- Presenters might become massive
- Controllers still handle navigation. Possible solutions &rarr;  extend the pattern with Router or Coordinator.

#### Common layers

- Data access layer: `CRUD` operations facilitated with `CoreData` `Realm` etc.
- Services: Classes that interacts with database entities, like retrieve data, transform them into objects.
- Extensions/utils

## Model-View-ViewModel

`ViewModel` has no references to the view. 

<p>
{% svg ../svgs/mvvm.svg class="center-image" %}
</p>

Binding is done using: `Combine Framework`, `RxSwift`, `Bond` or `KVO` or using delegation pattern 

* **`Model`** does same things as in `MVP` and `MVC`.
* **`View`** also is similar, but binds with `ViewModel` 
* **`ViewModel`** keeps updated state of the view, and process data for i

#### Advantages 

- Better reparation of responsibilities
- Better testability, without needing to take into account the views

#### Disadvantages 

- Might be slower and introduce dependency on external libraries
- Harder to learn and can become complex


#### **Extension with Coordinator MVVM-C**

Role of `Coordinator` is to manage navigation flow.

```swift
protocol Coordinator {
    var navigationController: UINavigationController { get set }
    func start()
}
```

and an example implementation

```swift
class ExampleCoordinator: Coordinator {
    var navigationController: UINavigationController
    init(navigationController: UINavigationController) {
        self.navigationController = UINavigationController
    }

    func start() {
        let viewModel = ExampleViewModel(someService: SomeService(),
                                         coordinator: self)
        navigationController.pushViewController(ExampleController(viewModel),
                                                animated: true)
    }

    func showList(_ list: ExampleListModel) {
        let listCoordinator = ListCoordinator(navigationController: navigationController
                                              list: list)
        listCoordinator.start()
    }
}
```

In the book I am reading the author created an `ExampleCoordinatorProtocol` with a `func showList(_ list: ExampleListModel)` where the `ExampleCoordinator` implemented it. I think it does not make any sense, however if we might want to inject the coordinator then we might want to relay on an abstraction.

```swift
func scene(_ scene: UIScene,
           willConnectTo session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {
    
    guard let scene = (scene as? UIWindowScene) else { return }
    
    let window = UIWindow(windowScene: scene)
    let navigationController = UINavigationController()
    exampleCoordinator = ExampleCoordinator(navigationController: navigationController)
    exampleCoordinator?.start()
    window.rootViewController = navigationController
    window.makeKeyAndVisible()
    self.window = window
}
```

## VIPER

<p>
 {% svg ../svgs/viper-ownership.svg class="center-image" %}
</p>


#### View

It includes `UIViewController`

- Made only to preserve elements like Buttons Labels
- It sends informations to presenters, and receive messages what to show and knows how

#### Interactor 

- Receives informations form databases, servers etc.
- The book says that the Interactor receive actions from presenter, and returns the result via Delegation Pattern. 
- The interactor never sends entities to the Presenter

#### Presenter

- Is in the centre and serves as a link
- Process events from the view and requests data from the Interctor. It receives that as primitives, never Entities.
- it handles navigation to the other screens using the Router

#### Entity

- Simple models usually data structures
- They can only be used by the Interactor

#### Router

- Creates screens
- Handles navigation, but itself does not know where to go to.
 - _The book says it is the owner of the `UINavigationController` and UIViewController, but it is contrary to other parts of the book, so I do not know_
- Similar to `Coordinator` form MVVM-C


<!--
https://github.com/ochococo/Design-Patterns-In-Swift

https://nalexn.github.io/clean-architecture-swiftui/
https://medium.com/@vladislavshkodich/architectures-comparing-for-swiftui-6351f1fb3605

-->

**`Entity.swift`**

```swift
struct User: Codable {
    let name: String
}
```

**`Interactor.swift`**

```swift
enum FetchError: Error {
    case failed
}

protocol AnyInteractor {
    var presenter: AnyPresenter? { get set }
    
    func getUsers()
}

class UserInteractor: AnyInteractor {
    @Injected(\.usersRepository) var usersRepository: AnyUsersRepository
    
    var presenter: AnyPresenter?
    
    func getUsers() {
        usersRepository.getUsers { [weak self] in self?.presenter?.interactorDidFetchUsers(with: $0) }
    }
}
```


**`Presenter.swift`**

```swift
protocol AnyPresenter {
    var router: AnyRouter? { get set }
    var interactor: AnyInteractor? { get set }
    var view: AnyView? { get set }
    
    func interactorDidFetchUsers(with result: Result<[User], Error>)
}

class UserPresenter: AnyPresenter {
    func interactorDidFetchUsers(with result: Result<[User], Error>) {
        switch result {
        case let .success(users):
            view?.update(with: users)
        case let .failure(error):
            view?.update(with: error.localizedDescription)
        }
    }
    
    var router: AnyRouter?
    
    var interactor: AnyInteractor? {
        didSet {
            interactor?.getUsers()
        }
    }
    
    var view: AnyView?
}
```

**`Router.swift`**

```swift
typealias EntryPoint = AnyView & UIViewController

protocol AnyRouter {
    var entry: EntryPoint? { get }
    static func start() -> AnyRouter
}

class UserRouter: AnyRouter {
    var entry: EntryPoint?
    
    
    static func start() -> AnyRouter {
        let router = UserRouter()
        
        var view: AnyView = UserViewController()
        var presenter: AnyPresenter = UserPresenter()
        var interactor: AnyInteractor = UserInteractor()
        
        view.presenter = presenter
        
        interactor.presenter = presenter
        
        presenter.router = router
        presenter.view = view
        presenter.interactor = interactor
        
        router.entry = view as? EntryPoint
        
        return router
    }
}



//There are a few retain cycles with view, presenter, router and interactor. One option you can do is to make those protocols conforms to AnyObject, and mark these references as "weak":
//1. router's ref to presenter
//2. router's ref to view
//3. presenter's ref to view
//4. interactor's ref to presenter
```

**`View.swift`**

```swift
protocol AnyView {
    var presenter: AnyPresenter? { get set }
    
    func update(with users: [User])
    func update(with error: String)
}

class UserViewController: UIViewController, AnyView {
    
    
    var presenter: AnyPresenter?
    
    private let tableView: UITableView = {
        let tableView = UITableView()
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        tableView.isHidden = true
        return tableView
    }()
    
    var users = [User]()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(tableView)
        tableView.delegate = self
        tableView.dataSource = self
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        tableView.frame = view.bounds
    }
    
    func update(with users: [User]) {
        DispatchQueue.main.async {
            self.users = users
            self.tableView.reloadData()
            self.tableView.isHidden = false
        }
    }
    
    func update(with error: String) {
        
    }
}


extension UserViewController: UITableViewDelegate, UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        users.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = users[indexPath.row].name
        return cell
    }
}
```


<!--
RIBs
https://github.com/uber/RIBs
https://medium.com/swlh/ios-architecture-exploring-ribs-3db765284fd8
https://github.com/uber/RIBs/wiki
-->

<!--
redux
https://medium.com/mackmobile/getting-started-with-redux-in-swift-54e00f323e2b
-->

<!-- 

# Factory

```swift
protocol ImageReader {
    func getDecodeImage() -> DecodedImage
}

class DecodedImage {
    private var image: String

    init(image: String) {
        self.image = image
    }

    var description: String {
        "\(image): is decoded"
    }
}

class GifReader: ImageReader {
    private var decodedImage: DecodedImage

    init(image: String) {
        self.decodedImage = DecodedImage(image: image)
    }

    func getDecodeImage() -> DecodedImage {
        decodedImage
    }
}

class JpegReader: ImageReader {
    private var decodedImage: DecodedImage

    init(image: String) {
        decodedImage = DecodedImage(image: image)
    }

    func getDecodeImage() -> DecodedImage {
        decodedImage
    }
}

func runFactoryExample() {
    let reader: ImageReader
    let format = "gif"
    let image = "example image"

    switch format {
    case "gif":
        reader = GifReader(image: image)
    default:
        reader = JpegReader(image: image)
    }
    
    let decodedImage = reader.getDecodeImage()
    print(decodedImage.description)
}
```


```swift

protocol Observer<ValueType> {
    associatedtype ValueType
    func update(value: ValueType)
}

struct Subject<T> {    
    private var observers: [(T) -> Void] = []
    
    mutating func attach<O: Observer>(observer: O) where O.ValueType == T {
        observers.append { observer.update(value: $0) }
    }

    func notyfi(value: T) {
        for observer in observers {
            observer(value)
        }
    }
}

class ConcreteObserver: Observer {
    func update(value: String) {
        print("received: \(value)")
    }
}

func runObserverExample() {
    var subject = Subject<String>()

    let observer1 = ConcreteObserver()
    subject.attach(observer: observer1)

    let observer2 = ConcreteObserver()
    subject.attach(observer: observer2)
    
    subject.notyfi(value: "some string")
}

// Version with more modern syntax
/*
protocol Observer<ValueType> {
    associatedtype ValueType
    func update(value: ValueType)
}

struct Subject<T> {
    private var observers = Array<any Observer<T>>()

    mutating func attach(observer: any Observer<T>) {
        observers.append(observer)
    }

    func notify(value: T) {
        for observer in observers {
            observer.update(value: value)
        }
    }
}
*/

```
-->
