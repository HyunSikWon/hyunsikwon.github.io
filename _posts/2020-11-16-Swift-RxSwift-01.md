---
title: RxSwift - Observable과 Subject 간단 요약
layout: single
comments: true
share: true
categories:
- Swift
tag:
- RxSwift
last_modified_at: 2020-11-16 T15:00:00+08:00

---



## Observable

***Observable sequence***는 보통의 ***sequence***와 같지만, 비동기적으로 동작할 수 있다는 주요한 이점이 있다.  


***Observable sequence***는 이벤트를 내보낼(emit) 수 있고, 아무것도 내보내지 않거나 이벤트를 추가로 보낼 수도 있다. **Sequence**에 값(혹은 값의 집합)이 추가되나 놓여지면, 이 값을 포함하는 이벤트를 sequence의 **observer**에 전달한다. 이 과정을 ***emitting***이라고 하고, 값들을 ***elements***라고 한다.

> 만약 에러가 발생하면, sequence는 에러 타입을 포함하는 ***error*** 이벤트를 내보낼 수 있고, 이는 sequence를 종료시킨다.

> 또한, Sequence는 정상적으로 종료될 수 있는데, 이때는 ***completed*** 이벤트를 내보낸다.

만약, sequence의 observing을 멈추고 싶다면, `dispose()` 를 호출하여 subscription을 중단할 수 있다. 이는 **KVO** 혹은 **NotificationCenter** API에서 oberver를 제거하는 것과 같다. Subscription에 **DisposeBag**를 추가하는 것으로도 중단할 수 있다.



## Subject

우리는 실시간으로 Observable에 값을 추가하고 Subscriber에게 방출하는 것이 필요하다. 이 때 Observable이자 Observer인 Subject 를 사용한다. Subject는 RxSwift의 특별한 타입으로, 다음과 같은 역할을 할 수 있다:

- **Observable sequence:** Subscribe 할 수 있다.
- **Observer:** 새로운 element를 subject에 추가하고 subejct subscriber에 내보낼 수 있다.



RxSwift에는 4가지 타입의 subject가 존재하고, 각각은 고유의 특징들을 지니고 있어, 여러 상황에서 유용하게 사용할 수 있다.

- PublishSubject
- BehaviorSubject
- ReplaySubject
- Variable *(Deprecated)

Subejct를 하나씩 살펴보자

### PublishSubject

**PublishSubject**는 오직 새로운 이벤트만 subscriber에 전달한다. 다시 말해서, 새 Subscriber가 subscribe 하기 전에 PublishSubject에 추가되는 element는 subsriber가 받지 않는다. ***Subscribe 이전의 event까지 subsriber에게 내보내는 방식을 replaying 이라고 하는데, PublishSubject는 replaying 하지 않는다.***

<img width="640" alt="S PublishSubject" src="https://user-images.githubusercontent.com/48352065/99218713-78751100-281e-11eb-8f6b-e91e4c529d13.png">

> 만약 실시간 element가 필요하다면, 새로운 subsription에서 이전 element는 상관할 바가 아니다. 오직 새로운 element만 있으면 된다. 이런 경우에 PublishSubject가 유용하다.

### BehaviorSubject

때때로, 새로운 subscriber가 seqeunce에서 가장 최근에 사용된 event를 받아야 하는 상황이 필요할 수 있다(최신 이벤트가 subscirbe 이전에 emit 되었어도).  ***BehaviorSubject***는 초기값과 함께 초기화되고, 새로운 subscriber에게 초기값 혹은 가장 최신의 element를 replay한다. 

<img width="640" alt="S BehaviorSubject" src="https://user-images.githubusercontent.com/48352065/99218715-790da780-281e-11eb-97b7-16aa0ae9fe56.png">

Subscribe 시점에 가장 최근 element가 replay되는 것을 알 수 있다.

### ReplaySubject

가장 최신의 element 뿐만 아니라 이전의 모든 element를 새로운 subscriber에 전달하고 싶을 때는 ***ReplaySubject***를 사용한다. ReplaySubject는 buffer와 함께 초기화 된다. 버퍼의 크기는 element가 추가되면 늘어난다. 모든 element는 버퍼에서 유지되고, 새로운 subscribe가 있을 때마다 replay 된다. 예를 들어, ReplaySubject는 검색 기능에서 최근 5개의 검색 목록을 나타내기 위해 사용되곤 한다.

<img width="640" alt="S ReplaySubject" src="https://user-images.githubusercontent.com/48352065/99218709-757a2080-281e-11eb-80ba-be17fd73f0d5.png">

## Reference
[ReactiveX](http://reactivex.io/)

[RxSwift foundation and basic components: part 1](https://www.notion.so/2020-11-16-Swift-RxSwiftFoundationAndBasicComponents-md-43cb08f34c804ab6b631d2087c982de5)
