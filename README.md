
This document provides an acceptance criteria for build time sanitization, it attempts to bestow best practices and guidelines for producing optimized code without sacrificing brevity. It is a living document which is susceptible to change.

### Prefer string interpolation over concatenation

Concatenating strings via the + operator is much less efficient than returning a string literal with interpolated values, this cost is inherent with the complexities of Unicode as well as having to resolve operator overloading.

#### Prefer:
```swift
func greet(name: String) -> String {
  "Hi \(name)"
}
```
#### Avoid:
```swift
func greet(name: String) -> String {
  "Hi " + name
}
```

### Reduce amount of extensions

Extensions add 150-200 MS per extension. Common sense dictates that we should extend classes which we are not the author of rather than using this language feature as a tool for styling/grouping methods in our own codebase.

#### Prefer:
```swift
extension ObjectFromSomeSDK {
  func myMethod() { 
      // ... 
  }
}
```

#### Prefer:
```swift
final class MyViewController: UIViewController {
  // MARK: - Private
  private func render() {
    // ..
  }
}
```

#### Avoid:
```swift
final class MyViewController: UIViewController {
  // ...
}

private extension MyViewController {
    func render() {
    // ..
  }
}
```

### Break down long/multiple arithmetic expressions

Literals in Swift are used extensively to specify a value, they are however typeless until the compiler infers type information based on the kind of the literal and its surrounding context. The cost of this inference, while neglibigle, quickly adds up with the additional overhead of operator overloading which is used pervasively throughout the Standard Library. To [demonstrate](https://www.pointfree.co/episodes/ep111-designing-dependencies-modularization#t1046), copy and paste the bellow code into an Xcode project and hit cmd + R:

```swift
public let __tmp0 = 2 * 2 * 2 * 2.0 / 2 + 2
public let __tmp1 = 2 * 2 * 2 * 2.0 / 2 + 2
public let __tmp2 = 2 * 2 * 2 * 2.0 / 2 + 2
public let __tmp3 = 2 * 2 * 2 * 2.0 / 2 + 2
public let __tmp4 = 2 * 2 * 2 * 2.0 / 2 + 2
public let __tmp5 = 2 * 2 * 2 * 2.0 / 2 + 2
public let __tmp6 = 2 * 2 * 2 * 2.0 / 2 + 2
public let __tmp7 = 2 * 2 * 2 * 2.0 / 2 + 2
public let __tmp8 = 2 * 2 * 2 * 2.0 / 2 + 2
public let __tmp9 = 2 * 2 * 2 * 2.0 / 2 + 2
```
While the above example might seem contrived, it's a perfect demonstration of the compiler's current limitations. A more real world scenario would be:

```swift

let frame = CGRect(x: 0, y: 40, width: (view.frame.width + subview.frame.height) / 2, height: view.frame.height / 2)

```
This inconspicuous looking piece of code might add a whole seconds worth (it adds up) to the overall build time, prefer:

```swift

let width: CGFloat = (view.frame.width + subview.frame.height) / 2

let height: CGFloat =  view.frame.height / 2

let frame = CGRect(x: 0, y: 40, width: width, height: height)

```

### Mark classes as final and avoid optional overusage

Marking a class as final steers clear of dynamic dispatch and vtable lookup, everything is dispatched statically for an added benefit of faster build time.

#### Prefer:
```swift

final class DetailViewController: UIViewController {
  // ...
}
```

#### Avoid:
```swift
class DetailViewController: UIViewController {
  // ...
}
```
Overuse of optionals is most likely a symptom of an architectural flaw. Prefer avoiding optionals wherever it makes sense to do so, also prefer to avoid using the double equals operator == to compare an optional with a nil literal. This operation requires internally that the optional be unwrapped as well as the nil literal be resolved via its ExpressibleByNilProtocol protocol.  
