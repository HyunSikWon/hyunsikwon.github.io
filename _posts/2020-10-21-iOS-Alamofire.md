---

title: 튜토리얼을 통해 Alamofire 공부하기!

layout: single

comments: true

share: true

categories:

-  iOS

tag:

- Alamofire

last_modified_at: 2020-10-21 T14:00:00+08:00

---

앱 개발을 하다보면 네트워크를 통해 데이터에 접근하는 경우가 많다. iOS 개발에서 네트워킹 작업은 Foundation 프레임 워크의 `URLSession` 을 사용해서 이루어진다. `URLSession` 은 가끔은 사용하기에 적합하지 않은데, 이러한 점을 보완하고자 Alamofire 를 사용한다.

Alamofire는 swift 기반 HTTP 네트워킹 라이브러리이다. Alamofire는 네트워킹 작업을 단순화하고 네트워킹을 위한 다양한 메소드와 JSON 파싱 등을 제공한다.

이번 튜토리얼에서 다음과 같은 네트워킹 작업을 수행할 것이다.

- 서드 파티 RESTful-API로부터 데이터 요청.
- 요청 파라미터 보내기.
- 응답을 JSON 으로 변환.
- `Codable` 프로토콜을 이용해서 응답을 Swift 데이터로 변환.

## Using the SW API

**SW API** 는 스타워즈 데이터를 제공하는 오픈 API 이다. 이를 이용해 튜토리얼을 진행할 것이다. 다양한 데이터에 접근할 수 있지만, 이번 튜토리얼에선 [https://swapi.dev/api/films](https://swapi.dev/api/films) 와  [https://swapi.dev/api/starships](https://swapi.dev/api/starships) 를 이용할 것이다.

## Understanding HTTP, REST and JSON

**HTTP** 는 웹 브라우저 혹은 iOS 앱 등, 서버에서 클라이언트로 데이터를 전송하기 위해 사용하는 어플리케이션 프로토콜이다. HTTP는 사용자가 목적에 따라 사용할 수 있는 몇 가지의 요청 메소드를 정의한다.

- **GET:** 웹 페이지 같은 데이터를 회수한다. 서버의 데이터를 바꾸지는 않는다.
- **HEAD:** GET과 동일하지만 실제 데이터가 아닌 header만 요청한다.
- **POST:** 서버에 데이터를 보낸다. 형식을 채우고 제출하기 버튼을 눌렀을 때와 같은 상황에서 사용된다.
- **PUT:** 특정 위치에 데이터를 보내기 위해 사용한다. 사용자의 프로필을 업데이트하는 경우에 사용된다.
- **DELETE:** 특정 위치의 데이터를 삭제하기 위해 사용한다.

**JSON**은 시스템간 데이터 전송을 위한 단순하고, 가독성이 좋은 메커니즘을 제공한다. JSON에서 사용할 수 있는 데이터 타입은 제한적이다: string, boolean, array, object/dictionary, number, null. swift에서는 `Codable` 을 사용해서 쉽게 JSON을 인코딩 혹은 디코딩 할 수 있다.

**REST**는 일관성 있는 웹 API를 설계하기 위한 규약이다. REST는 요청간 상태를 유지하지 않고, 요청을 cacheable하게 만들고, 균일 한 인터페이스를 제공하는 것과 같은 표준을 위한 몇몇 설계 규약을 가진다. 이는 개발자가 요청간 데이터 상태를 추적할 필요 없이 API를 개발하는 앱과 통합하기 쉽게 만들어 준다.

### Why use Alamofire?

그렇다면, 애플이 `URLSession` 등 HTTP 통신을 위한 클래스를 제공하고 있음에도 불구하고 Alamofire을 사용하는 이유는 무엇일까? 그에 대한 짧은 답은, Alamofire는 `URLSession` 을 기반으로 하여, 어려운 네트워킹 작업을 감춰주기 때문에, 주요 로직에 집중할 수 있게 해준다는 것이다. 이를 통해 쉽게 인터넷 데이터에 접근할 수 있고, 코드가 더 깔끔하고 가독성이 좋아질 수 있다.

다음은 Alamofire의 주요 함수이다:

- **AF.upload:** 스트림, 파일 메소드 등을 통해 파일을 업로드한다.
- **AF.download:** 파일을 다운로드 하거나, 이미 진행중인 다운로드를 재개한다.
- **AF.request:** 파일 전송과 관련없는 기타 HTTP 요청.

이와 같은 Alamofire 메소드는 전역 메소드기 때문에 이들을 사용하기 위해 클래스 인스턴스를 따로 생성할 필요 없다.

## Requesting Data

먼저 `MainTableViewController.swift` 파일을 열고 다음 코드를 추가해주자:

```swift
import Alamofire
```

이제 view controller에서 Alamofire를 사용할 수 있다. 다음 코드를 추가해라:

```swift
extension MainTableViewController {
  func fetchFilms() {
    // 1
    let request = AF.request("https://swapi.dev/api/films")
    // 2
    request.responseJSON { (data) in
      print(data)
    }
  }
}
```

- Alamofire는 **namespacing**을 사용한다. 따라서 모든 호출에 대해 접두사를 붙여줘야 한다. `AF`. `request(_:method:parameters:encoding:headers:interceptor:)`는 데이터의 endpoint를 파라미터로 받는다. 더 많은 파라미터를 받을 수 있지만, 이번에는 단지 string 타입의 URL을 보내기만 하므로 기본 파라미터 값을 사용하였다.
- JSON 데이터를 받아서 출력한다.

`viewDidLoad()` 에서 이 `fetchFilms()`를 호출하면 네트워크 요청이 시작되고 다음의 결과가 console에 출력된다.

```
success({
  count = 7;
  next = "<null>";
  previous = "<null>";
  results =  ({...})
})
```

## Using a Codable Data Model

그렇다면 반환받은 JSON 데이터를 어떻게 사용할까? JSON 구조는 복잡하기 때문에 바로 사용하기에는 무리가 있다. 따라서 이 데이터를 저장할 model을 생성할 것이다.

프로젝트의 **Networking** 그룹에 `Film.swift` 파일을 생성하고 다음을 추가하자:

```swift
struct Film: Decodable {
  let id: Int
  let title: String
  let openingCrawl: String
  let director: String
  let producer: String
  let releaseDate: String
  let starships: [String]
  
  enum CodingKeys: String, CodingKey {
    case id = "episode_id"
    case title
    case openingCrawl = "opening_crawl"
    case director
    case producer
    case releaseDate = "release_date"
    case starships
  }
} 
```

이는 API의 데이터를 가져 오는 데 필요한 데이터 속성과 코딩 키를 만드는 코드이다. `Decodable`을 채택하여 JSON을 데이터 모델로 변환할 수 있다.

 

이 튜토리얼 프로젝트는 추후 사용될 세부 정보를 쉽게 표시하기 위한 `Displayable` 프로토콜을 정의한다. `Film`이 이 프로토콜을 채택하게 하고 다음을 `Film.swift` 에 추가하자:

```swift
extension Film: Displayable {
  var titleLabelText: String {
    title
  }
  
  var subtitleLabelText: String {
    "Episode \(String(id))"
  }
  
  var item1: (label: String, value: String) {
    ("DIRECTOR", director)
  }
  
  var item2: (label: String, value: String) {
    ("PRODUCER", producer)
  }
  
  var item3: (label: String, value: String) {
    ("RELEASE DATE", releaseDate)
  }
  
  var listTitle: String {
    "STARSHIPS"
  }
  
  var listItems: [String] {
    starships
  }
}
```

같은 프로젝트 그룹 내에 `Films.siwft` 파일을 추가하고 다음 코드를 추가해라:

```swift
struct Films: Decodable {
  let count: Int
  let all: [Film]
  
  enum CodingKeys: String, CodingKey {
    case count
    case all = "results"
  }
}
```

이는 `Film`의 집합을 나타낸다. 콘솔에서 출력된 것을 봤듯이 **[swapi.dev/api/films](http://swapi.dev/api/films)** 는 4개의 메인 값을 반환한다: `count`, `next`, `previous`, `results`. 이 앱에서는 `count`와 `results` 만 사용할 것이기 때문에 `Films`는 해당 값을 위한 프로퍼티만 가지고 있다.

Coding key는 서버의 `results` 를 `all` 로 변환한다. 이는 `Films.results` 보다  `Films.all`이 더 직관적이기 때문이다. `Films` 역시 `Decodable` 프로토콜을 채택하여 JSON 데이터를 데이터 모델로 변활할 수 있다.

다시 **MainTableViewController.swift** 로 돌아가서, `fetchFilms()` 를 다음과 같이 수정하자:

```swift
request.responseDecodable(of: Films.self) { (response) in
  guard let films = response.value else { return }
  print(films.all[0].title)
}
```

이제 JSON으로 변환하지 않고 내부 데이터 모델인 `Films` 로 변환할 것이다. 디버깅을 위해 콘솔에 출력하면 첫번째 영화의 제목이 출력될 것이다.

## Method Chaining

Alamofire는 **method chaining**을 사용한다. Method chaing은 한 메소드의 응답을 다른 메소드의 입력으로 사용하여 작업을 연결하는 것을 말한다. 이는 코드를 간결하고 더 깔끔하게 해준다.

`fetchFilms()` 내부의 코드를 다시한번 수정하자:

```swift
AF.request("https://swapi.dev/api/films")
  .validate()
  .responseDecodable(of: Films.self) { (response) in
    guard let films = response.value else { return }
    print(films.all[0].title)
  }
```

위에서 본 코드에 유효성 검사를 위한 코드가 추가되었다.

코드를 위에서부터 살펴보면, endpoint에 요청을 하고, 응답이 HTTP 상태 코드를 반환하는지 유효성 검사를 한 뒤 데이터 모델로 decode 하는 과정이다.

## Setting up Your Table View & Updating the Detail View Controller

간단한 Table view와 segue 대한 설명이기 때문에 넘어가겠습니다. 혹시 table view의 기초가 부족하시면 원문을 읽어보세요.

# Fetching Multiple Asynchronous Endpoints

`Film` 을 보면 `[String]`타입의  `starships` 프로퍼티를 볼 수 있다. 이 프로퍼티는 starship 데이터의 모든 정보를 포함하지 않고 데이터를 위한 endpoint 배열만을 포함한다. 이는 필요 이상의 데이터에 접근할 필요 없이 개발자가 원하는 데이터에만 접근하기 위해 사용하는 흔한 패턴이다. 우리는 그저 endpoint에 접근해서 필요한 정보만을 가져오면 된다.

## Creating a Data Model for Starships

Starship에 대한 데이터를 가져오기 위해 새로운 데이터 모델을 생성하자.

```swift
struct Starship: Decodable {
  var name: String
  var model: String
  var manufacturer: String
  var cost: String
  var length: String
  var maximumSpeed: String
  var crewTotal: String
  var passengerTotal: String
  var cargoCapacity: String
  var consumables: String
  var hyperdriveRating: String
  var starshipClass: String
  var films: [String]
  
  enum CodingKeys: String, CodingKey {
    case name
    case model
    case manufacturer
    case cost = "cost_in_credits"
    case length
    case maximumSpeed = "max_atmosphering_speed"
    case crewTotal = "crew"
    case passengerTotal = "passengers"
    case cargoCapacity = "cargo_capacity"
    case consumables
    case hyperdriveRating = "hyperdrive_rating"
    case starshipClass = "starship_class"
    case films
  }
}
```

다른 데이터 모델과 마찬가지로 관련 코딩 키와 함께 사용하려는 모든 응답 데이터를 나열하기 만하면 된다.

또한 각각의 starship에 대한 정보를 표현하기 위해서 `Starship`은 `Displayable` 프로토콜을 준수한다.

```swift
extension Starship: Displayable {
  var titleLabelText: String {
    name
  }
  
  var subtitleLabelText: String {
    model
  }
  
  var item1: (label: String, value: String) {
    ("MANUFACTURER", manufacturer)
  }
  
  var item2: (label: String, value: String) {
    ("CLASS", starshipClass)
  }
  
  var item3: (label: String, value: String) {
    ("HYPERDRIVE RATING", hyperdriveRating)
  }
  
  var listTitle: String {
    "FILMS"
  }
  
  var listItems: [String] {
    films
  }
}
```

## Fetching the Starship Data

Starship 데이터를 가져오기 위해서, 새로운 네트워크 호출이 필요하다. `DetailViewController.swift` 를 열고 네트워크 작업을 위한 코드를 추가하자.

```swift
extension DetailViewController {
  // 1
  private func fetch<T: Decodable & Displayable>(_ list: [String], of: T.Type) {
    var items: [T] = []
    // 2
    let fetchGroup = DispatchGroup()
    
    // 3
    list.forEach { (url) in
      // 4
      fetchGroup.enter()
      // 5
      AF.request(url).validate().responseDecodable(of: T.self) { (response) in
        if let value = response.value {
          items.append(value)
        }
        // 6
        fetchGroup.leave()
      }
    }
    
    fetchGroup.notify(queue: .main) {
      self.listData = items
      self.listTableView.reloadData()
    }
  }
}
```

이 코드는 다음과 같은 작업을 수행한다.

1. `Starship`은 film 리스트를 가진다. `Film`과 `Starship` 모두 `Displayable` 프토토콜을 준수하기 때문에 네트워크 요청을 위해서 제네릭을 사용한다. 결과를 적절하게 디코딩 하기 위해서 가져 오는 항목의 타입 만 알면 된다.
2. 이 작업에서는 여러 네트워크 요청이 발생한다. 이 작업들은 비동기적으로 이루어지고 순서도 지켜지지 않기 때문에 이를 위해 `DispatchGroup`을 사용한다. 이를 통해 모든 작업들이 끝나면 이를 알 수 있다.
3. list의 모든 항목을 반복한다.
4. `DispatchGroup`에 작업을 알린다.
5. Alamofire 네트워크 요청을 하여 유효성 검사를 하고 응답을 적절한 타입으로 디코딩 한다.
6. 요청의 completion handler 안에서, 작업의 완료를 알린다.
7. `DispatchGroup`이 모든 `enter()`에 대한 `leave()`를 받으면, 우리의 작업은 main queue에서 이루어져야 한다. 리스트를 `listData`에 저장하고 table view를 리로드한다.

네트워크 요청을 위한 코드를 작성했으니, `extension` 안에 다음 코드를 추가하자:

```swift
func fetchList() {
  // 1
  guard let data = data else { return }
  
  // 2
  switch data {
  case is Film:
    fetch(data.listItems, of: Starship.self)
  default:
    print("Unknown type: ", String(describing: type(of: data)))
  }
}
```

1. data는 옵셔널이기 때문에, `nil` 이 아님을 보장해야한다.
2. data의 타입을 사용하여 어떻게 helper 메소드를 호출할지 결정하자.
3. 만약 data 가 `Film`이면 `listItem`은 starship의 목록일 것이다.

## Updating Your Table View

table view와 관련한 내용은 넘어가도록 하겠습니다.

# Sending Parameters With a Request

검색 기능을 위해서, 검색 내용에 맞는 starship 리스트가 필요하다. starship 리스트를 가져오기 위해서는 검색 내용을 endpoint에 전달해야 한다. Alamofire의 `request`는 문자열 타입의 URL 뿐 아니라 키/값 쌍의 배열도 매개 변수로 받을 수 있다. 이를 이용해서 검색 내용을 endpoint에 전달하도록 하자.

## Decoding Starships

먼저, staship 리스트를 위한  `Starships` 데이터 모델을 구성하자.

```swift
struct Starships: Decodable {
  var count: Int
  var all: [Starship]
  
  enum CodingKeys: String, CodingKey {
    case count
    case all = "results"
  }
}
```

`Films`와 같은 형태를 띄는 것을 볼 수 있다.

다음은 `MainTableViewController.swift` 에 다음 코드를 추가하자:

```swift
func searchStarships(for name: String) {
  // 1
  let url = "https://swapi.dev/api/starships"
  // 2
  let parameters: [String: String] = ["search": name]
  // 3
  AF.request(url, parameters: parameters)
    .validate()
    .responseDecodable(of: Starships.self) { response in
      // 4
      guard let starships = response.value else { return }
      self.items = starships.all
      self.tableView.reloadData()
  }
}
```

1. 먼저 starship 정보에 접근하기 위한 URL을 설정한다.
2. Endpoint에 전달할 Key-value 파라미터 설정한다.
3. 이전에 했던 것 처럼 네트워크 요청을 구성하자. 이전과 다른 점은 URL에 parameter을 추가하는 것이다. `validate`와 `Starships`으로 디코딩 하는 과정도 진행한다.
4. 마지막으로 요청이 성공하면 starship 리스트를 table view의 데이터로 할당하고 table view를 리로드 한다.

요청을 수행하면 URL은 [`https://swapi.dev/api/starships?search={name}`](https://swapi.dev/api/starships?search=%7Bname%7D)의 형태가 된다. 여기서 `{name}` 에는 검색 질의가 전달 된다.

## Searching for Ships & Display a Ship’s List of Films

이 파트 역시 Alamofire에 관한 내용이 아니고 간단한 UI 관련 내용이기 때문에 넘어갑니다.

# Where to Go From Here?

Almofire에 관한 기본적인 내용들을 공부했다. 더 깊은 내용을 배우기 위해선 [Almofire 사이트](https://github.com/Alamofire/Alamofire)의 문서를 통해 공부하는 것을 추천한다. 또한 Almofire와 함께 Apple의 URLSession 공부를 하는 것도 추천한다.

## Reference

[Raywenderlich](https://www.raywenderlich.com/6587213-alamofire-5-tutorial-for-ios-getting-started)
