---
title: RxSwift - Operator를 살펴보자!!
layout: single
comments: true
share: true
categories:

- Swift
tag:
- RxSwift
last_modified_at: 2020-11-22 T22:00:00+08:00

---

## Operator?

Operator는 Rx의 기본 요소이며, Operator를 사용해 Observable에 의해 방출되는 이벤트를 변환하고 처리할 수 있다. Operator는 크게 세 그룹으로 나뉜다

- Filtering Operator
- Transforming Operator
- Combining Operator

## 1. Filtering Operators

> Observable의 item을 선택적으로 방출(emit)하는 ***operator***.

### IgnoreElements

<img width="640" alt="ignoreElements c" src="https://user-images.githubusercontent.com/48352065/99904741-47aa4580-2d10-11eb-8649-8488379c97cc.png">

`ignoreElements`는 source observable에서 emit되는 요소는 무시하고, Observable의 종료 이벤트(`onError`, `onCompleted`)만 허용한다.

```swift
example(of: "ignoreElements") {
    
    let strikes = PublishSubject<String>()
    let disposeBag = DisposeBag()
  
    strikes
        .ignoreElements()
        .subscribe({ _ in
            print("You're out!")
        })
        .disposed(by: disposeBag)
    
    // 무시된다.
    strikes.onNext("X")
    strikes.onNext("X")
    strikes.onNext("X")
    
    // 허용
    strikes.onCompleted()
}

	// "You're out!" 출력
```

### ElementAt

<img width="752" alt="elementAt" src="https://user-images.githubusercontent.com/48352065/99904740-4711af00-2d10-11eb-9fdf-96bb98e98537.png">

`elementAt` 는 source observable에서 emit 되는 요소 중 오직 n번째 요소만 emit한다.

```swift
example(of: "elementAt") {
    let strikes = PublishSubject<String>()
    let disposeBag = DisposeBag()
    
    // index 2 즉, 3번째 값을 emit
    strikes
        .elementAt(2)
        .subscribe(onNext: { _ in
            print("You're out!")
        })
        .disposed(by: disposeBag)
    
    strikes.onNext("X")
    strikes.onNext("X")
    strikes.onNext("X")
}
```

### Filter

<img width="752" alt="filter" src="https://user-images.githubusercontent.com/48352065/99904737-4711af00-2d10-11eb-861d-593cf13d9b9a.png">
`filter`는 문자 그대로 observable의 요소들을 필터링하여 emit한다.

```swift
example(of: "filter") {
    
    let disposeBag = DisposeBag()
    
    Observable.of(1,2,3,4,5,6)
        .filter({ (int) -> Bool in
            int % 2 == 0
        })
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
}

// 출력
/* 
2
4
6
*/
```

## Skip

<img width="752" alt="skip" src="https://user-images.githubusercontent.com/48352065/99904736-4547eb80-2d10-11eb-9ca7-c53f3c36071f.png">

`skip`은 처음 n개의 요소들은 skip하고 그 이후의 요소들만 emit한다.

```swift
example(of: "skip") {
    let disposeBag = DisposeBag()
   
    Observable.of("A", "B", "C", "D", "E", "F")
        .skip(3)
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
}

// D, E, F 만 차례대로 출력
```

### SkipWhile

<img width="640" alt="skipWhile c" src="https://user-images.githubusercontent.com/48352065/99904735-437e2800-2d10-11eb-9f94-15846648cecd.png">

`skipWhile`은특정 조건이 false일 때까지 skip하고 그 이후의 요소만 emit한다.

```swift
example(of: "skipWhile") {
    
    let disposeBag = DisposeBag()
   
    Observable.of(2, 2, 3, 4, 4)
				// 조건이 false가 되는 시점부터 출력
        .skipWhile({ (int) -> Bool in
            int % 2 == 0
        })
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
}

// 3, 4, 4가 차례대로 출력
```

### SkipUntil

<img width="753" alt="skipUntil" src="https://user-images.githubusercontent.com/48352065/99904733-424cfb00-2d10-11eb-8950-446c22415664.png">
`skipUntil`은 다른 어떤 observable이 요소를 emit하기 전까지 skip한다.

```swift
example(of: "skipUntil") {
    let disposeBag = DisposeBag()
    
    let subject = PublishSubject<String>()
    let trigger = PublishSubject<String>()
    
    subject
        .skipUntil(trigger)
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
    
    subject.onNext("A")
    subject.onNext("B")
    
		// skip 종료!
    trigger.onNext("X")
    
    subject.onNext("C")
}
// C 만 출력
```

### Take

<img width="752" alt="take" src="https://user-images.githubusercontent.com/48352065/99904731-41b46480-2d10-11eb-8a07-36f179c88407.png">

`take`은 `skip`의 반대 개념. 처음 n번째 요소만 emit 한다.

```swift
example(of: "take") {
    let disposeBag = DisposeBag()
    
    // 1
    Observable.of(1,2,3,4,5,6)
        // 2
        .take(3)
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
}
// 1, 2, 3 만 출력
```

### TakeWhile

<img width="640" alt="takeWhile c" src="https://user-images.githubusercontent.com/48352065/99904726-3feaa100-2d10-11eb-85b6-f32d253967b8.png">

`takeWhile`은 특정 조건이 false일 때까지 emit 한다.

### Enumerated

`enumerated`는 방출된 요소의 index를 사용하고 싶은 경우에 사용한다.`takeWhile` 과 함께 코드로 살펴보자:

```swift
example(of: "takeWhile") {
    let disposeBag = DisposeBag()
    
    Observable.of(2,2,4,4,6,6)
        .enumerated() // 인덱스를 사용가능
        .takeWhile({ index, value in // 해당 조건이 false일 때까지 emit
            value % 2 == 0 && index < 3
        })
        .map { $0.element }
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
}

// 2, 2, 4 차례대로 출력
```

### TakeUntil

<img width="752" alt="takeUntil" src="https://user-images.githubusercontent.com/48352065/99904725-3f520a80-2d10-11eb-9e6e-3e95451f106c.png">

`takeUntil`은 다른 Observable이 어떤 요소 혹은 종료를 emit한 후에 혀emit되는 요소들을 무시한다.

```swift
example(of: "takeUntil") {
    let disposeBag = DisposeBag()
    
    let subject = PublishSubject<Int>()
    let trigger = PublishSubject<Int>()
    
    subject
        .takeUntil(trigger)
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
    
    
    subject.onNext(1)
    subject.onNext(2)
    
    trigger.onNext(0)
    
    subject.onNext(3)
    
}

// 1, 2 만 출력
```

### DistinctUntilChanged

<img width="822" alt="distinctUntilChanged" src="https://user-images.githubusercontent.com/48352065/99904724-3eb97400-2d10-11eb-8088-2604d0627cb2.png">

`distinctUntilChanged`는 연달아 나오는 중복 요소를 무시한다. 

```swift
example(of: "distinctUntilChanaged") {
    let disposeBag = DisposeBag()

    Observable.of(1,2,2,1)
        .distinctUntilChanged()
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
}

// 1, 2, 1 출력 
```

## 2. Transforming Operators

> Observable에서 방출(emit)된 item을 변환하는 ***operator*.**

### To

<img width="640" alt="to c" src="https://user-images.githubusercontent.com/48352065/99904723-3cefb080-2d10-11eb-83a1-a1c8c63fa6e9.png">

`to` 는 Observable을 다른 객체나 자료 구조로 변환해준다.

```swift
example(of: "toArray") {
    let disposeBag = DisposeBag()
    
    Observable.of(1,2,3,4)
        .toArray()
        .subscribe(onSuccess: {
            print($0)
        })
        .disposed(by: disposeBag)
}
```

### Map

<img width="752" alt="map" src="https://user-images.githubusercontent.com/48352065/99904722-3c571a00-2d10-11eb-95c0-aae9ef59603e.png">

`map`은 Observable에서 emit되는 요소 각각에 특정 함수를 적용하여 요소를 변환한다.

```swift
example(of: "map") {
    let disposeBag = DisposeBag()
    
    let formatter = NumberFormatter()
    formatter.numberStyle = .spellOut
    
    Observable.of(123, 4, 56)
        .map { // 변환
            formatter.string(from: $0) ?? ""
        }
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
}
/* 출력
one hundred twenty-three
four
fifty-six
*/
```

### FlapMap

<img width="640" alt="flatMap c" src="https://user-images.githubusercontent.com/48352065/99904721-3b25ed00-2d10-11eb-89c5-8e784ed057d0.png">

`flapMap`은 Observable에서 방출된 각각의 요소를 Observable로 변환하고, 이들을 결합하여 하나의 새로운 Observable로 만든다.

```swift
struct Student {
    var score: BehaviorSubject<Int>
}

example(of: "flatMap") {
    let disposeBag = DisposeBag()
   
    let ryan = Student(score: BehaviorSubject(value: 80))
    let charlotte = Student(score: BehaviorSubject(value: 90))
    
    let student = PublishSubject<Student>()
    
    student
        .flatMap{
            $0.score
        }
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
    
   
    student.onNext(ryan)  
    ryan.score.onNext(85)   
    student.onNext(charlotte)   
    ryan.score.onNext(95)   
    charlotte.score.onNext(100) 

	/*
	80
	85
	90
	95
	100
	*/
    
}
```

### FlapMapLatest

<img width="640" alt="flatMapLatest" src="https://user-images.githubusercontent.com/48352065/99904720-39f4c000-2d10-11eb-8f1e-fa38e74f8001.png">

`flapMapLatest`는 이름에서 알 수 있듯, `flapMap` 과 거의 동일하게 작동하지만, 변환된 Observable 중 가장 최신의 Observable만 받는다.

```swift
example(of: "flatMapLatest") {
    let disposeBag = DisposeBag()
    
    let ryan = Student(score: BehaviorSubject(value: 80))
    let charlotte = Student(score: BehaviorSubject(value: 90))
    
    let student = PublishSubject<Student>()
    
    student
        .flatMapLatest {
            $0.score
        }
        .subscribe(onNext: {
            print($0)
        })
        .disposed(by: disposeBag)
    
    student.onNext(ryan) // 80이 출력
    ryan.score.onNext(85) // 85 출력
    
    student.onNext(charlotte) // 90 출력
    
    // 1
    ryan.score.onNext(95) // 가장 최근에 반영한 obsrvable은 charlotte임. 반영이 안된다.
    charlotte.score.onNext(100) // 100 출력
    
    /* Prints:
     80 
		 85 
     90 
     100
     */
}
```

## 3. Combining Operators

> 여러 Observable을 결합하여 하나의 Observable로 생성하는 ***operator.***

### StartWith

<img width="752" alt="startWith" src="https://user-images.githubusercontent.com/48352065/99904719-395c2980-2d10-11eb-96a6-42e99124aedb.png">

`startWith`는 source observable에서 요소들을 emit 하기 전에 특정한 요소 sequence를 emit한다.

```swift
example(of: "startWith") {
    let numbers = Observable.of(2,3,4,5)
    
    numbers
        .startWith(1)
        .subscribe(onNext: { print($0)})
        .dispose()
}

/*
1
2
3
4
5
*/
```

### Concat

<img width="752" alt="concat" src="https://user-images.githubusercontent.com/48352065/99904718-382afc80-2d10-11eb-9a62-2f1a43537a91.png">

`concat`은 여러 observable의 출력을 연결하여 하나의 observable 처럼 작동하도록 만든다. 첫번째 observable이 모든 요소를 emit하면 다음 observable이 자신의 요소들을 emit한다. 

만약, observable의 어떤 부분에서 에러가 방출되면, `concat`된 observable도 에러를 방출하며 완전 종료된다.

```swift
example(of: "Observable.concat") {
    let first = Observable.of(1,2,3)
    let second = Observable.of(4,5,6)
    
    first.concat(second)
        .subscribe(onNext: {
            print($0)
        })
        .dispose()
    
}
/*
1
2
3
4
5
6
*/
```

### Merge

<img width="640" alt="merge" src="https://user-images.githubusercontent.com/48352065/99904717-36f9cf80-2d10-11eb-9b39-7cd0406f2062.png">

`merge`는 여러 observable의 출력을 결합하여 하나의 observable로 만든다.

```swift

example(of: "merge") {
    let subject1 = PublishSubject()
    let subject2 = PublishSubject()
    
    Observable.of(subject1, subject2)
        .merge()
        .subscribe {
            print($0)
        }
    
    subject1.on(.Next(10))
    subject1.on(.Next(11))
    subject1.on(.Next(12))
    subject2.on(.Next(20))
    subject2.on(.Next(21))
    subject1.on(.Next(14))
    subject1.on(.Completed)
    subject2.on(.Next(22))
    subject2.on(.Completed)
}

/*
 Next(10)
 Next(11)
 Next(12)
 Next(20)
 Next(21)
 Next(14)
 Next(22)
 Completed
 */
```

### CombineLatest

<img width="752" alt="combineLatest" src="https://user-images.githubusercontent.com/48352065/99904716-35c8a280-2d10-11eb-9c8d-4424a509c273.png">

`combineLatest`는 각 observable에서 방출되는 가장 최근의 요소들에 함수를 적용하여 결합하여 emit한다.

```swift
example(of: "combineLast") {
    let left = PublishSubject<String>()
    let right = PublishSubject<String>()
    
    let diposable = Observable.combineLatest(left, right) { left, right in
        "\(left) \(right)"
    }
    .subscribe(onNext: {print($0)})
    
    print("> Sending a value to Left")
    left.onNext("Hello,")
    print("> Sending a value to Right")
    right.onNext("world")
    print("> Sending another value to Right")
    right.onNext("RxSwift")
    print("> Sending another value to Left")
    left.onNext("Have a good day,")
    
    diposable.dispose()
}

/*
> Sending a value to Left
> Sending a value to Right
Hello, world
> Sending another value to Right
Hello, RxSwift
> Sending another value to Left
Have a good day, RxSwift
*/
```

### Zip

<img width="752" alt="zip" src="https://user-images.githubusercontent.com/48352065/99904715-35300c00-2d10-11eb-9778-ea5a77939e22.png">

`zip` 역시 여러 observable을 결합하여 하나의 observable로 만든다. `zip`은 emit 되는 요소들을 짝을 맞추어 결합하고, 둘 중 하나의 observable이라도 완료되면 `zip` 역시 종료된다.

```swift
example(of: "zip") {
    enum Weather {
        case cloudy
        case sunny
    }
    
    let left = Observable<Weather>.of(.cloudy,.sunny,.sunny,.cloudy)
    let right = Observable.of("Lisbon", "Copenhagen", "London", "Madrid", "Vienna")
    
    Observable.zip(left, right) { weather, city in
        "It's \(weather) in \(city)"
    }
    .subscribe(onNext: {print($0)})
    .dispose()
    
}

/*
It's cloudy in Lisbon
It's sunny in Copenhagen
It's sunny in London
It's cloudy in Madrid
*/
```

### WithLatestFrom

<img width="821" alt="withLatestFrom" src="https://user-images.githubusercontent.com/48352065/99904714-34977580-2d10-11eb-865c-8d921dd8b728.png">

`withLatestFrom` 은 `combineLatest` 와 유사하지만, source observable이 emit을 해야 다른 observable이 요소를 emit한다. 하나의 observable은 트리거 역할은 하는 것이다. 

```swift
example(of: "withLatestFrom") {
    let button = PublishSubject<Void>()
    let textField = PublishSubject<String>()
    
    button.withLatestFrom(textField)
        .subscribe(onNext: {print($0)})
    
    textField.onNext("Pa")
    textField.onNext("Par")
    button.onNext(()) // trigger
    
    textField.onNext("Paris")
    button.onNext(()) // trigger
    
}

/*
Par
Paris
*/
```

### SwitchLatest

<img width="640" alt="switch c" src="https://user-images.githubusercontent.com/48352065/99904713-33664880-2d10-11eb-9dbc-8bbd652ab716.png">

`switchLatest` 는 observable을 emit하는 observable을 하나의 observable로 만들어 준다.

가장 최근에 방출된 observable이 요소를 emit한다.

```swift
example(of: "switchLatest") {
    
    let disposeBag = DisposeBag()
    
    let one = PublishSubject<String>()
    let two = PublishSubject<String>()
    let three = PublishSubject<String>()
    
    let source = PublishSubject<Observable<String>>()
    
    source.switchLatest()
        .subscribe(onNext: {print($0)})
        .disposed(by: disposeBag)
    
    
    source.onNext(two)
    one.onNext("I'm one")
    two.onNext("I'm two") // emit
    three.onNext("I'm three")
    
    source.onNext(one)
    one.onNext("I'm one") // emit
    two.onNext("I'm two")
    three.onNext("I'm three")
    
    source.onNext(three)
    one.onNext("I'm one")
    two.onNext("I'm two")
    three.onNext("I'm three") // emit
    
    
}

/*
I'm two
I'm one
I'm three
*/
```

### Scan

<img width="640" alt="scan" src="https://user-images.githubusercontent.com/48352065/99904710-306b5800-2d10-11eb-9d6e-6e4b7b69325d.png">

`scan`은 observable에서 emit되는 요소들에 특정 함수를 적용하고, 이 결과를 emit한다. 또한 이 결과는 다음에 emit 되는 요소와 함께 사용된다.

```swift
example(of: "scan") {
    let source = Observable.of(1,3,5,7,9)
    
    source.scan(0, accumulator: +)
        .subscribe(onNext: {print($0)}) 
        .dispose()
}

/*
1
4
9
16
25
*/
```

## Reference

[ReactiveX](http://reactivex.io)
