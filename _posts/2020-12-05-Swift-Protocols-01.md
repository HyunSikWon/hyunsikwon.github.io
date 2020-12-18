---
title: Swift - 프로토콜!!
layout: single
comments: true
share: true
categories:
- Swift
tag:
- Protocols
last_modified_at: 2020-12-18 T23:18:08+08:00
---
iOS 개발 공부를 하면 할수록, swift의 중요성을 느낀다. (swift로 개발하니 당연한 이야기지만..). UIkit Framework만 잘 사용하면 되고 언어는 어느정도만 알면 되지,, 라는 안일한 생각을 했었는데, 바보같은 생각은 집어 치우고 다시한번 Swift를 전반적으로 공부하려 한다. 순서에 상관없이, 공부하며 접하는 것들을 [docs.swift.org](http://docs.swift.org) 를 통해 공부해보자. 오늘은 프로토콜!

## 프로토콜이란?

프로토콜은 특정 기능 수행에 필수적인 요소를 정의한 청사진(blueprint)이다. 구조체, 클래스, 열겨형은 프로토콜을 채택(Adopted)해서 특정 기능을 실행하기 위한 프로토콜의 요구사항을 실제로 구현할 수 있다. 어떤 프로토콜의 요구사항을 모두 따르는 타입은 '해당 프로토콜을 준수한다(Conform)'고 표현한다. 

### 채택(Adopt)와 준수(Conform)의 차이는?

The Swift Language Guide에 다음과 같은 설명이 잇다:

> If a type already conforms to all of the requirements of a protocol, but has not yet stated that it adopts that protocol, you can make it adopt the protocol with an empty extension:

```swift
struct Hamster {
    var name: String
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}
extension Hamster: TextRepresentable {}
```

유추해보면 준수(conform)은 protocol이 정의한 요구 사항을 구현하는 것이고, 채택(adopt)은 `:` 뒤에 준수할 프로토콜을 명시하는 것이다.

## Protocol Syntax

프로토콜의 정의는 클래스, 구조체 등과 유사하다.

```swift
protocol SomeProtocol {
    // protocol definition goes here
}
```

프로토콜을 채택하기 위해서는 타입 이름 뒤에 `:` 을 붙인 후 준수할 프로토콜 이름을 적는다. 여러 프로토콜을 준수하기 위해서는 `,` 로 구분하여 명시한다.

```swift
struct SomeStructure: FirstProtocol, AnotherProtocol {
    // structure definition goes here
}
```

서브클랙스의 경우 슈퍼클래스를 프로토콜 앞에 적어 준다.

```swift
class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // class definition goes here
}
```

## Property Requirements

프로토콜은 채택한 타입이 어떤 프로퍼티를 구현해아하는지 요구할 수 있다. 프로퍼티가 계산 프로퍼티인지 저장 프로퍼티인지 정확하게 명시하지는 않고, 단지 프로퍼티의 이름과 타입만 명시할 뿐이다.  또한 프로퍼티카 gettable 혹은 gettable&settable 인지 명시한다.

만약 프로토콜이 프로퍼티를 gettable&settable로 명시한 경우에는, 프로퍼티는 상수 저장 프로퍼티나 read-only 계산 프로퍼티가 될 수 없다. 만약 프로퍼티가 gettable 프로퍼티로 명시한 겨우에는 어떤 방식으로든 프로퍼티를 구현할 수 있다. 

프로퍼티 요구사항은 언제나 `var` 키워드와 함께 변수 프로퍼티로 정의한다. Gettable&settable 프로퍼티는 타입 선언 뒤에  `{ get set }` 을 작성하여 지정할 수 있고, gettable 프로퍼티는 `{ get }` 을 작성하면 된다.

```swift
protocol SomeProtocol {
    var mustBeSettable: Int { get set }
    var doesNotNeedToBeSettable: Int { get }
}
```

타입 프로퍼티를 프로토콜에 정의하기 위해선  `static` 키워드를 사용한다. 타입 프로퍼티는 `class` 키워드 혹은  `static` 키워드와 함께 사용되는데, 프로토콜에서는 이 둘을 구분하지 않고 모두 `static` 키워드로 타입 프로퍼티 요구사항을 정의한다.

```swift
protocol AnotherProtocol {
    static var someTypeProperty: Int { get set }
}
```

다음은 단일 인스턴스 프로퍼티 요구사항을 정의한 프로토콜이다.

```swift
protocol FullyNamed {
    var fullName: String { get }
}
```

```swift
struct Person: FullyNamed {
    var fullName: String
}
let john = Person(fullName: "John Appleseed")
// john.fullName is "John Appleseed"
```

`Person` 구조체는 타입을 선언하며 `FullyNamed` 프로토콜을 채택했다. `Person` 구조체의 인스턴스는 저장 프로퍼티인 문자열 타입의  `fullName`을 갖게되며, 이는 `FullyNamed` 프로토콜의 프로퍼티 요구사항을 정확히 준수했다는 것을 보여준다. 만약 프로토콜의 요구사항을 준수하지 않는다면 complie 에러가 표시된다.

다음은 조금더 복잡한 코드를 가진 `FullyNamed` 프로토콜을 채택하고 준수한 `Starship` 클래스이다.

```swift
class Starship: FullyNamed {
    var prefix: String?
    var name: String
    init(name: String, prefix: String? = nil) {
        self.name = name
        self.prefix = prefix
    }
    var fullName: String {
        return (prefix != nil ? prefix! + " " : "") + name
    }
}
var ncc1701 = Starship(name: "Enterprise", prefix: "USS")
// ncc1701.fullName is "USS Enterprise"
```

이 클래스는 `fullName` 프로퍼티를 read-only 계산 프로퍼티로 선언했다. `Starship` 클래스의 인스턴스는 `name`과 optional `prefix` 프로퍼티를 갖으며, `fullName`은 이 값들을 사용한다.

## Method Requirements

프로토콜은 자신을 준수하는 타입이 인스턴스 메소드 혹은 타입 메소드를 준수하도록 요구할 수 있다. 이 메소드는 프로토콜 정의의 일부로 보통의 인스턴스 혹은 타입 메소드와 거의 같은 방식으로 작성된다. 하지만 `{ }`와 메소드의 몸체부분은 제외한다. 또한, 프로토콜에서 정의하는 메소드는 파라미터의 default 값을 지정할 수 없다. 

프로퍼티 요구사항을 정의하는 것처럼 타입 메소드는 `static` 키워드를 통해 정의하고, 이 역시 `class` 혹은 `static` 키워드로 구현되는 타입 메소드를 따로 구분하지 않는다.

```swift
protocol SomeProtocol {
    static func someTypeMethod()
}
```

다음 예제는 단일 인스턴스 메소드 요구사항을 정의한 프로토콜이다

```swift
protocol RandomNumberGenerator {
    func random() -> Double
}
```

이 프로토콜을 준수하는 모든 타입은 `Doulbe` 값을 리턴하는 `random` 메소드를 가져야 한다. 이 프로토콜은 어떤식으로 랜덤 값이 만들어지는 지에 대한 내용은 없이, 단순히 랜덤 값을 생성하는 메소드를 요구할 뿐이다.

다음 클래스는 `RandomNumberGenerator` 프로토콜은 채택하고 준수하였고, 이 클래스는 *linear congruential generator*로 알려진 알고리즘을 구현한다.

```swift
class LinearCongruentialGenerator: RandomNumberGenerator {
    var lastRandom = 42.0
    let m = 139968.0
    let a = 3877.0
    let c = 29573.0
    func random() -> Double {
        lastRandom = ((lastRandom * a + c)
            .truncatingRemainder(dividingBy:m))
        return lastRandom / m
    }
}
let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// Prints "Here's a random number: 0.3746499199817101"
print("And another one: \(generator.random())")
// Prints "And another one: 0.729023776863283"
```

## Mutating Method Requirements

때때로 메소드는 해당 타입의 인스턴스를 수정해야할 필요가 있다. 예를들어, 값 타입(strucutres 혹은 enumerations)에서는 `mutating` 키워드를 `func` 앞에 위치시켜서, 함께 속한 어떤 인스턴스 혹은 프로퍼티를 수정한다는 것을 알려야한다.

어떠한 타입이든 인스턴스의 값을 바꾸는 인스턴스 메소드 요구하는 프로토콜을 정의한다면 `mutating` 키워드를 프로토콜을 정의할 때 사용해야한다. 이는 구조체나 열겨형이 이 프로토콜을 채택할 때, 메소드 요구사항을 잘 이행할 수 있도록 한다. 참조 타입인 클래스는 `mutating` 키워드를 명시하지 않아도 된다.

```swift
protocol Togglable {
    mutating func toggle()
}

enum OnOffSwitch: Togglable {
    case off, on
    mutating func toggle() {
        switch self {
        case .off:
            self = .on
        case .on:
            self = .off
        }
    }
}
var lightSwitch = OnOffSwitch.off
lightSwitch.toggle()
// lightSwitch is now equal to .on
```

## Initializer Requirements

프로토콜은 특정한 initializer도 요구사항으로 정의할 수 있다. 보통의 initailizer를 작성하는 방법과 동일하지만, `{ }` 와 구현부는 작성하지 않는다.

```swift
protocol SomeProtocol {
    init(someParameter: Int)
}
```

### Class Implemetations of Protocol Initializer Requirements

프로토콜 이니셜라이저 요구사항을 클래스에서 지정 이니셜라이저와 편의 이니셜라이저로 준수하여 수행할 수 있다. 두 경우 모두 `required` 식별자를 통해 구현한다. 

```swift
class SomeClass: SomeProtocol {
    required init(someParameter: Int) {
        // initializer implementation goes here
    }
}
```

만약 서브클래스가 수퍼클래스로부터 지정 이니셜라이저를 오버라이드 하면서 프로토콜의 이니셜라이져 요구사항을 수행한다면, `required` 와 `ovreride` 식별자를 모두 표기해야 한다.

```swift
protocol SomeProtocol {
    init()
}

class SomeSuperClass {
    init() {
        // initializer implementation goes here
    }
}

class SomeSubClass: SomeSuperClass, SomeProtocol {
    // "required" from SomeProtocol conformance; "override" from SomeSuperClass
    required override init() {
        // initializer implementation goes here
    }
}
```

### Failable Initializer Requirements

프로토콜은 실패 가능한 이니셜라이져를 정의할 수 있다. 프로토콜을 채택한 타입에서 실패 가능한 이니셜라이져를 구현하거나, 보통의 이니셜라이저를 구현함으로써 프로토콜을 준수할 수 있다.

## Protocols as Types

프로토콜은 어떠한 기능도 실제로 수행하지 않는다. 그럼에도 불구하고, 프로톨을 완전히 독립된 타입으로 사용할 수 있는데, 프로토콜을 타입으로 사용하는 것을 *existential type* 이라고 부르고, 이는 "어떤 타입 T가 있고, T는 프로토콜을 준수한다." 라는 의미이다.

프로토콜은 다른 타입들처럼 많은 곳에서 사용할 수 있다.

- 함수, 메소드, 이니셜라이져의 파라미터 타입 혹은 리턴 타입으로 사용될 수 있다.
- 상수, 변수, 프로퍼티의 타입으로 사용될 수 있다.
- 배열, 딕셔너리 혹은 다른 컨테이너의 항목 타입으로 사용될 수 있다.

예제코드:

```swift
class Dice {
    let sides: Int
    let generator: RandomNumberGenerator
    init(sides: Int, generator: RandomNumberGenerator) {
        self.sides = sides
        self.generator = generator
    }
    func roll() -> Int {
        return Int(generator.random() * Double(sides)) + 1
    }
}

var d6 = Dice(sides: 6, generator: LinearCongruentialGenerator())
for _ in 1...5 {
    print("Random dice roll is \(d6.roll())")
}
// Random dice roll is 3
// Random dice roll is 5
// Random dice roll is 4
// Random dice roll is 5
// Random dice roll is 4
```

`Dice` 인스턴스는 주사위의 면의 수를 나타내는 정수형  `sides` 프로퍼티와 랜덤한 숫자를 만드는 `generater` 프로퍼티를 갖게된다. `generator` 는 `RandomNumberGenerator` 타입의 프로퍼티이므로, 이 프로퍼티에 할당할 모든 인스턴스는 `RandomNumberGenerator` 프로토콜을 채택해야 한다. 다만, `generator` 의 기본 타입에 정의된 메소드와 프로퍼티는 사용할 수 없고, 다운캐스트를 통해서 사용해야 한다.

이니셜라이져에도 `RandomNumberGenerator` 타입을 준수하는 모든 값을 파라미터로 전달할 수 있다. `Dice` 는 또한 `roll()` 인스턴스 메소드를 제공하는데, `generator` 는 `RandomNumberGenerator`를 채택하기 때문에, `random()` 호출되는 것을 보장할 수 있다.

## Delegation

*Delegation*은 클래스나 구조체가 자신의 책임을 다른 타입의 인스턴스에 넘겨줄 수 있게끔 하는 디자인 패턴이다. 이 디자인 패턴은 위임할 책임(요구사항)을 캡슐화하는 프로토콜을 정의함으로써 수행된다. 이 프로토콜을 채택하는 타입은 위임자가 되어 위임받은 기능들을 제공하는 것을 보장한다. *Delegation*은 특정 행동에 대한 반응을 하기 위해서나, 외부 자원의 타입을 알 필요 없이 외부 자원으로부터 데이터를 회수하기 위해 사용된다.

자세한 내용은 여기서..

[Delegate Pattern에 대해 공부하고 정리한 글.](https://hyunsikwon.github.io/swift/Swift-delegate/)

## Adding Protocol Conformance with an Extension

프로토콜과 `extension`을 사용하면 소스코드에 접근할 필요 없이, 이미 존재하는 타입을 새로운 프로토콜을 채택하고 준수할 수 있게 확장할 수 있다. `extension`은 새로운 프로퍼티, 메소드, 서브스크립트를 이미 존재하는 타입에 추가할 수 있고, 따라서 프로토콜이 요구하는 모든 요구사항들을 추가할 수 있다. 

```swift
protocol TextRepresentable {
    var textualDescription: String { get }
}
extension Dice: TextRepresentable {
    var textualDescription: String {
        return "A \(sides)-sided dice"
    }
}

let d12 = Dice(sides: 12, generator: LinearCongruentialGenerator())
print(d12.textualDescription)
// Prints "A 12-sided dice"
```

### Conditionally Conforming to a Protocol

제네릭 타입은 프로토콜의 요구사항이 특정한 조건에서만 - 타입의 제네릭 파라미터가 프로토콜을 준수하는 것과 같은 경우 - 만족할 수 있도록 할 수 있다. 이는 채택할 프로토콜 이름 뒤에 제네릭 `where` 절을 작성하여 구현할 수 있다.

```swift
extension Array: TextRepresentable where Element: TextRepresentable {
    var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
    }
}
let myDice = [d6, d12]
print(myDice.textualDescription)
// Prints "[A 6-sided dice, A 12-sided dice]"
```

### Declaring Protocol Adoption with an Extension

만약 이미 프로토콜의 요구사항을 준수하고 있고, 채택은 하지 않은 상태라면, 빈 `extension`과 함께 프로토콜을 채택할 수 있다.

```swift
struct Hamster {
    var name: String
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}
extension Hamster: TextRepresentable {}
```

## Collections of Protocol Types

프로토콜은 배열이나 딕셔너리같은 컬렉션 타입에 저장되어 사용될 수 있다. 

```swift
let things: [TextRepresentable] = [game, d12, simonTheHamster]
```

이를통해 배열의 아이템을 이용해 반복문을 수행할 수 있다. 

```swift
for thing in things {
    print(thing.textualDescription)
}
// A game of Snakes and Ladders with 25 squares
// A 12-sided dice
// A hamster named Simon
```

## Protocol Inheritance

프로토콜은 하나 이상의 다른 프로토콜을 상속하고, 추가적인 요구사항을 추가할 수 있다. 클래스 상속과 유사한 방식으로 상속하고 여러 프로토콜을 상속할 경우 `,` 로 구분한다.

```swift
protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
    // protocol definition goes here
}
```

```swift
protocol PrettyTextRepresentable: TextRepresentable {
    var prettyTextualDescription: String { get }
}
```

이 예제 코드는 새로운 프로토콜인 `PrettyTextRepresentable`를 정의하고, 이 프로토콜은 `TextRepresentable`을 상속받았다. `PrettyTextRepresentable` 채택하는 모든 타입은 `TextRepresentable`에 의해 정의된 요구사항을 만족해야하고, 이와 함께 `PrettyTextRepresentable`의 추가적인 요구사항도 만족해야한다. `PrettyTextRepresentable` 는 한 개의 요구사항을 추가했고, 이는 gettable 프로퍼티인 `prettyTextualDescription`이며, 이는 문자열을 반환한다.

## Class-Only Protocols

`AnyObject` 프로토콜을 상속 리스트로 추가하여 클래스 타입만 프로토콜을 채택할 수 있도록 제한할 수 있다. 

```swift
protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
    // class-only protocol definition goes here
}
```

예제 코드에서 `SomeClassOnlyProtocol`는 오직 클래스 타입만 채택할 수 있고, 구조체나 열겨형에서 이 프로토콜을 채택하면 컴파일 에러가 나타난다.

> Class-only 프로토콜은, 프로토콜의 요구 사항에 의해 정의된 동작이, 자신을 준수하는 타입에 reference semantics가 있다고 가정하거나 요구하는 경우에만 사용한다.

 

## Protocol Composition

타입에게 여러개의 프로토콜을 통시에 준수하도록 요구하는 것이 매우 유용하다. *protocol compoistion*을 통해서 다수의 프로토콜을 하나의 요구사항으로 결합할 수 있다. 

*Protocol compoistion*은 `SomeProtocol & AnotherProtocol` 형태를 갖는다. 필요한 모든 프로토콜을 나열할 수 있고, `&` 를 이용해 구분하면 된다. *Protocol compoistion*은 또한 하나의 클래스 타입을 포함할 수 있다. 이를 통해서 요구되는 슈퍼클래스를 명시할 수 있다.

```swift
protocol Named {
    var name: String { get }
}
protocol Aged {
    var age: Int { get }
}
struct Person: Named, Aged {
    var name: String
    var age: Int
}
func wishHappyBirthday(to celebrator: Named & Aged) {
    print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
}
let birthdayPerson = Person(name: "Malcolm", age: 21)
wishHappyBirthday(to: birthdayPerson)
// Prints "Happy birthday, Malcolm, you're 21!"
```

위 예제 코드에서 `wishHappyBirthday(to:)` 함수의 파라미터 타입은 `Named & Aged` 이다. 이는 `Named` 프로토콜과 `Aged` 프로토콜을 모두 준수한 타입이라는 의미이다. 두 프로토콜을 준수하기만 하면 어느 타입이든 상관없다.

```swift
class Location {
    var latitude: Double
    var longitude: Double
    init(latitude: Double, longitude: Double) {
        self.latitude = latitude
        self.longitude = longitude
    }
}
class City: Location, Named {
    var name: String
    init(name: String, latitude: Double, longitude: Double) {
        self.name = name
        super.init(latitude: latitude, longitude: longitude)
    }
}
func beginConcert(in location: Location & Named) {
    print("Hello, \(location.name)!")
}

let seattle = City(name: "Seattle", latitude: 47.6, longitude: -122.3)
beginConcert(in: seattle)
// Prints "Hello, Seattle!"
```

`beginConcert(in:)` 함수는 `Location & Named` 타입의 파라미터를 갖는다. 이는 `Location` 의 서버클래스이고 `Named` 프로토콜을 준수한 모든 타입을 의미한다. 

## Checking for Protocol Conformance

`is`, `as` 타입 추론 연산자를 이용하면 프로토콜 준수를 확인하고 구체적인 프로토콜로 캐스트 할 수 있다. 프로토콜의 검사와 캐스팅은 타입 검사와 캐스팅과 정확히 같은 형태의 구문이다.

- `is` 연산자는 인스턴스의 프로토콜 준수 여부에 따라서 `true` 혹은 `false` 를 리턴한다.
- `as?` 다운캐스트 연산자는 프로토콜 타입의 옵셔널 값을 반환하고, 인스턴스가 프로토콜을 준수하지 않으면 `nil`을 반환한다.
- `as!` 다운캐스트 연산자는 강제로 프로토콜 타입으로 다운캐스트하고, 만약 다운캐스팅이 실패하면 런타임 에러가 발생한다.

예제코드:

```swift
protocol HasArea {
    var area: Double { get }
}
class Circle: HasArea {
    let pi = 3.1415927
    var radius: Double
    var area: Double { return pi * radius * radius }
    init(radius: Double) { self.radius = radius }
}
class Country: HasArea {
    var area: Double
    init(area: Double) { self.area = area }
}
class Animal {
    var legs: Int
    init(legs: Int) { self.legs = legs }
}
```

`HasArea` 프로토콜과 이를 준수하는 `Circle`과 `Country` 두 개의 클래스가 있다. `Animal` 클래스는 어떠한 프로토콜도 준수하고 있지 않다.

세 개의 클래스는 어떤 기본 클래스도 공유하고 있지 않지만, 모두 클래스기 때문에 다음과 같이 `AnyObject` 타입의 배열에 초기화되어 저장될 수 있다.

```swift
let objects: [AnyObject] = [
    Circle(radius: 2.0),
    Country(area: 243_610),
    Animal(legs: 4)
]
```

반복문을 통해 각각의 클래스에 접근하여 프로토콜 준수 여부를 확인해보자:

```swift
for object in objects {
    if let objectWithArea = object as? HasArea {
        print("Area is \(objectWithArea.area)")
    } else {
        print("Something that doesn't have an area")
    }
}
// Area is 12.5663708
// Area is 243610.0
// Something that doesn't have an area
```

배열에 들어있는 객체가 `HasArea` 프로토콜을 준수할 때마다,  `as?` 연산자에 의해 반환된 옵셔널 값이 옵셔널 바인딩을 통해  `objectWithArea` 상수로 언랩핑 된다. 이 과정에서 `Circle` 과 `Country`는 계속해서 타입을 유지한다. 그러나 옵셔널 바인딩 과정에서 `objectWithArea` 에 저장되면 이 타입은 `HasArea` 가 된다.

## Optional Protocol Requirements

프로토콜에 *optional requirement*을 정의할 수 있다. 이 요구사항은 프로토콜을 준수하는 타입이 수행하지 않아도 된다. O*ptional requirement*을 위해선 프로토콜을 정의할 때 `optional` 식별자를 붙여주면 된다. 또한 O*ptional requirement* 는 Objective-C에서도 사용되는 코드를 작성할 수 있도록 해준다. 이를 위해선, 프로토콜과 *optional requirement* 에 `@objc` 속성을 표시하면 된다. 이 속성이 붙여진 프로토콜은 오직 Objective-C 클래스를 상속한 클래스와 다른 `@objc` 클래스만 채택할 수 있다.

메소드나 프로퍼티를 *optional requirement*로 정의하면, 이들의 타입은 자동적으로 옵셔널이 된다. 예를들어 `(Int) -> String`은 `((Int) -> String)?`이 된다. 메소드의 반환타입이 아니라 함수 타입 전체가 옵셔널로 랩핑된다는 점을 주의하자.

 옵셔널 프로토콜 요구사항은 옵셔널 체이닝과 함께 호출될 수 있다. 이는 프로토콜을 준수하는 타입이 요구사항을 실행하지 않았을 가능성을 위함이다. 옵셔널 체이닝을 통해서 *optional requirement*  이행 여부를 검사할 수 있다.

 `Counter` 클래스는 증가량을 외부 data source에서 가져온다. 그리고 이 data source를 두 개의 *optional requirement*을 정의한  `CounterDataSource` 프로토콜로 정의했다.

```swift
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}
```

`CounterDataSource` 프로토콜은 옵셔널 메소드 요구사항인 `increment(forCount:)` 와 옵셔널 프로퍼티 요구사항인 `fixedIncrement`를 정의한다. 이 요구사항들은 `Counter` 인스턴스에 적절한 증갸량을 제공하는 data source를 위한 두 가지의 방식을 정의한다.

> 두 요구사항이 모두 옵셔널이기 때문에 `CounterDataSource`를 준수하면서 요구사항 모두를 생략할 수 있다. 기술적으로는 가능하지만 이는 좋은 방식이라고 할 수 없다.

예제 코드:

```swift
class Counter {
    var count = 0
    var dataSource: CounterDataSource?
    func increment() {
        if let amount = dataSource?.increment?(forCount: count) {
            count += amount
        } else if let amount = dataSource?.fixedIncrement {
            count += amount
        }
    }
}

class ThreeSource: NSObject, CounterDataSource {
    let fixedIncrement = 3
}

var counter = Counter()
counter.dataSource = ThreeSource()
for _ in 1...4 {
    counter.increment()
    print(counter.count)
}
// 3
// 6
// 9
// 12
```

## Protocol Extensions

프로토콜은 메소드, 이니셜라이져, 서브스크립트 등을 준수하는 타입에 제공하기위해 확장될 수 있다. 이는 준수하는 타입이 아니라, 프로토콜 자체로 특정 동작들을 정의할 수 있게 한다.

예를들어, `RandomNumberGenerator` 프로토콜은 `random()` 메소드의 결과를 사용하여 `Bool` 타입의 값을 반환하는  `randomBool()` 을 제공하기위해 확장될 수 있다.

```swift
extension RandomNumberGenerator {
    func randomBool() -> Bool {
        return random() > 0.5
    }
}
```

이 방식을 사용하면, 해당 프로토콜을 준수하는 모든 타입이 추가적인 수정 없이 자동적으로 이 메소드를 사용할 수 있다.

```swift
let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// Prints "Here's a random number: 0.3746499199817101"
print("And here's a random Boolean: \(generator.randomBool())")
// Prints "And here's a random Boolean: true"
```

*Protocol extension*은 추가적인 수행을 타입에게 제공할 수는 있지만, extension을 이용해서 다른 프로토콜을 상속받을 수는 없다.

### Providing Default Implementations

*Protocol extension*을 사용하여 기본 구현(default implementation)을 제공할 수 있다. 특정 프로토콜을 준수하는 타입 중에서 그 프로토콜의 요구사항에 대해 자체적으로 구현한게 있으면 그것을 사용하고 아니면 기본 구현을 사용하게 된다. 즉, 프로토콜에서는 선언만 할 수 있는데 익스텐션을 이용해 기본 구현을 제공할 수 있다.

```swift
extension PrettyTextRepresentable  {
    var prettyTextualDescription: String {
        return textualDescription
    }
}
```

> 이는 *optional requirement* 과는 차이가 있다. *Protocol extension* 역시 준수하는 타입이 반드시 기본 구현 요구사항을 따르지 않아도 되지만, 옵셔널 체이닝을 사용하지 않아도 된다는 점에서 차이가 있다!

### Adding Constraints to Protocol Extensions

익스텐션을 사용하여 요구사항을 정의할 때, 준수하는 타입이 익스텐션에서 정의한 메소드나 프로퍼티를 사용하기 위한 제약조건을 명시할 수 있다. 이는 `where` 절을 이용한다.

다음은 `Collection` 프로토콜의 익스텐션은, 요소가 `Equatable` 프로토콜을 준수해야만 적용된다. 

```swift
extension Collection where Element: Equatable {
    func allEqual() -> Bool {
        for element in self {
            if element != self.first {
                return false
            }
        }
        return true
    }
}
```

다음 두 배열은 `Int` 타입의 배열이고, `Int`는 `Equatable` 프로토콜을 준수하므로 익스텐션이 적용된다. 

```swift
let equalNumbers = [100, 100, 100, 100, 100]
let differentNumbers = [100, 100, 200, 100, 200]

print(equalNumbers.allEqual())
// Prints "true"
print(differentNumbers.allEqual())
// Prints "false"
```

## Reference

[The Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html)

야곰의 Swift Programming
