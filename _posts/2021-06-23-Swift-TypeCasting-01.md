---
title: Swift - Type Casting
layout: single
comments: true
share: true
categories:
- Swift
tag:
- Type Casting
last_modified_at: 2021-06-17 T14:35:00+08:00
---

타입 캐스팅은 인스턴스의 타입을 확인하거나 해당 인스턴스를 클래스 계층 구조 내의 수퍼 클래스 또는 하위 클래스로 취급하는 방법이다. Swift의 타입 캐스팅은 `is` 및 `as` 연산자로 구현되며, 두 연산자는 값의 유형을 확인하거나 값을 다른 유형으로 캐스팅하는데 사용된다.

## Defining a Class Hierarchy for Type Casting

타입 캐스팅을 사용하면 특정 클래스 인스턴스의 타입을 확인하고 해당 인스턴스를 동일한 클래스 계층 구조 내의 다른 클래스로 캐스팅 할 수 있다. 예제 코드를 살펴보자. 기본 클래스인 `MediaItem`과 이를 상속한 `Movie`, `Song` 클래스와 이 두 클래스의 인스턴스를 모두 포함하는 `library` 배열을 정의했다.

```swift
class MediaItem {
    var name: String
    init(name: String) {
        self.name = name
    }
}

class Movie: MediaItem {
    var director: String
    init(name: String, director: String) {
        self.director = director
        super.init(name: name)
    }
}

class Song: MediaItem {
    var artist: String
    init(name: String, artist: String) {
        self.artist = artist
        super.init(name: name)
    }
}

let library = [
    Movie(name: "Casablanca", director: "Michael Curtiz"),
    Song(name: "Blue Suede Shoes", artist: "Elvis Presley"),
    Movie(name: "Citizen Kane", director: "Orson Welles"),
    Song(name: "The One And Only", artist: "Chesney Hawkes"),
    Song(name: "Never Gonna Give You Up", artist: "Rick Astley")
]
```

`Movie`와 `Song` 모두 `MediaItem`이라는 공통 수퍼 클래스가 있기 때문에 `library`의 타입은 `[MediaItem]`으로 swift의 타입 시스템이 추론한다. 실제로 `library`에 저장된 요소는 `Movie`와 `Song`의 인스턴스이지만 배열을 iterate 하는 경우 모든 항목은 `Movie` 또는 `Song`이 아닌 `MediaItem`으로 표현된다. 원래 타입으로 작업하려면 타입을 확인하거나 다른 타입으로 다운 캐스트해야한다.

## Checking Type

인스턴스의 타입을 검사하기 위해서는 `is` 연산을 사용한다. `is` 연산자는 인스턴스의 타입을 검사하여 `Bool` 값을 반환한다. 

```swift
var movieCount = 0
var songCount = 0

for item in library {
    if item is Movie { 
        movieCount += 1
    } else if item is Song {
        songCount += 1
    }
}

print("Media library contains \(movieCount) movies and \(songCount) songs")
// Prints "Media library contains 2 movies and 3 songs"
```

## Downcasting

`as?`, `as!` 연산은 타입 캐스팅을 하는데에 사용되는 연산이다. 타입 캐스팅은 계층 관계를 가지는 타입 간에만 가능하다. 계층 관계가 없는 타입간에는 무조건 캐스팅이 실패하게 된다.  다운 캐스팅을 실패 할 수 있기 때문에 타입 캐스트 연산은 두 가지 다른 형태로 제공된다. 

`as?`는 다운 캐스트하려는 타입의 옵셔널 값을 반환하며, 다운 캐스트가 불가능한 경우 값은 `nil`이 된다. `as!` 연산은 다운 캐스트를 시도하고 결과를 강제 언랩핑한다. 만약 잘못된 클래스 타입으로 다운 캐스트하려고 하면 런타임 오류가 발생한다. 따라서, 다운 캐스트가 성공할지 확실하지 않은 경우 `as?` 연산을 사용하고 다운 캐스트가 항상 성공할 것이라고 확신하는 경우에는 `as!`의 연산을 사용할 수 있다. 

```swift
for item in library {
    if let movie = item as? Movie {
        print("Movie: \(movie.name), dir. \(movie.director)")
    } else if let song = item as? Song {
        print("Song: \(song.name), by \(song.artist)")
    }
}

// Movie: Casablanca, dir. Michael Curtiz
// Song: Blue Suede Shoes, by Elvis Presley
// Movie: Citizen Kane, dir. Orson Welles
// Song: The One And Only, by Chesney Hawkes
// Song: Never Gonna Give You Up, by Rick Astley
```

## Type Casting for Any and AnyObject

Swift는 `Any`, `AnyObject`라는 두 가지 특수 타입을 제공한다. `Any`는 함수를 포함하여 모든 타입의 인스턴스를 나타낼 수 있고, `AnyObject`는 모든 클래스 타입의 인스턴스를 나타낼 수 있다. 이들이 제공하는 동작과 기능이 명시적으로 필요한 경우에만 `Any` 및 `AnyObject`를 사용해아 한다. 사용할때는 실제 타입으로의 다운캐스팅이 대부분 필요하기 때문에 런타임 성능에 좋지 못한 영향을 주고, 설계상으로도 바람직하지 않다. 

```swift
var things: [Any] = []

things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
things.append((3.0, 5.0))
things.append(Movie(name: "Ghostbusters", director: "Ivan Reitman"))
things.append({ (name: String) -> String in "Hello, \(name)" })
```

`things`의 원소들을 사용할때는 `switch`문과 `is`, `as` 구문을 조합하여 사용할 수 있다. 

```swift
for thing in things {
    switch thing {
    case 0 as Int:
        print("zero as an Int")
    case 0 as Double:
        print("zero as a Double")
    case let someInt as Int:
        print("an integer value of \(someInt)")
    case let someDouble as Double where someDouble > 0:
        print("a positive double value of \(someDouble)")
    case is Double:
        print("some other double value that I don't want to print")
    case let someString as String:
        print("a string value of \"\(someString)\"")
    case let (x, y) as (Double, Double):
        print("an (x, y) point at \(x), \(y)")
    case let movie as Movie:
        print("a movie called \(movie.name), dir. \(movie.director)")
    case let stringConverter as (String) -> String:
        print(stringConverter("Michael"))
    default:
        print("something else")
    }
}

// zero as an Int
// zero as a Double
// an integer value of 42
// a positive double value of 3.14159
// a string value of "hello"
// an (x, y) point at 3.0, 5.0
// a movie called Ghostbusters, dir. Ivan Reitman
// Hello, Michael
```

## Reference

[https://docs.swift.org/swift-book/LanguageGuide/TypeCasting.html](https://docs.swift.org/swift-book/LanguageGuide/TypeCasting.html)
