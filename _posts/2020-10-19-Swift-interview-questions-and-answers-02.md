---

title: Swift Interview Questions and Answers - Intermediate

layout: single

comments: true

share: true

categories:

-  Swift

tag:

- Interview

last_modified_at: 2020-10-19 T14:18:00+08:00

---

[Beginner 단계](https://hyunsikwon.github.io/swift/Swift-interview-questions-and-answers-01/)에 이어 이번에는 Intermediate 단계의 질문을 다뤄보겠습니다.

## Intermediate Written Questions

### #1

`nil` 과 `.none` 의 차이는 무엇인가?

**answer:**

차이 없다. `Optional.none` 과 `nil` 은 동일하다. 

따라서 다음의 결과는 true 다.

```swift
nil == .none
```

`nil` 을 사용하는 것이 더 보편적이고 추천된다.

### #2

다음은 class와 struct 으로 구성한 온도계의 모델이다. 컴파일러는 마지막 줄에 대해 오류를 표시할 것이다. 왜 컴파일이 실패하는가?

```swift
public class ThermometerClass {
  private(set) var temperature: Double = 0.0
  public func registerTemperature(_ temperature: Double) {
    self.temperature = temperature
  }
}

let thermometerClass = ThermometerClass()
thermometerClass.registerTemperature(56.0)

public struct ThermometerStruct {
  private(set) var temperature: Double = 0.0
  public mutating func registerTemperature(_ temperature: Double) {
    self.temperature = temperature
  }
}

let thermometerStruct = ThermometerStruct()
thermometerStruct.registerTemperature(56.0)
```

**answer:**

`ThermometerStruct` 는 내부 변수인 `temperature` 를 변경하기 위한 *mutating function* 과 함께 적절하게 선언되었다. 하지만 `let` 으로 생성된 인스턴스에서 `registerTemperature` 를 호출했기 때문에 오류가 발생한다. `let` 을 `var` 로 변경하면 된다.

Structure에서 내부 상태를 변경하는 메소드를 작성할 때 항상 `mutating` 을 표시해야 한다. 

### #3

다음 코드의 출력 값은 무엇이고? 이유는 무엇인가?

```swift
var thing = "cars"

let closure = { [thing] in
  print("I love \(thing)")
}

thing = "airplanes"

closure()
```

**answer:**

**I love cars** 를 출력할 것이다. **캡쳐 리스트**는 클로저를 선언할 때 `thing` 의 복사본을 생성할 것이다. 따라서 캡쳐된 값은 `thing` 에 새로운 값을 할당해도 변하지 않는다.

만약 클로저에 **캡쳐 리스트**를 생략한다면, 컴파일러는 복사 대신 참조를 사용할 것이다. 이로인해 클로저를 호출하면 변수에 대한 모든 변화를 반영할 것이다.

```swift
var thing = "cars"

let closure = { 
  print("I love \(thing)")
}

thing = "airplanes"

closure() // "I love airplanes"

```

### #4

여기 배열에서 중복을 제외한 값들의 수를 구하는 전역 함수가 있다.

```swift
func countUniques<T: Comparable>(_ array: Array<T>) -> Int {
  let sorted = array.sorted()
  let initial: (T?, Int) = (.none, 0)
  let reduced = sorted.reduce(initial) {
    ($1, $0.0 == $1 ? $0.1 : $0.1 + 1)
  }
  return reduced.1
}
```

`sorted()` 를 사용하기 때문에 `Comparable` 프로토콜을 채택한 타입 `T` 를 사용한다.

다음과 같이 함수를 호출할 수 있다.

```swift
countUniques([1, 2, 3, 3]) // result is 3
```

다음과 같이 함수를 사용할 수 있도록 `extension` 을 사용하여  `Array` 의 메소드를 재작성하라.

```swift
[1, 2, 3, 3].countUniques() // should print 3
```

**ansewr:** 

전역 함수 `countUniques(_:)` 를 `extension` 을 사용하여 작성할 수 있다:

```swift
extension Array where Element: Comparable {
  func countUniques() -> Int {
    let sortedValues = sorted()
    let initial: (Element?, Int) = (.none, 0)
    let reduced = sortedValues.reduce(initial) { 
      ($1, $0.0 == $1 ? $0.1 : $0.1 + 1) 
    }
    return reduced.1
  }
}
```

새로운 메소드는 제네릭 `Element` 타입이 `Comparable` 프로토콜을 채택할 때만 사용할 수 있다는 점을 명심하라.

### #5

여기 두개의 `double` 타입의 옵셔널 값을 나누는 함수가 있다. 함수 안에는 실제 나누기 계산을 하기 전에 3개의 조건식이 있다.

- 피제수는 `nil` 이 아니어야 한다.
- 제수도 `nil` 이 아니어야 한다.
- 제수는 0아 아니어야 한다.

```swift
func divide(_ dividend: Double?, by divisor: Double?) -> Double? {
  if dividend == nil {
    return nil
  }
  if divisor == nil {
    return nil
  }
  if divisor == 0 {
    return nil
  }
  return dividend! / divisor!
}
```

`guard` 를 사용하면서 강제 언래핑을 사용하지 않고 함수를 개선해라.

**answer:** 

```swift
func divide(_ dividend: Double?, by divisor: Double?) -> Double? {
		guard let a = dividend, let b = divisor, b !=0 else { return nil }

  return a / b
}
```

### #6

5번 문제를 `if let` 구문으로 바꿔라.

**answer:**

```swift
func divide(_ dividend: Double?, by divisor: Double?) -> Double? {
    if let a = dividend, let b = divisor, b != 0 {
        
        return a / b
    }
    
    return nil
}
```

## Intermediate Verbal Questions

### #1

Objective-C 에서 상수는 다음과 같이 선언한다.

```objectivec
const int number = 0;
```

Swift 에서는 상수를 다음과 같이 선언한다.

```swift
let number = 0
```

둘 사이에 어떤 차이가 있는지 설명하라.

**answer:**

`const` 는 컴파일 시점에 결정되는 값이나 표현식으로, 컴파일 시에 초기화된다.

`let` 으로 선언되는 불변 값은 런타임 시점에 결정되는 상수이다. 정적 혹은 동적 표현식으로 초기화 할 수 있다. 이는 다음과 같은 코드를 가능하게 한다

```swift
let higherNumber = number + 5
```

### #2

정적 프로퍼티나 정적 함수를 선언하기 위해서는 `static` 수식어를 사용한다. 다음은 structure을 통한 예시다.

```swift
struct Sun {
  static func illuminate() {}
}
```

class에서는 `static` 수식어와 `class` 수식어 모두 사용가능하다. 이 둘은 모두 같은 목적을 달성하지만 방식은 서로 다르다. 어떻게 다른지 설명해라.

**answer:**

`static` 은 프로퍼티나 함수를 정적으로 만들고 ***override* 할 수 없다**. `class` 를 사용하면 프로퍼티나 함수를 *override* 할 수 있다.

`class` 에 적용해보면, `static` 은 `class final` 과 같다.

예를들어, 아래 코드는 `illuminate()` 를 *override* 하기 때문에 컴파일 에러가 발생한다.

```swift
class Star {
  class func spin() {}
  static func illuminate() {}
}
class Sun : Star {
  override class func spin() {
    super.spin()
  }
  // error: class method overrides a 'final' class method
  override static func illuminate() { 
    super.illuminate()
  }
}
```

### #3

`extension` 을 사용하여 특정 타입에 저장 프로퍼티를 추가할 수 있는가? 할 수 있다면 어떻게 추가를 하고, 할 수 없다면 왜 안되는가?

**answer:**

**불가능하다.** `extension` 은 타입에 새로운 동작을 추가할 때만 사용 가능하지 타입 자체와 인터페이스를 바꾸는데 사용할 수 없다. 만약 저장 프로퍼티를 추가하면 새로운 값을 저장하기 위한 추가 메모리가 필요한데, `extension` 은 그러한 작업을 다루지 않는다.

### #4

Swift의 `protocol`에 대해서 설명하라. 

**answer:**

프로토콜은 메소드, 프로퍼티, 기타 요구사항에 대한 청사진을 정의한다. *class*, *structure*, *enumeration* 은 프로토콜을 채택하여 정의되어 있는 작업들을 수행할 수 있다.

프로토콜은 스스로 기능들을 수행하지 않고 그것의 기능을 정의만 할 뿐이다. 프로토콜을 채택하여 프로토콜에 필수 구현을 추가하거나 추가적인 기능을 더하기 위해 프로토콜을 확장(extend)하는 것이 가능하다.

## Reference

[Raywenderlich](https://www.raywenderlich.com/762435-swift-interview-questions-and-answers)
