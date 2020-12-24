---
title: Swift - Initailization part3
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Initailization
last_modified_at: 2020-12-24 T20:40:00+08:00
---

# Failable Initializers

클래스, 구조체, 열거형의 인스턴스를 생성할 때, 초기화가 실패하는 경우가 발생할 수 있다. 초기화 실패는 클래스, 구조체, 열거형에 하나 이상의 failable initializers(실패 가능한 이니셜라이저)를 정의하므로써 대처할 수 있다. failable initializer는 `init` 키워드에 `?`를 붙여서 정의하면 된다 (`init?`).

> 일반적인 이니셜라이저와 failable initializer를 같은 모양(타입, 이름)으로 정의할 수 없다.

Failable initializer는 초기화하는 타입의 *optional* 값을 생성한다. 숫자형 변환 과정을 통해서 failable initializer를 알아보자. 숫자형 사이의 변환 과정에서, 값이 그대로 유지되는 것을 보장하기 위해 Swift에서는 `init(exactrly:)`를 사용할 수 있다. 만약 변환 과정에서 값이 유지될 수 없다면 초기화는 실패한다.

```swift
let wholeNumber: Double = 12345.0 // Int 타입으로 변환해도 값이 유지
let pi = 3.14159 // Int 타입으로 변환시에 값이 바뀐다.

if let valueMaintained = Int(exactly: wholeNumber) {
    print("\(wholeNumber) conversion to Int maintains value of \(valueMaintained)")
}
// Prints "12345.0 conversion to Int maintains value of 12345"

let valueChanged = Int(exactly: pi)
// valueChanged is of type Int?, not Int

if valueChanged == nil {
    print("\(pi) conversion to Int does not maintain value")
}
// Prints "3.14159 conversion to Int does not maintain value"
```

다음의 예제는 `Animal` 구조체를 정의한다. 여기에는 `String` 타입의 `species` 상수가 정의되어 있고, 한개의 파라미터를 가지는 실패 가능한 이니셜라이저도 정의한다. 이 이니셜라이저는 문자열이 비어있는지를 검사한다. 

```swift
struct Animal {
    let species: String
    init?(species: String) {
        if species.isEmpty { return nil }
        self.species = species
    }
}

// ----- 성공 -----
let someCreature = Animal(species: "Giraffe")
// someCreature is of type Animal?, not Animal

if let giraffe = someCreature { // 옵셔널 타입이므로 바인딩 과정이 필요하다.
    print("An animal was initialized with a species of \(giraffe.species)")
}
// Prints "An animal was initialized with a species of Giraffe"

// ----- 실패 -----
let anonymousCreature = Animal(species: "")
// anonymousCreature is of type Animal?, not Animal

if anonymousCreature == nil {
    print("The anonymous creature could not be initialized")
}
// Prints "The anonymous creature could not be initialized"
```

### Failable Initializers for Enumerations

실패 가능한 이니셜라이저를 적절한 열거형 케이스를 선택하는데 사용할 수 있다. 

```swift
enum TemperatureUnit {
    case kelvin, celsius, fahrenheit
    init?(symbol: Character) {
        switch symbol {
        case "K":
            self = .kelvin
        case "C":
            self = .celsius
        case "F":
            self = .fahrenheit
        default:
            return nil
        }
    }
}

let fahrenheitUnit = TemperatureUnit(symbol: "F")
if fahrenheitUnit != nil {
    print("This is a defined temperature unit, so initialization succeeded.")
}
// Prints "This is a defined temperature unit, so initialization succeeded."

let unknownUnit = TemperatureUnit(symbol: "X")
if unknownUnit == nil {
    print("This is not a defined temperature unit, so initialization failed.")
}
// Prints "This is not a defined temperature unit, so initialization failed."
```

### Failable Initializers for Enumerations with Raw Values

Raw value가 있는 열거형의 경우 자동으로 실패가능한 이니셜라이저를 가진다. 이 이니셜라이저는 `rawValue` 파라미터를 가지며 파라미터 값에 대응하는 열거형 케이스를 선택한다. 만약 대응하는 케이스가 없는 경우 초기화는 실패하게 된다.

```swift
enum TemperatureUnit: Character {
    case kelvin = "K", celsius = "C", fahrenheit = "F"
}

let fahrenheitUnit = TemperatureUnit(rawValue: "F")
if fahrenheitUnit != nil {
    print("This is a defined temperature unit, so initialization succeeded.")
}
// Prints "This is a defined temperature unit, so initialization succeeded."

let unknownUnit = TemperatureUnit(rawValue: "X")
if unknownUnit == nil {
    print("This is not a defined temperature unit, so initialization failed.")
}
// Prints "This is not a defined temperature unit, so initialization failed."
```

### Propagation of Initialization Failure

실패 가능한 이니셜라이저는 같은 클래스, 구조체, 열거형의 다른 실패 가능한 이니셜라에저에 초기화를 위임할 수 있다. 또한, 서브클래스의 실패가능한 이니셜라이저는 슈퍼클래스의 실패 가능한 이니셜라이저에 위임할 수 있다. 두 경우 모두 위임한 이니셜라이저가 실패하면 전체 초기화 과정이 즉시 실패되고 남은 코드는 수행되지 않는다.

다음의 코드에서, `CartItem`의 이니셜라이저에서 `quantitiy`가 `0`이면 초기화가 실패하여 이후의 코드는 수행되지 않고, `Product` 클래스의 이니셜라이저도 마찬가지로 `name`이 빈 문자열이면 초기화가 실패하여 즉시 종료된다.

```swift
class Product {
    let name: String
    init?(name: String) {
        if name.isEmpty { return nil }
        self.name = name
    }
}

class CartItem: Product {
    let quantity: Int
    init?(name: String, quantity: Int) {
        if quantity < 1 { return nil }
        self.quantity = quantity
        super.init(name: name)
    }
}
```

다음의 코드는 모든 초기화 과정이 실패없이 진행된다.

```swift
if let twoSocks = CartItem(name: "sock", quantity: 2) {
    print("Item: \(twoSocks.name), quantity: \(twoSocks.quantity)")
}
// Prints "Item: sock, quantity: 2"
```

다음 코드는 `CartItem`의 이니셜라이저에서 초기화가 실패한다.

```swift
if let zeroShirts = CartItem(name: "shirt", quantity: 0) {
    print("Item: \(zeroShirts.name), quantity: \(zeroShirts.quantity)")
} else {
    print("Unable to initialize zero shirts")
}
// Prints "Unable to initialize zero shirts"
```

비슷하게, 다음 코드는 `Product`의  이니셜라이저에서 초기화가 실패한다.

```swift
if let oneUnnamed = CartItem(name: "", quantity: 1) {
    print("Item: \(oneUnnamed.name), quantity: \(oneUnnamed.quantity)")
} else {
    print("Unable to initialize one unnamed product")
}
// Prints "Unable to initialize one unnamed product"
```

### Overriding a Failable Initializer

서브클래스에서 슈퍼클래스의 failable initializer를 오버라이드 할 수도 있다. Failable initializer를 서브클래스에서 일반적인 이니셜라이저로 재정의 해도 된다. 만약 이럴 경우 슈퍼클래스에 이니셜라이저로 위임하는 유일한 방법은 강제 언랩핑을 사용하는 것이다.

다음의 `Document` 클래스는 문자열과 `nil`이 될 수 있는 `name` 프로퍼티를 가진다. 다만, 이 클래스의 인스턴스는 빈 문자열을 가질 수 없다.

```swift
class Document {
    var name: String?
    // this initializer creates a document with a nil name value
    init() {}
    // this initializer creates a document with a nonempty name value
    init?(name: String) {
        if name.isEmpty { return nil }
        self.name = name
    }
}
```

다음의 `AutomaticallyNamedDocument`은  `Document`의 서브클래스이고 지정 이니셜라이저를 모두 오버라이드 했다.

```swift
class AutomaticallyNamedDocument: Document {
    override init() {
        super.init()
        self.name = "[Untitled]"
    }
    override init(name: String) {
        super.init()
        if name.isEmpty {
            self.name = "[Untitled]"
        } else {
            self.name = name
        }
    }
}
```

`AutomaticallyNamedDocument`는 빈 문자열의 값을 슈퍼클래스와는 다른 방식으로 처리하기 때문에 슈퍼클래스의 실패 가능한 이니셜라이저를 일반 이니셜라이저로 재정의 했다. 

강제 언랩핑을 사용하여 슈퍼클래스의 실패 가능한 이니셜라이저를 호출할 수 있다. 

```swift
class UntitledDocument: Document {
    override init() {
        super.init(name: "[Untitled]")!
    }
}
```

슈퍼클래스의 실패 가능한 이니셜라이저는 빈 문자열을 보낼 때만 실패하기 때문에 강제 언랩핑을 사용해도 에러가 발생지 않을 것이다.

### The init! Failable Initializer

강제 언랩핑된 옵셔널 타입의 인스턴스를 생성하는 failable Initializer를 정의할 수도 있다. 물음표 대신 느낌표를 사용하면 된다. `init!` 

# Required Initializers

`required` 수식어를 사용하여 서브클래스가 반드시 이니셜라이저를 구현하도록 강제할 수 있다.

```swift
class SomeClass {
    required init() {
        // initializer implementation goes here
    }
}
```

Required Initializers를 구현할 때, 서브클래스에서도 반드시 `required` 수식어를 붙여야 한다. Required Initializer를 재정의할 때는 `override` 할 필요는 없다.

```swift
class SomeSubclass: SomeClass {
    required init() {
        // subclass implementation of the required initializer goes here
    }
}
```

# Setting a Default Property Value with a Closure or Function

클로저나 전역함수를 사용해서 저장 프로퍼티의 기본 값을 지정할 수도 있다. 새로운 인스턴스가 초기화 될 때마다 클로저나 함수가 호출되고, 그의 리턴 값이 프로퍼티의 기본 값으로 할당된다.

이와 같은 클로저나 함수는 보통 같은 타입의 임시 값을 생성해서, 이 임시 값을 원하는 초기 상태로 조작하고 이를 리턴하는 방식으로 사용된다. 이 리턴 값이 기본 값으로 할당되는 것이다.

```swift
class SomeClass {
    let someProperty: SomeType = {
        // create a default value for someProperty inside this closure
        // someValue must be of the same type as SomeType
        return someValue
    }()
}
```

클로저의 끝에 빈 괄호가 있는 것을 주목하자. 이는 Swift에게 클로저를 즉시 실행하도록 한다. 괄호를 생략하면 클로저의 리턴 값이 아니라, 클로저 자체를 프로퍼티에 할당하려 한다.

> 만약 클로저를 사용해서 프로퍼티를 초기화 한다면, 클로저가 수행되는 시점에선 인스턴스의 나머지 것들은 아직 초기화 되지 않은 상태임을 기억해라. 즉, 클로저에서 다른 프로퍼티 값에 접근할 수 없다(기본 값을 가진 프로퍼티 일지라도). 또한 `self` 로도 접근할 수 없고 메소드도 마찬가지다.

다음의 예제는 `Chessboard` 구조체를 정의한다. 체스 보드를 표현하기 위해서, `Chessboard`는 64개의 `Bool` 값을 가진 배열  `boardColors`를 가진다. `true`는 검은색, `false`는 흰색을 나타낸다. 체스 보드는 다음과 같이 초기화 될 것이다.

```swift
struct Chessboard {
    let boardColors: [Bool] = {
        var temporaryBoard = [Bool]()
        var isBlack = false
        for i in 1...8 {
            for j in 1...8 {
                temporaryBoard.append(isBlack)
                isBlack = !isBlack
            }
            isBlack = !isBlack
        }
        return temporaryBoard
    }()
    func squareIsBlackAt(row: Int, column: Int) -> Bool {
        return boardColors[(row * 8) + column]
    }
}
```

`Chessboard` 인스턴스가 생성 될 때마다 클로저는 실행되고, `boardColors`의 기본 값이 계산되어 리턴될 것이다. 또한 `row`, `column`을 이용하여 해당 위치가 검은색인지 확인할 수 있는 함수도 구현되어 있다.

```swift
let board = Chessboard()
print(board.squareIsBlackAt(row: 0, column: 1))
// Prints "true"
print(board.squareIsBlackAt(row: 7, column: 7))
// Prints "false"
```

## Reference

[docs.swift.org](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html)
