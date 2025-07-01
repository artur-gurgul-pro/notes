---
layout: default
title: Recipies
categories: swift
---

```swift

extension UIFont {
	var bold: UIFont {
		guard let newDescriptor = fontDescriptor.withSymbolicTraits(.traitBold) else {
		 	return self
		}
		return UIFont(descriptor: newDescriptor, size: pointSize)
	}
}
```

```swift
public struct FontStyle: ViewModifier {
    let isBold: Bool
    private let font: Font
    
    public init(isBold: Bool = false) {
        self.isBold = isBold
        self.font = self.font.bold
    }
    
    public func body(content: Self.Content) -> some View {
        content.font(font)
    }
}

extension View {
    public func fontStyle(isBold: Bool = false) -> some View {
        modifier(FontStyle(isBold: isBold))
    }
}
```


## Testing

```swift
class ExampleXCTestCase: XCTestCase {
    let app = XCUIApplication()
    
    override func tearDown() {
        app.terminate()
    }
    
    override func tearDownWithError() throws {
        app.terminate()
    }
}
```

In tests (it has to be called before `app.launch()`)

```swift
launchEnvironment["key"] = "value"
```

in the app:

```swift
ProcessInfo.processInfo.environment["key"]
```
