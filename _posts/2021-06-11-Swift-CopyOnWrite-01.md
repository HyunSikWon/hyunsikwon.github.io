---
title: Swift - Copy on Write
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Copy on Write
last_modified_at: 2021-06-11 T22:30:00+08:00
---

Swift에는 참조타입(클래스)와 값타입(구조체, 튜플, 열거형)이 있다. 이 중 값타입은 copy semantic 즉, 값 타입을 변수에 대입(assign)하거나 합수의 매개변수로 넘겨줄때 해당 값의 데이터가 모두 '복사'된다는 특징을 가진다. 이러한 경우 동일한 두 값을 서로 다른 메모리 주소에 갖고 있게 될 것이다.

## **What is this Copy-on-Write?**

Swift에서 큰 값 타입의 데이터를 변수에 대입하거나 매개변수로 넘기게 되면 모든 복사된 데이터를 위한 메모리가 필요하므로 성능상 큰 문제가 발생할 수 있다. 이러한 문제를 최소화하기 위해서 Swift 표준 라이브러리는 배열과 같은 몇몇 값 타입에 대해 하나의 참조만 존재한다면 복사가 아니라 해당 참조 내에서 값 변경을 변경하는 메커니즘을 설계했다. 그래서 배열을 변수에 대입하거나 함수의 매개변수로 넘겨주는 것이 반드시 배열의 모든 데이터를 복사한다는 것을 의미하지 않는다.

즉, Copy-on-Write은 실제로 값을 복사하지 않고, 동일한 값을 참조하다가 데이터 변경이 발생될 시에 복사해 값을 변경하는 기법이다.

**예제 코드**

```swift

func print(address o: UnsafeRawPointer ) {
    print(String(format: "%p", Int(bitPattern: o)))
}

var array1: [Int] = [0, 1, 2, 3]
var array2 = array1

// Print with just assign
print(address: array1) //0x600000078de0
print(address: array2) //0x600000078de0

// Let's mutate array2 to see what's
array2.append(4)

print(address: array2) //0x6000000aa100

//Output
//0x600000078de0 array1 address
//0x600000078de0 값 변경 전 array2의 주소 
//0x6000000aa100 값 변경 후 array2의 주소 
```

이 코드는 Copy-on-Write이 어떻게 작동하는지 보여주는 간단한 예이다. 코드를 살펴보면 먼저 array1을 생성하고 array2에 대입했다. 이 시점에 Copy-on-Write 때문에 값이 복사되 array1과 array2는 동일한 메모리 공간을 가리키고 있게 된다. 그리고 값의 변경(mutation)이 일어날 때 값이 복사된다.

## **Implementing Copy-on-Write behavior for your custom value types**

Copy-on-Write behavior를 직접 구현할 수도 있다. 

```swift
final class Ref<T> {
    var val : T
		init(_ v : T) {val = v}
}

struct Box<T> {
    var ref: Ref<T>
		init(_ x: T) { ref = Ref(x) }

    var value: T {
        get { return ref.val }
        set {
          if (!isUniquelyReferencedNonObjC(&ref)) {
            ref = Ref(newValue)
            return
					}
          ref.val = newValue
        }
    }
}
// This code was an example taken from the swift repo doc file OptimizationTips
// Link: https://github.com/apple/swift/blob/master/docs/OptimizationTips.rst#advice-use-copy-on-write-semantics-for-large-values
```

이 샘플 코드는 참조 타입을 사용하여 generic 값 타입 T에 대한 Copy-on-Write 을 어떻게 구현하는지 보여준다. 기본적으로 참조타입을 관리하고 하나의 참조만 있는 경우가 아니라면 새로운 인스턴스를 반환하는 wrapper이다. 하나의 참조만 있다면 새로운 인스턴스를 생성하지 않고 참조타입의 값을 변형(mutate)시킬 뿐이다.

## **Conclusion**

Copy-on-Write은 Swift에서 매우 무거운(heavy) 연산인 값 타입의 복사를 최적화하는 아주 좋은 방법이다. 

## Reference

[Understanding Swift Copy-on-Write mechanisms](https://medium.com/@lucianoalmeida1/understanding-swift-copy-on-write-mechanisms-52ac31d68f2f)
