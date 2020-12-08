---
title: Swift - 열거형을 알아보자!!
layout: single
comments: true
share: true
categories:
- Swift
tag:
- Enumerations
last_modified_at: 2020-12-08 T23:00:08+08:00
---

## Enumerations

*Enumeration*은 연관 값들의 집합을 공통의 타입으로 정의하여 코드 내에서 type-safe하게 값들을 활용할 수 있도록 한다. 열거형은 다음과 같은 경우에 잘 사용될 수 있다.

- 제한된 선택지를 주고 싶을 때
- 정해진 값 외에는 입력받고 싶지 않을 때
- 예상된 입력 값이 한정되어 있을 때

## Enumeration Syntax

열거형의 기본 문법은 `enum` 키워드를 사용한다.

```swift
enum SomeEnumeration {
    // enumeration definition goes here
}
```

다음 코드는 나침반을 표현하는 `CompassPoint`이다. 

```swift
enum CompassPoint {
    case north
    case south
    case east
    case west
}
```

> Swift의 열거형 case는 C나 Objectivce-C와는 다르게 default 값으로 정수형을 갖지 않는다. 위의 예에서, `north`, `south`, `east`, `west` 는 `0,1,2,3` 을 암시하지 않는다. 각각의 항목은 그 자체가 고유의 값으로 표현된다.

다음과 같이 열거형의 항목들을 한줄에 표현하는 것도 가능하다.

```swift
enum Planet {
    case mercury, venus, earth, mars, jupiter, saturn, uranus, neptune
}
```

열거형 정의는 새로운 타입을 정의한다. 따라서 Swift의 다른 타입들과 마찬가지로 대문자로 시작해야 한다.

```swift
var directionToHead = CompassPoint.west
```

`directionToHead`는 초기화 되면서 `CompassPoint` 타입의 값으로 추론되어, 다음부터는 축약형 `.` 문법(dot syntax)을 사용할 수 있다.

```swift
directionToHead = .east
```

## Matching Enumeration Values with a Switch Statement

각각의 열거형 값을 `switch` 문으로 매칭할 수 있다.

```swift
directionToHead = .south
switch directionToHead {
case .north:
    print("Lots of planets have a north")
case .south:
    print("Watch out for penguins")
case .east:
    print("Where the sun rises")
case .west:
    print("Where the skies are blue")
}
// Prints "Watch out for penguins"
```

`switch` 문은 가능한 모든 경우를 포함해야하므로, 만약 `.west` 가 생략될 경우 `CompassPoint` 의 모든 항목들을 고려할 수 없기 때문에 컴파일 될 수 없다. 열거형의 모든 `case` 를 제공하는 것이 적절하지 않은 경우에는, `default`를 사용하여 일괄적으로 처리한다.

```swift
let somePlanet = Planet.earth
switch somePlanet {
case .earth:
    print("Mostly harmless")
default:
    print("Not a safe place for humans")
}
// Prints "Mostly harmless"
```

## Iterating over Enumeration Cases

열거형을 사용하는 상황에서, 가끔은 열거형의 모든 case의 collection이 필요할 때가 있다. 이를 위해선 열거형의 이름 뒤에 `: CaseIterable` 을 적어주면 된다. 이후에는 `allCases` 프로퍼티를 통해서 접근할 수 있다.

```swift
enum Beverage: CaseIterable {
    case coffee, tea, juice
}
let numberOfChoices = Beverage.allCases.count
print("\(numberOfChoices) beverages available")
// Prints "3 beverages available"

for beverage in Beverage.allCases {
    print(beverage)
}
// coffee
// tea
// juice
```

## Associated Values

때때로 열거형의 각 case는 고유의 값과 함께 연관 값을 가질 수 있다. 연관 값의 타입은 각각의 case마다 다를 수 있다. 

예를들어, 재고 추적 시스템이 두 가지 타입의 바코드를 추적할 필요가 있다고 가정하자. 몇몇 제품은 0~9를 사용하는 UPC 포멧의 1D 바코드가 부착되어 있고, 바코드는 한자리의 시스템 숫자,  5자리 숫자의 제조사 코드, 5자리 숫자의 제품 코드, 코드 스캔이 정확히 이루어 졌는지를 나타내는 1자리의 검사 숫자로 이루어져 있다. 

![barcode_UPC_2x](https://user-images.githubusercontent.com/48352065/101494096-416fc680-39aa-11eb-9a84-e8221befb0dc.png)

다른 제품에는 QR code 포켓의 2D 바코드가 부착되어 있다. 이 바코드는 ISO 8859-1 문자를 사용하고, 이는 2953자의 문자열로 인코딩된다.

![barcode_QR_2x](https://user-images.githubusercontent.com/48352065/101494092-3fa60300-39aa-11eb-96d1-16939b20b978.png)

Swift 열거형을 통해 각 타입의 제품 바코드를 정의하면 다음과 같다:

```swift
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}
```

각각의 case는 `(Int, Int, Int, Int)`타입 연관 값을 갖는 `upc` 와, `String`타입 연관 값을 갖는 `qrCode` 이다.

여기서 각각의 연관 값에 실제 값이 정의되어있지는 않고, 단지 연관 값의 타입만 알려주고 있을 뿐이다. 이제 새로운 바코드를 다음과 같이 생성할 수 있다:

```swift
var productBarcode = Barcode.upc(8, 85909, 51226, 3)
```

또한 같은 제품에 **다른 타입**의 바코드를 할당할 수도 있다. 

```swift
productBarcode = .qrCode("ABCDEFGHIJKLMNOP")
```

앞서 봤던 것 처럼, `switch` 문을 이용하여 바코드 타입을 체크할 수 있다. 그리고 이번에는, 연관 값들을 `switch`문 안에서 추출해보자. 각각의 연관 값들은 상수 혹은 변수로 사용될 수 있다.

```swift
switch productBarcode {
case .upc(let numberSystem, let manufacturer, let product, let check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
case .qrCode(let productCode):
    print("QR code: \(productCode).")
}
// Prints "QR code: ABCDEFGHIJKLMNOP."
```

여기서 연관 값들이 모두 상수이거나 혹은 모두 변수이면, 다음과 같이 하나의 `var` 혹은 `let` 을 사용하면 된다.

```swift
switch productBarcode {
case let .upc(numberSystem, manufacturer, product, check):
    print("UPC : \(numberSystem), \(manufacturer), \(product), \(check).")
case let .qrCode(productCode):
    print("QR code: \(productCode).")
}
// Prints "QR code: ABCDEFGHIJKLMNOP."
```

## Raw Values

연관 값을 대체하여 열거형의 각각의 case는 같은 타입의 원시 값을 가질 수 있다. 원시 값은 열거형의 선언에서 **유일해야 하며 중복되면 안된다.**

```swift
enum ASCIIControlCharacter: Character {
    case tab = "\t"
    case lineFeed = "\n"
    case carriageReturn = "\r"
}
```

### Implicitly Assigned Raw Values

열거형이 정수형이나 문자열 타입의 원시 값을 갖는다면, 각각의 case에 원시 값을 할당하지 않아도 된다. Swift는 자동적으로 값을 할당해준다.

예를들어 정수형 원시값을 사용할 때, 모든 값은 이전의 값보다 1만큼 크게 할당된다. 만약 첫 case에 원시 값이 할당되지 않았다면, 0으로 할당된다. 

```swift
enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
}
```

`Planet.venus`의 원시 값은 `2` 이고 오름차순으로 할당된다.

문자열을 원시 값으로 사용하면 각각의 case의 이름이 원시 값으로 사용된다.

```swift
enum CompassPoint: String {
    case north, south, east, west
}
let earthsOrder = Planet.earth.rawValue
// earthsOrder is 3

let sunsetDirection = CompassPoint.west.rawValue
// sunsetDirection is "west"
```

### Initializing from a Raw Value

만약 원시 값과 함께 열거형을 정의하였다면, 열거형은 `rawValue` 파라미터로 원시 값을 받아서 자동적으로 초기화 하고, 해당 case 혹은 `nil` 을 반환한다. 

```swift
let possiblePlanet = Planet(rawValue: 7)
// possiblePlanet is of type Planet? and equals Planet.uranus
```

`rawValue` 파라미터로 전달된 원시 값과 일치하는 case가 없을 수 있기 때문에, 언제나 옵셔널 타입의 case가 반환된다.

```swift
let positionToFind = 11
if let somePlanet = Planet(rawValue: positionToFind) {
    switch somePlanet {
    case .earth:
        print("Mostly harmless")
    default:
        print("Not a safe place for humans")
    }
} else {
    print("There isn't a planet at position \(positionToFind)")
}
// Prints "There isn't a planet at position 11"
```

## Recursive Enumerations

*Recursive enumeration*은 하나 이상의 case에서 **다른 열거형 인스턴스를 연관 값으로 갖는 것**을 말한다. `indirect` 를 작성하여 해당 case가 재귀적이라는 것을 명시한다.

```swift
enum ArithmeticExpression {
    case number(Int)
    indirect case addition(ArithmeticExpression, ArithmeticExpression)
    indirect case multiplication(ArithmeticExpression, ArithmeticExpression)
}

```

연관 값을 갖는 모든 case에  `indirect` 을 명시하고 싶을 땐, 선언부에 한번만 작성해주어도 된다.

```swift
indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}  
```

`ArithmeticExpression`은 세 가지의 수식을 저장한다: 숫자, 두 수식의 합, 두 수식의 곱셈. `addition`, `multiplication`의 경우, 연관 값으로 두 개의 `ArithmeticExpression`를 갖는데, 이는 중첩된 수식을 계산할 수 있다. 예를들어, `(5 + 4) * 2` 는 좌측의 더하기 수식과 우측의 숫자의 곱셉을 수행한다.

```swift
let five = ArithmeticExpression.number(5)
let four = ArithmeticExpression.number(4)
let sum = ArithmeticExpression.addition(five, four)
let product = ArithmeticExpression.multiplication(sum, ArithmeticExpression.number(2))
```

재귀 함수의 경우 재귀적인 구조를 갖는 데이터를 처리하기 위해 좋은 방식이다. 

```swift
func evaluate(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .number(value):
        return value
    case let .addition(left, right):
        return evaluate(left) + evaluate(right)
    case let .multiplication(left, right):
        return evaluate(left) * evaluate(right)
    }
}

print(evaluate(product))
// Prints "18"
```

## Reference

[The Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html)
