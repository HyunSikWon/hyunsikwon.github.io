---
title: Swift - Method Dispatch
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Method Dispatch
last_modified_at: 2021-06-15 T15:00:40+08:00

---

## Method Dispatch

대부분의 객체 지향 언어들에서는 하위 클래스에서 상위 클래스의 메소드와 프로퍼티들을 오버라이드 할 수 있다. 이렇게 오버라이드를 할 경우, 프로그램은 실제 호출할 함수가 어떤 것인지 결정하는 과정이 필요한데, 이때 사용되는 메커니즘을 method dispatch 라고 부른다. 메소드 디스패치란 어떤 연산(operation)을 실행해야하는지 결정하도록 돕는 메커니즘으로, 더 정확히 말하자면 어떤 메소드 구현이 사용되어야하는지 결정하는 것이다.

## Types of Dispatch

정적 디스패치는 값, 참조타입 모두에서 지원되고 동적 디스패치는 참조타입에서만 지원된다. 이러한 이유는 동적 디스패치를 위해서는 상속이 필요한데 값 타입은 상속을 지원하지 않기 때문이다.

### **Static Dispatch(Direct Call)**

변수의 명목상 타입에 맞춰서 메소드와 프로퍼티를 참조한다. 이 경우 참조될 요소를 컴파일 타임에 결정할 수 있고, 실제로 그렇게 합니다. 컴파일 타임에 실제 호출할 함수를 결정할 수 있기 때문에 함수 호출 과정이 간단하고, 컴파일러가 이것을 최적화할 수 여지가 많아 속도가 빠르다는 특징이 있다. 그러나, 자식 클래스의 요소 호출하고 싶으면 명시적인 타입 캐스팅으로 변수를 자식 타입으로 만들어줘야 한다. 따라서 프로그램이 다형성을 활용하기 어렵게 만드는 단점이 있다.

### **Dynamic Dispatch(Indirect Call)**

변수의 실제 타입의 맞춰서 메소드와 프로퍼티를 호출한다. 코드상으로는 이것이 드러나지 않기 때문에 실제 참조될 요소는 런타임에 결정됩니다. 어떤 서브클래스가 들어와도 실제 타입에 맞는 요소를 참조하기 때문에 다형성 활용에 유리하다. 다만, 런타임에 실제 참조할 요소를 찾는 과정이 있기 때문에 Static Dispatch보다 성능상에서 손해를 보게 된다는 단점이 있다. 대부분의 OOP 언어들은 다형성을 위해 dynamic dispatch를 지원한다.

## **Table Dispatch vs** **Message Dispatch**

Table Dispatch와 Message Dispatch는 Dynamic Dispatch의 일종이다. Message Dispatch와 Table Dispatch는 메소드 리스트를 유지하고 검색하는 방법에서 차이를 보인다.

### Table Dispatch

동적 디스패치의 가장 기본적인 구현 방식이다. Table Dispatch는 테이블을 사용하는데, 이 테이블이란 함수 포인터로 이루어진 배열을 의미하며 주로 virtual table 라고 불리며, Swift에서는 witness table 라고 부른다.  모든 서브클래스는 각자의 테이블을 갖고있다. 이 테이블은 서브클래스가 재정의한 모든 메소드에 대해 다른 함수 포인터를 갖고있고 서브클래스가 새로운 메소드를 추가하면 해당 메소드에 대한 함수 포인터가 배열 끝에 추가된다. 컴파일러가 런타임에 이 테이블을 사용하여 어떤 메소드를 호출할지 결정한다.

**예제 코드:**

```swift
class ParentClass {
    func method1() {}
    func method2() {}
}
class ChildClass: ParentClass {
    override func method2() {}
    func method3() {}
}
```

이 상황에서 컴파일러는 각 클래스 고유의 테이블을 생성할 것이다. 정적 디스패치와 달리 컴파일러가 먼저 테이블로부터 메모리 주소를 읽고(read) 해당 위치로 이동(jump)해야 하므로 두번의 연산이 요구된다. 

![virtual-dispatch](https://user-images.githubusercontent.com/48352065/122001317-1d72bf00-cdeb-11eb-83fa-338b84fec648.png)

### Message Dispatch

Message Dispatch는 자기 자신이 오버라이드 하거나 새로 정의한 메소드들만 테이블에 유지한다. 대신 부모 타입으로의 포인터를 가지고 있어서, 부모 타입의 메소드들은 부모 타입에서 찾아서 실행한다. Cocoa 프레임워크에서도 KVO, 코어데이터와 같은 곳에서 자주 사용된다. 또한 이 방법은 런타임에 메소드의 기능(functionality)를 바꾸는 method swizzling을 가능하게 한다(메소드의 구현을 런타임에 동적으로 변경하는 행위를 method swizzling이라고 한다.). Objective-C 런타임은 이를 위한 인터페이스를 제공해주며, Swift에서도 마찬가지로 이를 사용할 수 있다. 

**예제 코드:**

```swift
class ParentClass {
    dynamic func method1() {}
    dynamic func method2() {}
}
class ChildClass: ParentClass {
    override func method2() {}
    dynamic func method3() {}
}
```

자식 클래스는 자신의 오버라이드 하거나 새로 정의한 메소드만 테이블에 유지하고, 상위 클래스에 대한 포인터를 가지고 있다. 메시지가 디스패치 될 때 런타임은 클래스 위계를 확인하여 실행할 메소드를 결정한다. 이 과정이 매우 느리기 때문에 성능향상을 위해 캐시를 제공하기도 한다.

![message-dispatch](https://user-images.githubusercontent.com/48352065/122001313-1ba8fb80-cdeb-11eb-83de-e30cc6102a6c.png)


## Reference

[https://www.rightpoint.com/rplabs/switch-method-dispatch-table](https://www.rightpoint.com/rplabs/switch-method-dispatch-table)

[https://developer.apple.com/swift/blog/?id=27](https://developer.apple.com/swift/blog/?id=27)

[https://medium.com/flawless-app-stories/static-vs-dynamic-dispatch-in-swift-a-decisive-choice-cece1e872d](https://medium.com/flawless-app-stories/static-vs-dynamic-dispatch-in-swift-a-decisive-choice-cece1e872d)
