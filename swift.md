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

#### core datatypes
```swift
String
Int
Double
Bool
```

less used
```swift
Int32
Int64
Int8
UInt
Float
```

#### strings
always created using double quotes  
(no option for single quote)

```swift
print("Hello World")
```

#### string templates
```swift
"The \(price) price is \(otherPrice * 1.1)"
```
it will convert to string

#### tuples
```swift
let coordinate: (Int, In) = (4, 5)
coordinate.1
coordinate.0
```

don't confuse for coordinate[0] (that would be array)

tuples can have different types
```swift
let city: (String, Int) = ("Minneapolis", 420_000)
print(city.0, city.1)
```

#### tupes with named segments
it's not a class, just a tuple
```swift
let state: (name: String, population: Int, isNice: Bool)
state = ("Minnesota", 5_640_000, true)
```

opposite to object arguments  
tupes when used as arguments  
are sent by value (not reference)
```swift
func printState(state: (name: String, population: Int, isNice: Bool))
state = ("Minnesota", 5_640_000, true)
prinState(state: state)
```

