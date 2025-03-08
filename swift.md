iOS App Development with Swift
==============================
https://frontendmasters.com/courses/swift-ios/
https://firtman.github.io/intro-swift/lessons/intro-to-the-course/intro

### Basic Expressions
Swift looks quite modern

#### variables
```swift
var name = "Maximiliano";
```
semicolon is optional

#### print to console
```swift
print(name)
```

you can write code outside of
function or class

#### functions
```swift
func greet() {
  // access global variable
  print(name)
}
greet()
```

#### boolean conditions don't need parenthesis
```swift
if x > 1 {}
if x > 1 && y < 2 {}
while x < 10 {
  // there is no x++
  x += 1
}
```
although you can add them if you want

#### naming guidelines
vars, constants, functions: camelCase
```swift
var name: String
let tax = 7.8
func printMessage() { }
```

data types: TitleCase
```swift
class CustomerOrder { }
enum UserType { }
protocol MyProtocol { }
struct ApiResponse { }
```

you can use emoji in identifiers

#### visibility
everything we define is accessible
by other files in the same app or framework

everything is public by default
```swift
public var visible = 1
private var invisible = 2
fileprivate var kindOfInvisible = 3
```

in practice, in swift
it's not usual to use private
or use `_` prefix for

#### fileprivate
means if you create two classes in one file
they will be able to acess their variables

```swift
class ClassA {
  fileprivate var name = "I'm A"
}

class ClassB {
  func test() {
    var object = ClassA()
    object.name
  }
}
```

#### variables
```swift
var data = 3
data = 5
data = 7
```

#### constant
`let` is for constants
*(very different than javascript)*

can be used for 
constants (defined at compile time)
immutable variables (created at run time)

actually, because of immutable vars,
in practice we will more often use `let`

```swift
let tax = 7.8
```

#### types
```swift
let name: String
```

although most of time you don't need
to type because types can be inferred
```swift
let name = "Jane"
```

type cannot be changed

#### any
this will work

```swift
let name: Any = "Jane"
name = 4
```

but it makes the language unable to know
what that type is, so we don't get support
in completions, and we need to use casting
(but there is a use case for them)

#### type inferrence
```swift
var price: Double
var otherPrice = 235.99
var aThirdPrice = 23
```

aThirdPrice will be inferred as int
to solve it use either of these 
```swift
var aThirdPrice = 23.0
var aThirdPrice: Double = 23
```
