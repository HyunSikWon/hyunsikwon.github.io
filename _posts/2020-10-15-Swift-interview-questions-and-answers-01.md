---

title: Swift Interview Questions and Answers - Beginner

layout: single

comments: true

share: true

categories:

- Swift

tag:

- Interview

last_modified_at: 2020-10-15 T21:48:00+08:00

---

**Raywenderlich**에서 Swift 언어와 관련한 면접 공부를 하며 정리한 글 입니다. **Beginner, Intermediate, Advanced** 3단계로 나누어져 있으며 각각의 단계에는 **writtend questions**과 **verbal questions** 두 가지 타입의 질문이 있습니다.

오늘은 Beginner에 해당하는 질문만 정리하겠습니다.

## Beginner Writtend Questions

### #1

```swift
struct Tutorial {
  var difficulty: Int = 1
}

var tutorial1 = Tutorial()
var tutorial2 = tutorial1
tutorial2.difficulty = 2
```

`tutorial1.difficulty`의 값과 `tutorial2.difficulty` 의 값은 무엇인가? `Tutorial` 이 class 인 겨우에는 어떤 차이가 있는가?

**answer:** 

`tutorial1.difficulty` 의 **값은 1**이고, `tutorial2.difficulty` 의 **값은 2**이다.

Swift의 `Structure`는 **value type**으로 참조 대신 값에 의해 value type을 복사한다. 다음의 코드는 `tutorial1` 의 복사본을 생성하여 `tutorial2` 에 할당한다.

```swift
var tutorial2 = tutorial1
```

`tutorial2` 의 변화는 `tutorial1` 에 반영되지 않는다.

다만, `Tutorial` 이 class 라면 `tutorial1.difficulty` 와  `tutorial2.difficulty` 모두 값이 2가 될 것이다. Swift의 `Class` 는 **reference type**이기 때문에 `tutorial1`의 프로퍼티 값을 바꾸면  `tutorial2` 에도 반영 될 것이다.

### #2

다음과 같이 `view1` 을 `var` 로 선언하고  `view2` 을 `let` 으로 선언했다. 무슨 차이가 있는가? 마지막 라인은 컴파일 되는가? 

```swift
import UIKit

var view1 = UIView()
view1.alpha = 0.5

let view2 = UIView()
view2.alpha = 0.5 // Will this line compile?
```

**answer:**

**마지막 라인의 코드는 컴파일 된다.** `view1` 은 변수로 새로운 `UIView` 의 인스턴스를 재할당 할 수 있다. 반면에 `let` 으로 선언한다면 값을 오직 한번만 할당할 수 있기 때문에 다음 코드는 컴파일되지 않을 것이다.

```swift
view2 = view1 // Error: view2 is immutable
```

그러나 `UIView` 는 참조 구문인 class기 때문에 `view2` 의 프로퍼티를 수정할 수 있다. - 따라서 다음의 코드는 컴파일 된다.

```swift
let view2 = UIView()
view2.alpha = 0.5 
```

### #3

다음은 배열을 알파벳 순으로 정렬하는 코드다. closure를 최대한 단순화 시켜라.

```swift
var animals = ["fish", "cat", "chicken", "dog"]
animals.sort { (one: String, two: String) -> Bool in
    return one < two
}
print(animals)
```

**answer:**

타입 추론 시스템이 자동적으로 클로저의 파라미터와 리턴 값의 타입을 계산하기 때문에 이들을 제거할 수 있다.

```swift
animals.sort {(one, two) in return one < two }
```

파라미터의 이름을 `$i` 로 대신할 수 있다.

```swift
animals.sort {return $0 < $1}
```

Single statement 클로저에서 `return` 키워드를 생략할 수 있다. 마지막 statment가 클로저의 리턴 값이 된다.

```swift
animals.sort {$0 < $1}
```

마지막으로 Swift는 배열의 원소들이 `Equatable` 프로토콜을 채택하는 것을 알고있다.

```swift
animals.sort(by: <)
```

### #4

이 코드는 `Adress` 와 `Person` 클래스를 생성하고 Ray, Brian 이라는 두 개의 `Person` 인스턴스를 생성한다.

```swift
class Address {
  var fullAddress: String
  var city: String
  
  init(fullAddress: String, city: String) {
    self.fullAddress = fullAddress
    self.city = city
  }
}

class Person {
  var name: String
  var address: Address
  
  init(name: String, address: Address) {
    self.name = name
    self.address = address
  }
}

var headquarters = Address(fullAddress: "123 Tutorial Street", city: "Appletown")
var ray = Person(name: "Ray", address: headquarters)
var brian = Person(name: "Brian", address: headquarters)
```

Brian이 다른 빌딩으로 이동했다고 가정하자. 따라서 Brain의 기록을 다음과 같이 바꾸자.

```swift
brian.address.fullAddress = "148 Tutorial Street"
```

에러없이 컴파일 되지만 Ray의 주소를 검사해보면 Ray이 주소도 바꼈다는 것을 알 수 있다.

```swift
print (ray.address.fullAddress)
```

무엇 때문에 이런일이 생기는가? 어떤 방식으로 고칠 수 있나?

**answer:**

`Address` 는 class기 때문에 참조 구조이다. 따라서 `headquarters` 는 동일한 인스턴스이고 `ray`, `brian` 모두에서 접근할 수 있다. `headquarters` 의 주소를 변경하면 두명의 주소가 모두 바뀔 것이다.

이 문제를 해결하기 위해서 새로운 `Address` 를 생성하여 Brian에 할당하거나, `Address` 를 struct으로 선언하면 된다.

## Beginner Verbal Questions

### #1 What is an optional and which problem do optionals solve?

Optional은 타입에 상관없이 변수 값의 부재를 허락한다. Objective-C에서 값의 부재는 `nil` 이라는 특별한 값을 통해 오직 reference type만 사용 가능했다. `int` , `float` 과 같은 value type은 이를 사용할 수 없었다.

Swift에서는 이 개념을 optional을 통해 reference type과 value type 모두 사용 가능하도록 확장하였다. Optional 변수는 값이나 `nil` 을 모두 저장할 수 있다.

### #2 Summarize the main differences between a structure and a class.

- `Class`는 상속이 가능하지만, `Sturture`는 불가능하다.
- `Class`는 reference type, `Sturture`는 value type.

### #3 What are generics and which problem do they solve?

Generic을 사용하면 **코드 중복 문제**를 해결할 수 있다. 예를들어, `Int` 타입의 파라미터를 가진 메소드가 있을 때, 같은 기능을 수행하지만  `String` 타입을 위해 중복된 코드를 작성해야 하는 경우가 존재할 것이다.

다음과 같이 같은 기능의 두 함수는 서로 다른 타입의 파라미터를 갖는다. 

```swift
func areIntEqual(_ x: Int, _ y: Int) -> Bool {
  return x == y
}

func areStringsEqual(_ x: String, _ y: String) -> Bool {
  return x == y
}

areStringsEqual("ray", "ray") // true
areIntEqual(1, 1) // true
```

반면에 generic을 사용하면 하나의 함수로 여러 타입을 파라미터로 취할 수 있다.

```swift
func areTheyEqual<T: Equatable>(_ x: T, _ y: T) -> Bool {
  return x == y
}

areTheyEqual("ray", "ray")
areTheyEqual(1, 1)
```

이 경우 **동일성을 테스트하고 있기 때문에** 파라미터를 `Equatable` 프로토콜을 채택한 타입만 가능하도록 제한하고 있다. 이를 통해 의도된 결과를 정확히 수행하고 잘못된 타입이 전달되지 않도록 예방할 수 있다.

### #4 In some cases, you can't avoid using implicitly unwrapped optionals. When? Why?

Implicitly unwrapped optionals를 사용해야 하는 가장 흔한 경우는 다음과 같다:

1. 인스턴스화 시점에서 본질적으로 프로퍼티가 `nil` 이 아닌 값으로 초기화 할 수 없는 상황. 전형적인 예시로는 Interface Builder outlet이 있다. 이는 항상 소유자가 초기화 된 후에 초기화 된다. 이 경우 outlet을 사용하기 전에 `nil` 이 아니라는 것을 보장된다.
2. 강한 참조 순환 문제를 해결하기 위해서 사용한다. 이 경우 참조의 한쪽을 `unowned` 로 표시하고 다른 한쪽은 implicitly unwrapped optional를 사용한다.

### #5 What are the various ways to unwrap an optional? How do they rate in terms of safety?

Optional를 unwrap하는 방법에는 무엇이 있는가? 각각의 경우에 안정성 측면에서 어떠한가? 

```swift
var x : String? = "Test"
```

***Forced unwrapping*** — unsafe.

```swift
let a: String = x!
```

***Implicitly unwrapped*** variable declaration — unsafe in many cases.

```swift
var a = x!
```

***Optional binding*** — safe.

```swift
if let a = x {
  print("x was successfully unwrapped and is = \(a)")
}
```

***Optional chaining*** — safe.

```swift
let a = x?.count
```

***Nil coalescing*** operator — safe.

```swift
let a = x ?? ""
```

`Guard` statement — safe.

```swift
guard let a = x else {
  return
}
```

***Optional pattern*** — safe.

```swift
if case let a? = x {
  print(a)
}
```

## Reference

[Raywenderlich](https://www.raywenderlich.com/762435-swift-interview-questions-and-answers)
