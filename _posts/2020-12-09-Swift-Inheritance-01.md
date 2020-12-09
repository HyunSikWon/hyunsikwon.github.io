---
title: Swift - 상속!
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Inheritance
last_modified_at: 2020-12-09 T19:54:00+08:00
---

## Inheritance

클래스는 메소드, 프로퍼티 등 다른 클래스의 특성들을 상속 받을 수 있다. 어떤 클래스를 상속받는 클래스를 *subclass*, 상속을 하는 클래스를 *superclass* 라고 한다. 클래스는 슈퍼클래스에 속한 메소드, 프로퍼티, 서브스크립트를 호출하고 접근할 수 있고, 자신의 고유 메소드, 프로퍼티 등을 `override` 하여 구성할 수 있다. 또한 상속받은 프로퍼티에 프로퍼티 옵저버를 추가하여 값의 변화에 반응할 수 있다. 

## Defining a Base Class

다른 클래스를 상속받지 않는 모든 클래스는 *기본 클래스(base class)*라고 부른다. 

```swift
class Vehicle {
    var currentSpeed = 0.0
    var description: String {
        return "traveling at \(currentSpeed) miles per hour"
    }
    func makeNoise() {
        // do nothing - an arbitrary vehicle doesn't necessarily make a noise
    }
}
```

초기화 문법을 통해서 `Vehicle` 클래스의 새로운 인스턴스를 생성한다:

```swift
let someVehicle = Vehicle()
```

인스턴스를 통해서 프로퍼티에 접근할 수 있다.

```swift
print("Vehicle: \(someVehicle.description)")
// Vehicle: traveling at 0.0 miles per hour
```

## Subclassing

서브 클래스는 존재하는 클래스의 특성을 상속받고 이 특성을 정제하거나 새로운 특성을 추가할 수도 있다.

서브클래스가 슈퍼클래스를 가진다는 것은 다음과 같은 방법으로 표시한다:

```swift
class SomeSubclass: SomeSuperclass {
    // subclass definition goes here
}
class Bicycle: Vehicle {
    var hasBasket = false
}
```

새로운 `Bicycle` 클래스는 자동적으로 `currentSpeed`, `description`, `makeNoise()` 같은 모든 `Vehicle` 의 특성을 갖게 된다. 상속받은 것과 더불어, 새로운 저장 프로퍼티인 `hasBasket` 프로퍼티를 갖는다. 

서브 클래스에서 정의한 프로퍼티와 슈퍼클래스의 특성에 자유로운 수정이 가능하다.

```swift
let bicycle = Bicycle()
bicycle.hasBasket = true

bicycle.currentSpeed = 15.0
print("Bicycle: \(bicycle.description)")
// Bicycle: traveling at 15.0 miles per hour
```

서브클래스도 서브클래싱 될 수 있다.

```swift
class Tandem: Bicycle {
    var currentNumberOfPassengers = 0
}
```

`Tandem` 서브 클래스는 `Vehicle`의 모든 메소드와 프로퍼티를 상속받은 `Bicycle`의 프로퍼티와 메소드를 상속받는다. `Tandem` 서브 클래스는 고유의 새로운 프로퍼티를 정의하고, `Tandem`의 인스턴스는 새로운, 상속받은 프로퍼티를 사용할 수 있다. 

```swift
let tandem = Tandem()
tandem.hasBasket = true
tandem.currentNumberOfPassengers = 2
tandem.currentSpeed = 22.0
print("Tandem: \(tandem.description)")
// Tandem: traveling at 22.0 miles per hour
```

## Overriding

서브클래스는 슈퍼 클래스의 인스턴스 메소드, 타입 메소드, 타입 프로퍼티 혹은 서브스크립트를 재정의 할 수 있다. 이를 **overriding** 이라고 하며, `override` 키워드를 사용한다.

### Accessing Superclass Methods, Properties, and Subscripts

서브클래스에서 오버라이딩 할 때, 슈퍼클래스에서의 수행을 오버라이딩의 일부로 사용하는 것도 때때로 좋은 방법일 수 있다. 이때, `super` 접두사를 통해서 슈퍼클래스 버젼의 메소드, 프로퍼티, 서브스크립트에 접근할 수 있다.

- 오버라이드 된 메소드 `someMethod()`는 슈퍼클래스의 `someMethod()`를 `super.someMethod()` 를 통해 호출할 수 있다.
- 오버라이드 된 프로퍼티 `someProperty` 는 `super.someProperty` 를 통해 슈퍼클래스 버전에 접근할 수 있다.
- 서브스크립트도 같은 방식으로 `super[someIndex]` 로 접근할 수 있다.

### Overriding Methods

상속 받은 인스턴스 혹은 타입 메소드를 오버라이드:

```swift
class Train: Vehicle {
    override func makeNoise() {
        print("Choo Choo")
    }
}
let train = Train()
train.makeNoise()
// Prints "Choo Choo"
```

`Train`의 인스턴스가 `makeNoise()` 메소드를 호출할 경우, 서브클래스 버전의 메소드가 호출된다.

### Overriding Properties

상속된 인스턴스 혹은 타입 프로퍼티를 고유의 방식으로 정제하고 프로퍼티 옵저버를 추가할 수도 있다.

**Overriding Property Getters and Setters**

상속된 프로퍼티가 저장 프로퍼티인지 계산 프로퍼티인지 여부의 상관없이, 모두 getter(setter도 가능) 프로퍼티로 오버라이드 할 수 있다. 서브클래스는 슈퍼클래스의 프로퍼티가 저장, 계산인지 여부는 모르고 이름과 타입만 알고 있다. 따라서 오버라이드 할 때, 이름과 타입을 함께 명시해야 컴파일러가 검사할 수 있다.

상속된 읽기 전용(read-only) 프로퍼티를, *getter-setter*를 통해 읽고 쓸 수 있는(read-writed) 프로퍼티로 오버라이딩할 수 있지만, 상속된 읽고 쓸 수 있는 프로퍼티를 읽기 전용 프로퍼티로 오버라이딩 할 순 없다.

> 프로퍼티를 오버라이드 할 때, setter를 제공한다면, getter과 함께 제공해야 한다.

다음의 예제는 `Vehicle`의 서브클래스인 `Car` 클래스를 정의한다. 기본 값이 `1`인 `gear` 프로퍼티를 새로 정의하고,  `Vehicle` 클래스의 `description` 프로퍼티를 오버라이딩 한다.

```swift
class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}
```

오버라이드된 `description` 프로퍼티는 `Vehicle` 클래스의 `description` 프로퍼티를 반환하는 `super.description`에 추가적인 텍스트를 추가하였다.

```swift
let car = Car()
car.currentSpeed = 25.0
car.gear = 3
print("Car: \(car.description)")
// Car: traveling at 25.0 miles per hour in gear 3
```

**Overriding Property Observers**

상속된 프로퍼티에 프로퍼티 옵저버를 추가할 수 있다. 이를 통해서 슈퍼 클래스의 값과는 상관 없이 상속된 프로퍼티의 값의 변화를 추적할 수 있다.

> 상속된 상수 저장 프로퍼티 혹은 읽기 전용 계산 프로퍼티는 프로퍼티 옵저버를 추가할 수 없다. 이 프로퍼티의 값들은 set 할 수 없기 때문에 오버라이드의 일부로 `willSet` 혹은 `didSet` 을 제공하는 것은 적절치 않다. 또 같은 프로퍼티에 옵저버와 setter를 추가해 둘을 동시에 사용할 수 없다. 이미 setter를 설정했다면 옵저버를 붙인 것과 같은 동작을 하기 때문이다.

다음은 `Car`의 서브클래스이며 오버라이드 된 프로퍼티의 값의 변화에 따라 `gear` 의 값을 지정해준다.

```swift
class AutomaticCar: Car {
    override var currentSpeed: Double {
        didSet {
            gear = Int(currentSpeed / 10.0) + 1
        }
    }
}

let automatic = AutomaticCar()
automatic.currentSpeed = 35.0
print("AutomaticCar: \(automatic.description)")
// AutomaticCar: traveling at 35.0 miles per hour in gear 4
```

## Preventing Overrides

메소드, 프로퍼티, 서브스크립트가 오버라이드 되는 것을 방지할 수 있다. `final` 식별자를 앞에 붙여 방지하면 된다. (`final var`, `final func`, `final class`, `final subscript` )  

`final` 식별자가 붙은 것들을 오버라이드 하면 compile-time 에러가 발생한다. `extension` 에서도 `final`을 표시할 수 있다. 전체 클래스에 `final` 식별자를 붙여 클래스 전체를 상속하지 못하도록 할 수 있다.

## Reference

[docs.swift.org](https://docs.swift.org/swift-book/LanguageGuide/Inheritance.html)
