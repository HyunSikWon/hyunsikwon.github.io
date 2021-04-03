---
title: Choosing Between Structures and Classes
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Structures and Classes
last_modified_at: 2021-03-01 T21:50:00+08:00
---

## Overview

구조체와 클래스는 데이터를 저장하거나 앱 내의 동작(behavior)을 모델링하기 위해 사용한다. 이 둘은 매우 유사한 특징을 가졌는데, Apple은 구조체/클래스를 선택함에 있어 다음의 옵션들을 고려하는 것을 추천한다.
- 기본적으로는 구조체를 사용하는 것이 좋다.
- Objective-C와 상호 운용(interoperability)이 필요한 경우 클래스를 사용해라.
- 모델링한 데이터의 동일성(identity)을 통제할 필요하다면 클래스를 사용해라.
- 구조체와 프로토콜을 사용해서 공통된 기능을 공유해라.

## Choose Structures by Default

Swift의 구조체는 다른 프로그래밍 언어의 클래스에는 없는 많은 기능들을 포함한다. Swift의 구조체는 저장 프로퍼티, 계산 프로퍼티, 메소드를 포함할 수 있으며, 프로토콜을 채택할 수도 있다. Swift 표준 라이브러리와 Foundation 프레임 워크는 숫자형, 문자열, 배열, 딕셔너리 같은 여러 타입을 구조체를 통해 구현했다.

구조체를 사용하면 앱의 전체 상태에 대해 고려할 필요가 없어진다. 구조체는 클래스와 달리 값 타입(value type)이기 때문에 구조체 내에 변경이 생기는 경우에도 변경 사항을 의도적으로 앱의 흐름에 끼워넣지 않는 이상 앱 내 다른 부분에서 신경쓰지 않아도 된다. 

## Use Classes When You Need Objective-C Interoperability

데이터 처리를 위해서 Obejctive-C API를 사용하거나, 데이터 모델을 Objective-C 프레임워크에 선언된 클레스 계층내에 적용해야 한다면 클래스나 클래스 상속을 사용해야한다. 

## Use Classes When You Need to Control Identity

Swift의 클래스는 참조 타입(reference types)이기 때문에, 동일성(identity)의 개념을 가진다. 이는 서로 다른 두 클래스의 인스턴스가 저장 프로퍼티로 완전히 같은 값을 가지더라도 둘을 identity operator(===)로 비교하면 서도 다르다고 여겨지는 것을 의미한다. 또한, 앱 전체에 걸쳐 공유하는 클래스의 인스턴스를 변경하면, 변경사항은 해당 인스턴스에 대한 참조를 가지는 모든 코드에 영향을 준다. 따라서 인스턴스의 동일성을 위와 같은 방식으로 처리해야 한다면, 클래스를 사용하면 된다. 대표적으로 File Handling, Network Connection, Hardware intermeiaries (CBCentralManager - core bluetooth) 등에서 클래스가 사용된다.

> 동일성(Identity)은 조심히 다뤄야한다. 클래스 인스턴스를 앱내에서 광범위하게 공유하면, 로직상의 에러가 나기 쉽다. 여기저기서 공유된 인스턴스에 대한 결과값을 예측하기 어렵기 때문에, 코드를 더 정확하게 작성해야 한다.

## Use Structures When You Don't Control Identity

모델링한 데이터의 동일성을 관리할 필요가 없다면 구조체를 사용해라. 

예를 들어 원격 데이터베이스를 사용하는 앱이라면, 인스턴스의 동일성을 완전히 외부 개체가 제어하고 식별자를 이용해 통신한다. 만약 앱 모델의 일관성(consistency)이 서버에 저장되어 있다면, 모델을 식별자들과 함께 구조체에 저장하기만 하면된다.

```swift
struct PenPalRecord {
    let myID: Int
    var myNickname: String
    var recommendedPenPalID: Int
}

var myRecord = try JSONDecoder().decode(PenPalRecord.self, from: jsonResponse)
```

로컬에서 위의 `PenPalRecord`와 같은 모델 타입을 변경하는 것으로 예를 들 수 있다. 예를 들어 사용자 피드백으로 여러 개의 펜팔을 보여줘야 하는 경우,  `PenPalRecord` 구조체가 데이터베이스 개체의 동일성을 제어하지 않기 때문에 로컬 `PenPalRecord` 인스턴스에 대한 변경으로 인해 데이터베이스의 값이 실수로 변경될 위험이 없다.

만약 앱의 다른 부분에서 `myNickName`을 변경하고 서버로 변경 요청을 보낸다고 하더라도, `myID`가 상수로 선언되어 있어 로컬에서 변경될 수 없고, 따라서 서버 데이터베이스의 기록도 변경될 수 없을 것이다.

## Use Structures and Protocols to Model Inheritance and Share Behavior

구조체와 클래스는 모두 상속을 지원한다. 그러나 구조체와 프로토콜은 프로토콜만 채택할 수 있고 클래스를 상속하지 못한다. 그러나 구조체와 프로토콜을 함께 사용하면 클래스의 상속과 같은 계층 구조를 만들어 사용할 수 있다.

개발 초기부터 상속 관계를 설계한다면 프로토콜 상속을 추천한다. 프로토콜은 클래스, 구조체, 열거형에서도 사용할 수 있지만, 클래스는 오직 클래스만 상속할 수 있기 때문이다. 데이터를 어떻게 모델링할지 결정해야 하는 상황이라면, 프로토콜 상속을 통해 데이터 타입의 계층 구조를 설계하고, 구조체가 해당 프로토콜을 채택하는 방식을 사용해라.

## Reference

[Apple Developer](https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes)
