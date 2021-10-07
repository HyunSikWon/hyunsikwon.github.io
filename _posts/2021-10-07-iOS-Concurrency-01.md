---
title: iOS - Concurrency
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Concurrency
last_modified_at: 2021-10-07 T15:00:00+08:00

---

## Concurrency

Concurrency(동시성)은 앱의 로직 내에서 특정 부분이 동시에 그리고 임의의 순서로 실행되는 것을 말한다. 동시에 여러 작업을 수행할 수 있기 때문에 성능 향상에 이점을 가져다 줄 수 있다. 동시성은 데이터 흐름을 정확하게 유지하는 것이 중요하기 때문에 동시에 이루어지는 두 작업이 하나의 데이터를 조작하면 안된다.

동시성은 성능 향상 뿐만 아니라 유저의 사용성의 측면에서도 이점이 있다. 네트워크 작업을 통해 이미지를 내려받는다고 가정해보자. 네트워크 환경에 따라 이미지를 내려받는 시간이 지연된다면 UI를 다루는 작업이 수행되지 않아 앱이 멈추게 되고 사용자의 사용성에 부정적인 영향을 주게 될 것이다. 그러나 UI 작업, 이미지 fetch 작업이 동시에 수행된다면 이 문제를 해결할 수 있을 것이다.

## Concurrency를 사용하는 방법

Swift는 동시성 프로그래밍을 위해서 GCD와 Operation API를 제공한다.

### GCD

Low level API인 GCD는 스레드 풀을 관리하며 dispatch queue에 있는 작업들을 사용 가능한 스레드에 스케쥴링 한다. 개발자는 스레드 관리에 대해선 신경쓰지 않아도 되며 작업을 dispatch queue에 넣어주기만 하면된다. 해당 작업들은 시스템에 의해 스레드에 할당된다.

### Dispatch queues

Dispatch queue를 생성하면 OS가 하나 이상의 스레드를 생성하고 해당 큐에 할당해준다. 이미 존재하는 스레드가 이용 가능 상태면 재사용되지만 그렇지 않다면 OS가 새로운 스레드를 생성한다.

**Serial vs Concurrent**

Dispatch queue에는 serial queue와 concurrent queue가 있다. serial queue는 queue의 작업을 오직 하나의 스레드에서만 수행하고, concurrent queue는 여러 스레드에서 작업을 수행한다.

note: concurrent queue가 언제나 한번에 여러 작업이 수행됨을 보장하진 않는다. 자원의 환경에 따라 한번에 한 작업만 수행할 수 있다.

**sync vs async**

큐에 들어있는 작업들은 동기적(sync)으로 또는 비동기적(async)으로 수행될 수 있다. 작업이 동기적으로 수행될 때는 앱이 다음 작업으로 넘어가기 전에 현재 run loop의 수행이 끝날 때까지 기다린다. 반면 비동기적인 작업의 수행은 현재 작업을 수행하는 즉시 반환되어 다음 작업을 바로 수행한다.

**Main queue**

Main queue는 앱이 시작되면 자동으로 생성된다. Serial queue이며 언제나 UI를 위해서만 사용해야한다. `DispatchQueue.main` 을 통해 접근할 수 있다. Main queue는 언제나 비동기적으로 작업을 수행해야 한다.

**QoS**

QoS는 Dispatch queue 작업의 중요도를 나타낸다. QoS는 리소스를 요구하는 다른 모든 작업에 대해 우선 순위를 지정하여 큐에 전송되는 작업이 얼마나 중요한지 시스템에 알려주는 것이다. 우선 순위가 높은 작업은 더 빨리 수행되어야하며 완료하는데 더 많은 시스템 리소스가 필요하고 우선 순위가 낮은 작업보다 더 많은 에너지가 필요할 수 있다. Concurrent queue에서만 필요하며 직접 정의하지 않고 DispatchQueue의 `global` 클래스 메소드를 사용하여 미리 정의된 전역 대기열 중 하나를 가져올 수 있다.

```swift
let queue = DispatchQueue.global(qos: .userInteractive)
```

**QoS의 종류(중요도 순)**

- `.userInteractive`
    
     `.userInteractive`는 유저와 직접 상호 작용하는 작업에서 사용한다. UI 업데이트, 애니메이션과 같은 UI의 즉각적인 반응과 빠른 속도를 필요로 하는 작업을 위해 사용한다.
    
- `.userInitiated`
    
    `.userInitiated`는 사용자와의 인터렉션이 UI에서 즉시 발생해야하지만 비동기 적으로 수행 할 수 있는 작업을 위해 사용한다. 예를 들어 문서를 열거나 로컬 데이터베이스에서 데이터를 읽어야 하는 경우를 예로들 수 있다.
    
- `.utility`
    
    일반적으로 어느정도 시간이 요구되는 연산, I/O, 네트워킹 등 주로 progress indicator를 필요로 하는 작업에 사용한다. 시스템은 응답성과 성능의 균형을 맞추려고 한다.
    
- `.background`
    
    사용자가 직접 인식하지 못하는 작업의 경우에 사용한다. 사용자와의 상호 작용이 필요하지 않으며 시간에도 크게 예민하지 않은 작업에 사용하면 된다. 데이터베이스 유지 관리, 원격 서버 동기화 및 백업 수행이 좋은 예 이다. OS는 속도 대신 에너지 효율성에 중점을 둔다.
    
- `.default` 및 `.unspecified`
    
    QoS의 디폴트 값인 `.default`는 `.userInitiated`와 `.utility` 사이에 있다. 두번째는 `.unspecified`이며 스레드를 옵트아웃 할 수 있는 레거시 API를 지원하기 위해 존재한다.
    

**QoS 추론**

OS는 queue에 전달된 작업의 유형을 확인하고 필요에 따라 QoS를 변경한다. queue의 QoS보다 우선순위가 높은 작업을 제출하면 queue의 QoS 레벨이 높아지고 queue에 포함된 모든 작업의 우선 순위도 함께 높아진다.

현재 context가 Main thread인 경우 OS는 QoS를 `.userInitiated`로 추론한다. queue에 더 높은 QoS의 작업을 추가함으로써 해당 큐의 QoS를 이에 맞게 증가시킬 수 있다.

**`DispatchQueue`의 Capturing**

```swift
DispatchQueue.global(qos: .utility).async { [weak self] in

	guard let self = self else { return }

	// Perform your work here

	// ...

	// Switch back to the main queue to

	// update your UI

	DispatchQueue.main.async {

		self.textLabel.text = "New articles available!"

	}

}
```

GCD `async` 클로저에서 `self` 를 강하게 capturing 하는 것은 참조 사이클을 유발하진 않는다. 클로저가 작업을 완료하면 전체 클로저가 deallocate 되기 때문이다. 그러나 `self` 자체의 lifetime을 늘릴 수는 있다. 네트워크 요청을 수행하는 클로저는 View Controller가 dismiss되어도 cloure가 작업을 마칠 때까지 여전히 메모리에 살아있을 것이다. 반대로 약하게 capturing하면 `nil`이 될 것이다.

**DispatchWorkItem**

`DispatchWorkItem`  클래스를 통해 `DispatchQueue`에 작업을 전달할 수도 있다. `DispatchWorkItem` 클래스는 queue에 전달할 코드를 갖는 객체 형태로 사용할 수 있다.

```swift
let queue = DispatchQueue(label: "xyz")

let workItem = DispatchWorkItem {
	print("The block of code ran!")
}

queue.async(execute: workItem)
```

`DispatchWorkItem` 을 사용하는 이유는 작업의 수행 도중 혹은 수행 전에 취소할 수 있기 때문이다. 취소를 위해선  `cancel()` 메소드를 호출하면 된다. 메소드를 호출 했을때 작업이 아직 시작되지 않은 경우 queue에서 제거된다. 반대로 작업이 현재 실행 중이면 `isCancelled` 프로퍼티가 `true`로 설정된다. `isCancelled` 프로퍼티를 주기적으로 확인하여  작업을 취소하기 위한 적절한 조치를 취해야한다.

`DispatchWorkItem` 클래스는 `notify(queue: execute:)` 메소드를 제공하기도 하는데, 이를 통해 작업간 순서를 지정해줄 수 있다.

```swift
let queue = DispatchQueue(label: "xyz")

let backgroundWorkItem = DispatchWorkItem { }

let updateUIWorkItem = DispatchWorkItem { }

backgroundWorkItem.notify(queue: DispatchQueue.main, execute: updateUIWorkItem)

queue.async(execute: backgroundWorkItem)
```

### DispatchGroup

`DispatchGroup` 클래스를 사용해서 작업 그룹이 모두 수행되었는지 추적할 수 있다. 하나의 그룹을 사용하면서 작업을 여러 큐에 제출할 수 있다. `DispatchGroup` 은 제출된 모든 작업이 완료되는 즉시 알림을 받는데 사용할 수 있는 `notify(queue:)` 메소드를 제공한다. 이 메소드는 파라미터로 dispatch queue를 받으며 작업이 모두 완료되면 원하는 작업을 해당 dispatch queue에서 실행할 수 있다.

```swift
let group = DispatchGroup()

someQueue.async(group: group) { 
	// ... your work ... 
}

someQueue.async(group: group) { 
	// ... more work .... 
}

someOtherQueue.async(group: group) { 
	// ... other work ...
 }

group.notify(queue: DispatchQueue.main) { [weak self] in
	self?.textLabel.text = "All jobs have completed"
}
```

기본적으로 group의 작업 완료 알림은 비동기적이다. 그룹의 완료 알림에 응답할 수 없는 경우엔 `wait` 메소드를 사용하면 된다. 이는 동기적인 메소드이며 현재 큐를 블럭시키고 작업 그룹이 완료되길 기다린다.

```swift
let group = DispatchGroup()

someQueue.async(group: group) { }

someQueue.async(group: group) { }

someOtherQueue.async(group: group) { }

if group.wait(timeout: .now() + 60) == .timedOut {
	print("The jobs didn’t finish in 60 seconds")
}
```

`DispatchGroup`을 사용할때 만약 비동기적인 메소드를 클로저 내부에서 호출하게 되면 내부의 비동기 메소드가 완료되기 전에 클로저가 종료되는 문제가 발생할 수 있다. 따라서 이부분을 처리해주는 작업이 반드시 필요하다. `DispatchGroup`에서 제공하는 `enter()` 와 `leave()` 메소드를 사용하면 실행 중인 작업의 개수를 카운팅해서 해당 작업 그룹이 모두 완료되었는지 파악할 수 있다. 오류 상태에서도 `leave()`를 호출해야하기 때문에 일반적으로 `defer` 문을 사용한다.

```swift
func myAsyncAdd(lhs: Int, rhs: Int, completion: @escaping (Int) -> Void) {
	// Lots of cool code here
	completion(lhs + rhs)
}

func myAsyncAddForGroups(group: DispatchGroup, lhs: Int, rhs: Int, 
completion: @escaping (Int) -> Void) {

	group.enter()

	myAsyncAdd(first: first, second: second) { result in

		defer { group.leave() }
		completion(result)

	}

}
```

### Semaphores

공유 자원에 액세스 할 수 있는 스레드 수를 제어해야하는 경우가 있다. 예를 들어 네트워크에서 데이터를 다운로드하는 경우, 가져 오는 데이터가 상당히 크고 처리하는데 많은 리소스가 필요하다면 한 번에 4 개의 다운로드만 수행하도록 허용하여 성능을 개선할 수 있다.

이는 `DispatchSemaphore`를 사용하면 되는데, 리소스를 원하는대로 사용하기 전에 동기 함수인 `wait` 메서드를 호출하면 리소스를 사용할 수 있을 때까지 스레드가 실행을 일시 중지한다. 사용가능한 경우 즉시 액세스 할 수 있고, 다른 스레드가 점유하고 있는 경우 완료되었다는 신호를 받을 때까지 기다린다.

세마포어를 만들 때 리소스에 대한 동시 액세스가 허용되는 수를 파라미터로 지정한다. 한 번에 네 개의 네트워크 다운로드를 활성화하려면 4를 전달한다. 독점 액세스를 위해 리소스를 잠 그려는 경우 1을 지정하면된다.

```swift
let semaphore = DispatchSemaphore(value: 4)
```

### Concurrency Problems

**Race conditions**

Race condition이란 두 개 이상의 프로세스가 공통 자원을 병행적으로(concurrently) 읽거나 쓰는 동작을 할 때 발생하는 문제이다. 즉, 여러 스레드가 하나의 공통된 자원을 놓고 서로 경쟁하는 상황을 말한다.

일반적으로 serial queue을 사용하여 race condition을 해결할 수 있다. 동시에 액세스 해야하는 변수가 있는 경우 다음과 같이 읽기/쓰기를 래핑 할 수 있다.

```swift
private let threadSafeCountQueue = DispatchQueue(label: "...")
private var _count = 0

public var count: Int {
	get {
		return threadSafeCountQueue.sync { _count }
	}
	set {
		threadSafeCountQueue.sync { _count = newValue }
	}

}
```

GCD의 dispatch barrier를 사용하면 concurrent queue로도 같은 기능을 구현할 수 있다. Barrier란 말 그대로 장벽이며 concurrent queue에서 `flags`가 barrier로 설정된 작업이 실행되면 그 작업이 끝날 때 까지 serial queue 처럼 동작하게 된다. barrier에 도달하면 queue는 serial 처럼 동작하고 완료 될 때까지 barrier 작업만 실행할 수 있다. 완료되면 barrier 작업 이후 제출 된 모든 작업을 concurrent로 다시 실행할 수 있다.

**Deadlock**

시스템 자원에 대한 요구가 뒤엉킨 상태이다. 즉, 둘 이상의 프로세스가 다른 프로세스가 점유하고 있는 자원을 서로 기다릴 때 무한 대기에 빠지는 상황을 말한다. 흔하게 발생하는 일은 아니지만 Swift에서는 semaphore를 사용할때 주로 발생한다.

Semaphore를 사용하여 여러 리소스에 대한 액세스를 제어하는 경우 동일한 순서로 리소스를 요청해야 한다. 스레드 1이 망치와 톱을 요청하고 스레드 2가 톱과 망치를 요청하면 교착 상태가 발생할 수 있다. 스레드 1은 망치를 요청하고 받는 동시에 스레드 2는 톱을 요청하고 받는다. 그런 다음 스레드 1은 망치를 놓지 않고 톱을 요청하지만 스레드 2는 리소스를 소유하므로 스레드 1은 기다려야하고 스레드 2는 톱을 요청하지만 스레드 1은 여전히 리소스를 소유하고 있으므로 스레드 2는 톱이 사용 가능해질 때까지 기다려야한다. 요청 된 리소스가 해제 될 때까지 진행할 수 없으므로 두 스레드는 이제 교착 상태에 있는 것이다.

**Priority inversion**

일반적으로 작업을 queue에 제출하면 queue 자체의 우선 순위가 적용된다. 그러나 상황에 따라 특정 작업의 우선 순위가 이보다 높거나 낮게 지정될 수 있다. 예를들어, `.userInitiated` queue와 `.utility` queue를 사용하고 있고 `.userInteractive` queue를 사용하여 `.utility` queue에 여러 작업을 제출하면,  `.utility` queue는 운영 체제에 의해 더 높은 우선 순위가 할당된다. 갑자기 queue의 모든 작업이 (대부분이 실제로 `.utility` QoS 인)  `.userInitiated` queue의 작업보다 먼저 실행되는 것이다.

Priority inversion이 발생하는 보다 일반적인 상황은 더 높은 QoS의 queue가 낮은 QoS queue와 리소스를 공유하는 경우이다. QoS가 낮은 queue가 객체에 대한 lock을 얻으면 lock이 해제 될 때까지 우선 순위가 높은 queue는 우선 순위가 낮은 작업이 실행되는 동안 아무 작업도하지 않고 멈춘다.

## Operations

Opertion은 GCD 위에서 동작하며 GCD에 더 복잡한 요구사항을 지원하기 위한 객체지향 모델, 의존성, 작업 취소 같은 특성들을 추가한 API 이다. 추상 클래스이므로 서브클래스 정의해서 사용하거나 시스템에서 제공하는 서브클래스를 사용해서 작업을 해야한다. 

`Operation`의 특징은 재사용성이다. `Operation`은 Swift 객체이다.  따라서 입력을 전달하여 작업을 설정하고 helper 메서드를 구현할 수도 있다. 작업 단위 혹은 작업을 래핑하고 나중에 실행할 수 있으며 해당 작업 단위를 여러번 제출할 수도 있다.

**Operation States**

`Operation`은 생명 주기에 따라 각기 다른 상태를 갖게된다.

- 인스턴스화 되고 실행할 준비가 되면 `isReady` 상태로 전환된다.
- `start` 메소드를 호출 하면 이 시점에서 `isExecuting` 상태로 전환된다.
- 앱이 `cancel` 메소드를 호출하면 `isFinished` 상태로 이동하기 전에 `isCancelled` 상태로 전환된다.
- 취소되지 않은 경우 `isExecution` 에서 `isFinished`로 직접 이동한다

**BlockOperation**

`BlockOperation` 클래스를 사용하여 코드 블록으로 `Operation`을 간단하게 만들 수 있다. 일반적으로 이니셜라이저에 클로저를 전달하기만 하면 된다.

```swift
let operation = BlockOperation {
	print("2 + 3 = \(2 + 3)")
}
```

`BlockOperation`은 클로저 그룹을 관리하기도 한다. 모든 클로저가 완료되면 자신을 완료된 것으로 표시한다는 점에서 dispatch group과 유사하게 작동한다. `addExecutionBlock` 메소드를 사용하면 된다.

```swift
let sentence = "Ray’s courses are the best!"

let wordOperation = BlockOperation()

for word in sentence.split(separator: " ") {

	wordOperation.addExecutionBlock { print(word) }

}

wordOperation.start()
```

### Operatoin Queue

GCD의 `DispatchQueue`와 마찬가지로 `OperationQueue` 클래스는 작업 스케줄링과 동시에 실행할 수 있는 작업의 최대 수를 관리하는데 사용하는 클래스이다.

`OperationQueue`를 사용하면 다음 세 가지 방법으로 작업을 추가 할 수 있다.

- `Operation`을 전달
- 클로저를 전달
- `[Operation]`을 전달

`OperationQueue`는 QoS 값 및 작업의 의존성에 따라 준비상태의 작업을 실행한다.  `OperationQueue`에 작업을 추가한 후에는 같은 작업을 다른 `OperationQueue`에 추가 할 수 없다. 

`OperationQueue`의  `waitUntilAllOperationsAreFinished`라는 메서드를 사용하여 모든 Operation이 끝날 때까지 기다릴 수 있다. 이 메소드를 호출하면 현재 스레드가 block 되므로 main UI 스레드에서 이 메서드를 호출해서는 안된다. 모든 작업이 완료 될 때까지 기다릴 필요 없고 특정 작업 집합의 완료를 기다리는 것은 `OperationQueue`의 `addOperations(_: waitUntilFinished:)` 메서드를 사용하면 된다.

**QoS**

`OperationQueue`의 기본 QoS는 `.background` 이다. `OperationQueue`에서 `qualityOfService` 프로퍼티로 값을 설정할 수 있지만 큐에서 관리하는 개별 작업에 설정된 QoS에 의해 재정의 될 수 있다.

**큐 일시 중지**

`isSuspended` 프로퍼티를 `true`로 설정하여 큐를 일시 중지 할 수 있다. 현재 작업은 계속 실행되지만 새로 추가된 작업은 `isSuspended`를 다시 `false`로 변경할 때까지 스케줄링되지 않는다.

**최대 작업 수**

한 번에 실행되는 작업의 수를 제한해야하는 경우가 있다. 기본적으로 queue는 한 번에 처리할 수 있는 작업을 최대한 많이 실행한다다. 해당 수를 제한하려면 queue에서 `maxConcurrentOperationCount` 속성을 설정하면 된다. `maxConcurrentOperationCount`를 1로 설정하면 serial queue의 기능을 한다.

### Asynchronous Operations

`Operation`은 기본적으로 동기적으로 수행된다. 작업이 동기적으로 수행되면서 `Operation`은 준비 -> 실행 -> 종료 의 상태 변화 과정을 거친다. 동기적인 작업의 경우 `Operation`의 `main` 함수가 종료된 시점에 실행 상태에서 종료상태로 전환해도 문제가 없다. 그러나 비동기적인 작업의 경우 `main` 함수가 종료된 시점에도 함수내의 비동기 작업이 끝나지 않았을 수 있기 때문에 종료 상태로 전환할 수 없다. 따라서, 비동기 작업을 정의하려면 작업의 진행 상태를 모니터링하고 KVO를 사용하여 해당 상태의 변경 사항을 리포트해야 한다.

### Operation Dependencies

`Operation` 사이의 의존성을 추가해 작업간의 순서를 지정해줄 수 있다. 작업간 순서를 지정해줌으로써 공통된 데이터를 주고받는 안전한 방법을 제공할 수 있다.

의존성은  `addDependecy` 메소드를 사용하여 지정한다.

```swift
let networkOp = NetworkImageOperation()

let decryptOp = DecryptOperation()

let tiltShiftOp = TiltShiftOperation()

decryptOp.addDependency(op: networkOp)

tiltShiftOp.addDependency(op: decryptOp)
```

`removeDependency` 메소드를 통해 의존성을 제거할 수 있다.

```swift
tiltShiftOp.removeDependency(op: decryptOp)
```

### Canceling Opertions

큐에 있는 작업을 `cancel` 메소드를 통해 취소할 수 있다. 취소 작업 외에는 제출된 `Operation`에 대한 어떠한 통제도 할 수 없다. 작업의 취소는 `Operatoin` 객채의 `cancel()` 메소드를 호출하거나 `OperationQueue` 클래스의 `cancelAllOperations()` 메서드를 호출하여 수행한다.

`Operation` 을 직접 구현한다면 항상 취소 관련 구문을 작성해야한다. 특히 메인 작업 코드는 `isCancelled` 속성의 값을 주기적으로 확인해야 한다. 속성이 `true` 이면 가 가능한 빨리 정리되고 종료되어야 한다. `start()` 메소드를 구현하는 경우 해당 메소드는 취소 여부를 확인하고 적절한 처리를 해줘야 한다.

작업이 취소 될 때 단순히 종료하는 것 외에도 작업의 상태 변경 작업도 필요하다. `isFinished` 및 `isExecuting` 프로퍼티의 값을 직접 관리하는 경우 상황에 따라 상태를 업데이트해야 한다. 실행을 시작하기 전에 작업이 취소 된 경우에도 이러한 변경을 수행해야 한다.

## Reference

Concurrency by Tutorials - Raywenderich
