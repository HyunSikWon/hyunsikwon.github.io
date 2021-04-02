---
title: Core Location Tutorial for iOS - Tracking Visited Locations
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Core Location
last_modified_at: 2021-03-12 T15:21:00+08:00
---

## Introductions

이 튜토리얼은 iOS 위치기반 API인 Core Location을 사용한 간단한 여행 기록 앱이다.  

앱의 주요 기능은 다음과 같다:

- 사용자의 위치를 자동으로 추적한다(앱이 실행 중이 아닐때도).
- 앱이 새로운 위치를 기록하면, 사용자에게 로컬 알림을 보낸다.
- 위치들을 파일에 저장한다.
- 기록된 위치를 표시한다.
- 사용자의 위치와 기록된 위치가 보이는 지도가 표시된다.
- 사용자가 수동으로 위치를 기록할 수 있도록 한다.

## Getting Started

앱은 기록된 위치를 나타내는 테이블 뷰와 지도 뷰로 구성되며 탭바를 통해 구분된다.

### AppDelegate.swift

`AppDelegate.swift` 파일을 살펴보면, 두 개의 프레임워크를 import 했다.

```swift
import CoreLocation
import UserNotifications
```

`CoreLocation`은 사용자 위치의 업데이트 정보를 위함이고, `UserNotification`은 앱이 새로운 위치를 기록할때 알림 배너를 보여주기 위함이다. 

두 개의 프로퍼티도 볼 수 있다. 이 프로퍼티를 통해서 두 프레임워크의 API에 접근(access)할 것이다.

```swift
let center = UNUserNotificationCenter.current()
let locationManager = CLLocationManager()
```

### Location.swift

`Location.swift`은 모델 클래스이며, 다섯개의 프로퍼티를 가진다.

```swift
let latitude: Double
let longitude: Double
let date: Date
let dateString: String
let description: String
```

- `latitude`와 `longtitude`는 위치 좌표.
- `date`는 위치가 기록된 날짜.
- `dateString`은 날짜의 문자열 버전.
- `description`은 위치의 문자열 버전.

`Location.swift`은 클래스의 객체를 디스크에 저장하기 위해서 `Codable` 프로토콜을 채택한다.

### LocationsStorage.swift

`Location.swift`은 싱글톤 객체로 이를 통해 데이터를 앱의 문서 폴더에 저장할 것이다.

```swift
// 1
private let fileManager: FileManager
private let documentsURL: URL 

// 2
private(set) var locations: [Location]
```

1. 디스크로부터 읽기/쓰기 위해 사용되는 프로퍼티
2. 기록된 위치에 접근하기 위한 프로퍼티.

## Core Location: Asking for User Locations

첫번째로 사용자의 위치를 추적하기 위해선 반드시 허가(permission) 단계를 거쳐야한다. 

### Providing a Proper Description

위치 정보를 얻기 위해 다음을 `Info.plist` 파일에 추가해야 한다.

<img width="499" alt="Screen-Shot-2018-04-15-at-9 21 50-PM" src="https://user-images.githubusercontent.com/48352065/110900832-bc000780-8346-11eb-835b-b919f898b8f1.png">


앱은 사용자에게 허가 요청을 할때 이 문자열을 띄운다. 내용은 자유롭게 설정할 수 있지만 다음의 조건을 고려해야 한다.

- 사용자가 접근을 허가할 수 있도록 독려해야 한다.
- 위치 정보 사용에 대해서 사용자가 명확한 이유와 방법을 알아야 한다.
- 반드시 진실만을 말해야 한다.

### Asking for Locations Permissions

`AppDelegate.swift` 파일의 `application(_:didFinishLaunchingWithOptions:)` 내부에 다음의 코드를 추가하자:

```swift
locationManager.requestAlwaysAuthorization()
```

위 라인은 앱이 백그라운드/포그라운드 모두에서 위치 데이터에 접근하는 것에대한 허가 요청을 한다. 

> 만약 사용자가 접근 요청을 거부한다면, 개발자는 이 상황에 대한 적절한 처리가 반드시 필요함.

### Asking for Notifications Permissions

사용자에게 알림을 보내기 위해서도 허가 요청이 필요하다. 알림의 경우 다른 문자열을 추가할 필요는 없다. 위에서 추가한 코드 바로 위에 다음 코드를 추가하자:

```swift
center.requestAuthorization(options: [.alert, .sound]) { granted, error in
}
```

## Choosing the Most Appropriate Locations Data

Core Location 프레임워크는 다양한 방법을 통해 사용자의 위치를 추적하고 각각은 서로 다른 특징을 갖는다.

- Standard location services: 배터리 소모가 크다. 그만큼 위치 정확도가 높으며 네비게이션 혹은 피트니스 앱에 사용되면 좋다.
- Significnat locatoin changes: 중간 정도의 배터리 소모. 중간 정도의 정확도. 정지 상태에 대한 낮은 정확도.
- Regional monitoring: 적은 배터리 소모. 좋은 위치 정확도. 위치를 모니터할 수 있는 지역이 정해져 있다.

3가지 모두가 이 튜토리얼에서 만드는 앱에 적절한 방법이 아니다. 다행이도.. 한가지 다른 API를 사용할 수 있다.

### Visit Monitoring

방문 모니터링은 사용자가 잠시 머무른 위치를 추적한다. 새로운 방문이 감지되면 앱을 깨우며 에너지 효율도 좋고 위치 제약도 없다.

## Subscribe to Location Changes

본격적인 구현에 들어가보자.

### CLLocationManager

`AppDelegate.swift`의 위치 권한 요청 코드 밑에 다음의 코드를 추가하자:

```swift
locationManager.startMonitoringVisits()
locationManager.delegate = self
```

이를 통해 추적 기능을 시작하고, 위치 변화를 알리기위해 델리게이트를 사용한다.

익스텐션을 사용하여 다음의 코드를 추가하자:

```swift
extension AppDelegate: CLLocationManagerDelegate {
  func locationManager(_ manager: CLLocationManager, didVisit visit: CLVisit) {
    // create CLLocation from the coordinates of CLVisit
    let clLocation = CLLocation(latitude: visit.coordinate.latitude, longitude: visit.coordinate.longitude) 

    // Get location description
  }

  func newVisitReceived(_ visit: CLVisit, description: String) {
    let location = Location(visit: visit, descriptionString: description)

    // Save location to disk
  }
}
```

첫번째 메소드는 `CLLocationManager`로부터의 콜백 메소드이며 새로운 방문을 기록하고 `CLVisit`을 통해 정보를 제공한다. 

`CLVisit` 은 네가지 프로퍼티를 가진다.

1. `arrivalDate`: 방문 시작 날짜.
2. `departureDate`: 방문 종료 날짜.
3. `coordinate`: 디바이스가 방문한 지역의 중앙.
4. `horizontalAccuracy`: 방문 위치의 추정 반지름(미터 단위).

이 데이터를 사용해서 `Location` 객체를 생성해야 하고, 이 객체의 이니셜라이저는 `CLVisit`객체와 날짜 그리고 description 문자열을 필요로 한다.

```swift
init(_ location: CLLocationCoordinate2D, date: Date, descriptionString: String)
```

### Location Description

`descriptionString`을 위해서 `CLGeocoder`를 사용한다. Geocoding은 좌표를 실제 주소 혹은 이름으로 변환하는 과정이다. 좌표 집합에서 주소를 얻기 위해선 리버스 지오코딩을 사용하면 된다.

`AppDelegate.swift`에 다음 프로퍼티를 추가한다.

```swift
static let geoCoder = CLGeocoder()
```

`locationManager(_:didVisit:)`에 다음 코드를 추가한다:

```swift
AppDelegate.geoCoder.reverseGeocodeLocation(clLocation) { placemarks, _ in
  if let place = placemarks?.first {
    let description = "\(place)"
    self.newVisitReceived(visit, description: description)
  }
}
```

`geoCoder`를 사용해서 placemarks를 가질 수 있다. placemarks는 주소와 같은 좌표 값에 대한 유용한 정보들을 포함한다. 첫번째 placemark의 문자열을 통해 `description`을 생성하여 `newVisitReceived(_:description:)` 함수를 호출한다.

### Sending Local Notifications

이제 사용자에게 새로운 방문 위치가 기록되었다는 것을 알릴 차례이다. `ewVisitReceived(_:description:)`에 다음 코드를 추가한다.

```swift
// 1
let content = UNMutableNotificationContent()
content.title = "New Journal entry 📌"
content.body = location.description
content.sound = .default

// 2
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 1, repeats: false)
let request = UNNotificationRequest(identifier: location.dateString, content: content, trigger: trigger)

// 3
center.add(request, withCompletionHandler: nil)
```

1. 알림(notification) 컨텐츠를 생성한다.
2. 1초 길이의 트리거를 생성하고, 알림 요청을 이 트리거와 함께 생성한다.
3. notification center에 요청을 추가하여 알림을 예약한다.

## Persisting Location Data

방문 위치를 `Codable` 프로토콜을 통해 JSON으로 인코딩한 뒤 저장해보자.

### Saving Records on Disk

 `LocationsStorage.swift`에 다음 함수를 추가한다.

```swift
func saveLocationOnDisk(_ location: Location) {
  // 1
  let encoder = JSONEncoder()
  let timestamp = location.date.timeIntervalSince1970

  // 2
  let fileURL = documentsURL.appendingPathComponent("\(timestamp)")

  // 3
  let data = try! encoder.encode(location)

  // 4
  try! data.write(to: fileURL)

  // 5
  locations.append(location)
}
```

1. 인코더 생성.
2. 파일의 URL은 설정. 파일 이름은 날짜의 타임 스탬프를 사용.
3. `Location` 객체를 로우 데이터로 변환.
4. 데이터를 파일에 쓴다.
5. 저장된 위치를 로컬 배열에 추가.

> 편의상 변환과 쓰기가 항상 성공한다고 가정함. 실제 개발에선 옵셔널을 통해 에러 처리를 해줘야 함.

`AppDelegate.swift`파일의  `newVisitReceived(_:description:)`에 저장 코드를 추가한다.

```swift
LocationsStorage.shared.saveLocationOnDisk(location)
```

### Saving a Current Location

현재 위치를 추가하기 위해서 `MapViewController.swift` 파일에 있는 `addItemPressed(_:)` 메소드에 다음 코드를 추가한다.

```swift
guard let currentLocation = mapView.userLocation.location else {
  return
}

LocationsStorage.shared.saveCLLocationToDisk(currentLocation)
```

`LocationsStorage.swift`에 `saveCLLocationToDisk(_:)` 메소드를 새로 만든다. 

```swift
func saveCLLocationToDisk(_ clLocation: CLLocation) {
  let currentDate = Date()
  AppDelegate.geoCoder.reverseGeocodeLocation(clLocation) { placemarks, _ in
    if let place = placemarks?.first {
      let location = Location(clLocation.coordinate, date: currentDate, descriptionString: "\(place)")
      self.saveLocationOnDisk(location)
    }
  }
}
```

`clLocation`, 현재 날짜, 위치 설명을 통해 `Location` 객체를 생성하고 같은 방법으로 저장한다.

이제 `LocationsStorage` 의 이니셜라이저에 있는 다음 코드를 수정한다.

```swift
self.locations = []
```

초기화 과정에서 저장된 데이터들을 불러오는 코드로 바꾼다:

```swift
let jsonDecoder = JSONDecoder()

// 1
let locationFilesURLs = try! fileManager
  .contentsOfDirectory(at: documentsURL, includingPropertiesForKeys: nil)
locations = locationFilesURLs.compactMap { url -> Location? in
  // 2
  guard !url.absoluteString.contains(".DS_Store") else {
    return nil
  }
  // 3
  guard let data = try? Data(contentsOf: url) else {
    return nil
  }
  // 4
  return try? jsonDecoder.decode(Location.self, from: data)
  // 5
  }.sorted(by: { $0.date < $1.date })
```

1. Documents 폴더에 모든 파일 URL을 가져온다.
2. `.DS_Store` 파일은 스킵한다.
3. 파일로부터 데이터를 읽는다.
4. 로우 데이터를 `Location` 객체로 디코딩한다. 
5. 날짜를 기준으로 정렬한다.

앱이 시작되면 이 코드를 통해서  `LocationStorage`가 디스크로부터 모든 위치 데이터를 갖게 된다.

## Setting up the App to Use Stored Data

### Setting up a Table View

지금까지의 작업들을 UI로 나타내보자.

테이블 뷰의 셀 데이터 표시.

```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
  let cell = tableView.dequeueReusableCell(withIdentifier: "PlaceCell", for: indexPath)
  let location = LocationsStorage.shared.locations[indexPath.row]
  cell.textLabel?.numberOfLines = 3
  cell.textLabel?.text = location.description
  cell.detailTextLabel?.text = location.dateString
  return cell
}
```

### Updating the List When a Location is Logged

리스트가 항상 최신 상태를 유지하려면, 앱이 새로운 위치가 생성된 것을 알도록  알려야한다. 이는 `UNNotification`를 사용하는 것이 아닌 `Notification`을 사용한다는 것을 주의해라. 이 알림은(notification)은 앱의 내부에서 사용되는 것으로 사용자를 위한 알림이 아니다.

> 특정 객체가 NotificationCenter에 등록된 이벤트를 발생시키면, 그 이벤트를 처리한다고 약속한 옵저버가 이벤트에 대한 동작을 수행하는 것이 기본적인 Notification의 작동 방식이다. 특정 객체가 이벤트를 발생시키는 것을 포스트(post)라고 한다.

`LocationsStorage.swift` 파일에 다음 코드를 추가하자.

```swift
extension Notification.Name {
  static let newLocationSaved = Notification.Name("newLocationSaved")
}
```

이는 우리가 발생시킬(post) 알림이다.

이제 `saveLocationOnDisk(_:)`에 다음 코드를 추가하자.

```swift
NotificationCenter.default.post(name: .newLocationSaved, object: self, userInfo: ["location": location])
```

이제 발생한 알림을 처리할 옵저버가 필요하다.

`PlacesTableViewController`가 이 알림을 처리해야 한다. `PlacesTableViewController.swift` 파일에 다음 코드를 추가하자.

```swift
override func viewDidLoad() {
  super.viewDidLoad()

  // 1
  NotificationCenter.default.addObserver(
    self, 
    selector: #selector(newLocationAdded(_:)), 
    name: .newLocationSaved, 
    object: nil)
}

// 2
@objc func newLocationAdded(_ notification: Notification) {
  // 3
  tableView.reloadData()
}
```

1. 알림이 도착하면 호출될 메소드를 등록한다.
2. 파라미터로 알림을 받는다.
3. 데이터를 리로드 한다.

### Setting up MapView With All Logged Locations

이 튜토리얼의 마지막 파트는 모든 데이터를 지도에 나타내는 것이다.

지도에 핀을 추가히려면, 위치를 `MKAnnotation`으로 변환해야 한다. `MKAnnotatoin`은 지도에서 표현되는 프로토콜이다.

`MapViewController.swift` 파일에서 다음 메소드를 추가하자:

```swift
func annotationForLocation(_ location: Location) -> MKAnnotation {
  let annotation = MKPointAnnotation()
  annotation.title = location.dateString
  annotation.coordinate = location.coordinates
  return annotation
}
```

타이틀과 좌표를 통해서 핀 어노테이션을 생성한다.

`viewDidLoad()`에 다음 코드를 추가하자:

```swift
let annotations = LocationsStorage.shared.locations.map { annotationForLocation($0) }
mapView.addAnnotations(annotations)
```

이는 핀을 생성해서 지도에 추가하는 코드이다.

마지막으로 새로운 위치가 기록되면 핀을 추가하는 기능을 구현하자. Notification을 사용한다. 다음 메소드를 추가하자

```swift
@objc func newLocationAdded(_ notification: Notification) {
  guard let location = notification.userInfo?["location"] as? Location else {
    return
  }

  let annotation = annotationForLocation(location)
  mapView.addAnnotation(annotation)
}
```

`viewDidLoad`에 새로운 옵저버를 추가한다.

```swift
NotificationCenter.default.addObserver(
  self, 
  selector: #selector(newLocationAdded(_:)), 
  name: .newLocationSaved, 
  object: nil)
```

## Where to Go From Here?

이번 주제에 대한 더 깊은 이해를 위한 자료:

- [WWDC 2014 session about location services](https://developer.apple.com/videos/play/wwdc2014/706/)
- [Apple documentation about `CLVisit`](https://developer.apple.com/documentation/corelocation/clvisit)
- [MapKit tutorial](https://www.raywenderlich.com/160517/mapkit-tutorial-getting-started)

## Reference

[Raywenderich](https://www.raywenderlich.com/5247-core-location-tutorial-for-ios-tracking-visited-locations)
