---
title: AVFoundation
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- AVFoundation
last_modified_at: 2020-12-16 T22:37:00+08:00

---

[Apple의 Media Playback Programming Guide](https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/MediaPlaybackGuide/Contents/Resources/en.lproj/ExploringAVFoundation/ExploringAVFoundation.html#//apple_ref/doc/uid/TP40016757-CH4-SW1), [Apple 개발자 문서](https://developer.apple.com/documentation/avfoundation/avasset)를 번역 및 정리하며 AVFoundation을 공부하였습니다.

## **AVFoundation**

`AVFoundatoin`은 iOS, tvOS, macOS를 위한 미디어 프레임 워크이다. 우리는 이 프레임 워크를 편집, low-level 처리 등 다양한 미디어 처리 작업을 위해 사용할 수 있는데, 특히 media 재생을 위해서 가장 많이 사용된다. `AVFoundation`을 사용하면 MP3 오디오 파일 부터 HTTP Live Streaming을 통한 미디어 파일 등 다양한 미디어 자원을 효율적으로 로드하고 관리 할 수 있다.

![avfoundation1](https://user-images.githubusercontent.com/48352065/94444267-1cf4b280-01e1-11eb-89be-f46488e4b2f1.png)

## **AVAsset**

`AVFoundation` 프레임 워크는 `AVAsset` 클래스를 사용해서 자원들을 모델링 한다. `AVAsset` 클래스는 추상 클래스로 단일 미디어 자원을 나타내는 불변의 타입이다. `AVAsset`의 인스턴스는 로컬에 있는 미디어 파일을 구성할 수 있고 원격에서 다운로드 받은 파일, HLS를 통해 스트리밍 되는 파일 또한 표현할 수 있다.

`AVAsset`은 두 가지 방식으로 미디어 작업을 단순화한다. 먼저, 미디어의 포맷으로부터의 독립성을 제공한다. 미디어 포맷 타입과 관련없이 미디어 파일을 관리할 수 있게 도와주고 컨테이너 포맷과 관련한 세부 작업이나 코덱 작업은 프레임워크로 넘겨준다. 두번째로, 미디어의 위치로부터의 독립성을 제공한다. Asset 인스턴스를 미디어의 URL을 통해 초기화 할 수 있는데, 이 URL은 앱 번들이나 파일 시스템 어딘가에 있는 로컬 url 일수도 있고, 원격 host에서 제공될 수도 있다.이러한 **미디어 포맷으로부터의 독립성, 미디어의 위치로부터의 독립성**은 개발자의 시청각 미디어 작업을 크게 단순화해준다.


`AVAsset`은 하나 이상의 `AVAssetTrack` 인스턴스로 구성된다. 각 트랙은 tracks 프로퍼티로 접근할 수 있다.

![avfoundation2](https://user-images.githubusercontent.com/48352065/94444279-2120d000-01e1-11eb-8051-e8a4758f9428.png)

## **Creating an Asset**

`AVAsset`은 미디어 자원에 대한 로컬 혹은 원격 URL을 이용해 초기화된다.

```swift
let url: URL = // Local or Remote Asset URL
let asset = AVAsset(url: url)
```

`AVAsset`은 추상 클래스기 때문에 위와 같은 코드는 실제로는 서브클래스인 `AVURLAsset`의 인스턴스가 생성된다. 대부분의 경우 위와 같이 asset을 생성하는게 적절하지만, 좀 더 세밀한 초기화를 위해선 다음과 같이  `AVURLAsset`의 인스턴스를 직접적으로 생성할 수 있다. 이때, `AVURLAsset`에 `options` 파라미터가 있는데, 특정한 상황에 맞게 조정하는 역할을 한다. 예를 들어, HLS stream을 위한 asset을 생성할 때, 사용자가 cellular 네트워크를 통해서 미디어를 가져오는 것을 막고싶다면 다음과 같은 코드를 작성하면 된다.

```swift
let url: URL = // Remote Asset URL
let options = [AVURLAssetAllowsCellularAccessKey: false]
let asset = AVURLAsset(url: url, options: options)
```

`AVURLAssetAllowsCellularAccessKey`의 값으로 `false`를 전달하면, 사용자가 Wi-Fi를 통해서만 미디어 자원을 가져올 수 있다.

## **Preparing Asset For Use**

`AVAsset`의 프로퍼티들을 사용하여 미디어를 재생하는데 적절한지 여부, 길이, 생성일, meta data와 같은 asset의 특성과 기능 등을 결정할 수 있다. 프로퍼티의 값들은 asset을 생성할 때 자동으로 생성되는 것이 아니라 값들이 요청될 때 로드된다. 프로퍼티에 접근하는 것은 synchronous하기 때문에 만약 프로퍼티의 값이 이전에 로드된적이 없다면, framework는 값을 반환하기 위해 많은 양의 작업을 수행해야 할 것이다. 만약, 로드된 적 없는 프로퍼티 값을 요청하는 작업이 오래 블럭된다면 이는 미디어 작업에 큰 문제를 줄 수 있다. 따라서 asset의 프로퍼티 값을 불러오는 것은 반드시 asynchronous하게 해야 한다.

`AVAsset`과 `AVAssetTrack`은 `AVAsynchronousKeyValueLoading` 프로토콜을 채택한다. 이 프로토콜은 현재 프로퍼티의 상태와 비동기적으로 하나 이상의 프로퍼티 값을 불러오는 메소드를 정의한다.

```swift
public func loadValuesAsynchronously(forKeys keys: [String], completionHandler handler: (() -> Void)?)
public func statusOfValue(forKey key: String, error outError: NSErrorPointer) -> AVKeyValueStatus
```


`loadValuesAsynchronouslyForKeys:completionHandler:` 메소드는 비동기적으로 프로퍼티 값을 불러오는데 사용하고 불러올 프로퍼티들의 이름이 담긴 keys 배열과 상태가 결정되면 호출되는 completion 블럭을 전달한다. `statusOfValueForKey:error:` 메소드는 프로퍼티 값이 성공적으로 로드되어 즉시 사용가능한지 확인할 수 있는 메소드다.

예제코드:

```swift
// URL of a bundle asset called 'example.mp4'
let url = Bundle.main.url(forResource: "example", withExtension: "mp4")!
let asset = AVAsset(url: url)
let playableKey = "playable"
 
// Load the "playable" property
asset.loadValuesAsynchronously(forKeys: [playableKey]) {
    var error: NSError? = nil
    let status = asset.statusOfValue(forKey: playableKey, error: &error)
    switch status {
    case .loaded:
        // Sucessfully loaded. Continue processing.
    case .failed:
        // Handle error
    case .cancelled:
        // Terminate processing
    default:
        // Handle all other cases
    }
}
```

## **Working with Metadata**

미디어 컨테이너 포맷은 미디어에 대한 metadata를 담고있다. 컨테이너 포맷은 각각의 고유한 메타데이터 포맷을 가지고, 로우 레벨한 이해가 필요해서 메타데이터를 통한 작업은 매우 어려운 일이다, 그러나  `AVFoundation`은 `AVMetaDataItem` 클래스를 이용해서 metadata와 관련한 작업을 단순화 해준다. `AVMetaDataItem`의 인스턴스는 key-value 쌍으로 이루어지면서 영화의 제목 등과 같은 단일 metadata를 표현한다.

### Retrieving a Collection of Metadata

`AVFoundation` 프레임워크는 metadata를 쉽게 찾고 구분하기위해서 관련된 metadata를 key spaces로 그룹화 한다.

**Format-specific key spaces:** QuickTime이나 MP3같은 특정 container 혹은 file format과 연관된 metadata의 키 값이다. 하나의 `AVAsset` 메타데이터는 여러 key spaces에 분포되어 존재할 수 있다. 이렇게 여러 key spaces에 분포되어있는 하나의 `AVAsset` 메타데이터들 집합은 `metadata` 프로퍼티를 통해 접근이 가능하다.

**Common key space:** 영화의 개봉일 같은 포맷에 상관없는 여러 공통 정보들과 관련된 key로 `commonMetadata` 프로퍼티로 접근할 수 있다.

asset이 어떤 metadata를 포함하고 있는지는 `availableMetadataFormats` 프로퍼티를 통해 알 수 있다. 이 프로퍼티는 metadata를 구분하는 식별자를 담은 문자열 타입의 배열을 반환해준다. format-specific metadata를 얻기 위해선 다음과 같이 적절한 format 식별자를 파라미터로 보내는 `metadataForFormat:` 메소드를 이용한다:

```swift
let url = Bundle.main.url(forResource: "audio", withExtension: "m4a")!
let asset = AVAsset(url: url)
let formatsKey = "availableMetadataFormats"

asset.loadValuesAsynchronously(forKeys: [formatsKey]) {
    var error: NSError? = nil
    let status = asset.statusOfValue(forKey: formatsKey, error: &error)
    if status == .loaded {
        for format in asset.availableMetadataFormats {
            let metadata = asset.metadata(forFormat: format)
            // process format-specific metadata collection
        }
    }
}

```

### Finding and Using Metadata Values

metadata의 collections을 얻었으면 이젠 특정 metadata에 접근할 차례다. `AVMetadataItem`의 다양한 클래스 메소드를 이용해서 원하는 값들을 구할 수 있다. 가장 쉬운 방법은 identifier을 이용하는 방법이다. 

```swift
let metadata = asset.commonMetadata
let titleID = AVMetadataCommonIdentifierTitle
let titleItems = AVMetadataItem.metadataItems(from: metadata, filteredByIdentifier: titleID)
if let item = titleItems.first {
    // process title item
}
```

필요한 metadata를 얻었으면 다음은 그것의 value 프로퍼티를 활용할 차례다. 반환된 타입은 `NSObject`와 `NSCopying` 프로토콜을 채택한 객체 타입이고 원하는 적절한 타입으로 변환할 수 있다. 하지만 metadata의 [stringValue](https://developer.apple.com/documentation/avfoundation/avmetadataitem/1390846-stringvalue),[numberValue](https://developer.apple.com/documentation/avfoundation/avmetadataitem/1390681-numbervalue),[dateValue](https://developer.apple.com/documentation/avfoundation/avmetadataitem/1385563-datevalue), and [dataValue](https://developer.apple.com/documentation/avfoundation/avmetadataitem/1387641-datavalue) 프로퍼티를 사용하는게 더 안전하고 쉽다.

다음 예제코드에선 `dataValue` 프로퍼티를 사용한다.

```swift
// Collection of "common" metadata
let metadata = asset.commonMetadata
// Filter metadata to find the asset's artwork
let artworkItems =
    AVMetadataItem.metadataItems(from: metadata,
                                 filteredByIdentifier: AVMetadataCommonIdentifierArtwork)
if let artworkItem = artworkItems.first {
    // Coerce the value to an NSData using its dataValue property
    if let imageData = artworkItem.dataValue {
        let image = UIImage(data: imageData)
        // process image
    } else {
        // No image data found
    }
}
```

## **Playing Media**

Asset은 미디어 재생을 위해 필수적이지만 이는 하나의 모델로, 실질적인 재생을 위해선 추가적인 객체들이 필요하다. 이제 미디어 재생을 위해 필요한 객체들을 살펴보고 미디어 재생을 위해 어떻게 객체들을 구성하는지 알아보자.

![avfoundation3](https://user-images.githubusercontent.com/48352065/94444295-24b45700-01e1-11eb-8567-f6136a08b679.png)

### AVPlayer

`AVPlayer`은 media asset의 재생을 위한 중심 클래스이고, media asset의 시간과 재생을 관리한다. 로컬, 다운로드, 스트리밍 등을 할 때 사용할 수 있다.

`AVPlayer`은 한번에 하나의 media asset을 재생할 수 있다. 연속적으로 여러 asset을 관리하기 위해선 `AVPlayer`의 subclass인 `AVQueuePlayer`를 사용해야 한다.

### AVPlayerItem

`AVAsset`은 길이, 생성 날짜와 같은 미디어의 정적인 측면만 모델링하기 때문에 실제 재생을 위해선 동적인 성격인 `AVPlayerItem` 인스턴스가 필요하다. 이는 `AVPlayer`에 의해 재생되는 asset의 재생 상태, 타이밍을 모델링한다. `AVPlayerItem`의 메소드와 프로퍼티를 이용하면 미디어의 다양한 시간대를 탐색할 수 있고, 크기, 미디어의 현재 시간 등을 찾을 수 있다.

### AVKit and AVPlayerLayer

`AVPlayer`와 `AVPlayerItem`은 눈에 보이지 않는 객체이고 이들 자체로는 미디어의 영상을 화면에 표시할 수 없다. 미디어의 영상을 화면에 표시할 수 있는 두 가지 방법이 존재한다.

- **AVKit:** 영상을 화면에 표시하기 위해선, `AVKit` 프레임워크의 `AVPlayerViewController`를 iOS, tvOS에서, `AVPlayerView`를 macOS에서 사용하는 것이 가장 좋은 방법이다. 이 객체들은 다양한 기능을 제공한다.

- **AVPlayerLayer:** 커스텀 플레이어를 만들고 싶다면 `AVFoundation`이 제공하는 `AVPlayerLayer`을 사용하면 된다. Player layer를 뷰의 하위 layer나 layer 계층 구조에 추가할 수 있다. 하지만 재생 관리를 위한 기능을 제공하지 않고 화면에 보여주는 역할만 하기 때문에 play, pause 버튼과 같은 컨트롤러는 직접 구현해야 한다.

### Setting Up the Playback Objects

재생을 위한 객체 생성 과정 코드:

```swift
class PlayerViewController: UIViewController {
 
    @IBOutlet weak var playerViewController: AVPlayerViewController!
 
    var player: AVPlayer!
    var playerItem: AVPlayerItem!
 
    override func viewDidLoad() {
        super.viewDidLoad()
 
        // 1) Define asset URL
        let url: URL = // URL to local or streamed media
 
        // 2) Create asset instance
        let asset = AVAsset(url: url)
 
        // 3) Create player item
        playerItem = AVPlayerItem(asset: asset)
 
        // 4) Create player instance
        player = AVPlayer(playerItem: playerItem)
 
        // 5) Associate player with view controller
        playerViewController.player = player
    }
 
}
```

재생 객체가 생성되면 player의 play 메소드를 이용해 재생을 시작할 수 있다.

### **Oberving Playback State**

`AVPlayer`와 `AVPlayerItem`은 상태가 빈번하게 변화하는 동적인 객체이다. 객체의 변화에 따라 다양한 동작을 취할 수 있는데 이는 KVO를 사용해서 이루어진다. KVO는 한 객체가 다른 객체의 상태를 관찰할 observer를 생성하고 객체의 상태 변화가 생기면 observer가 세부 상태를 알리는 형태로 동작한다. 이러한 KVO를 사용하여 `AVPlayer`과 `AVPlayerITem`의 상태 변화를 추적하고 그에따른 적절한 조치를 취할 수 있다. 

`AVPlayerItem`의 가장 중요한 프로퍼티 중 하나는 `status`다. `status`는 현재 플레이어의 미디어 데이터가 재생할 준비가 되어있는지를 판단할 때 사용한다. player item을 처음 생성하면 `status` 값은 `AVPlayerItemStatusUnknow`의 값이고 이는 media가 아직 완전히 로드가 되지 않았거나 재생할 준비가 안됐다는 의미이다. player item이 재생될 준비가 되면 `status`는 `AVPlayerItemStatusReadyToPlay`로 변한다. 다음 코드는 상태 변화를 어떻게 추적하는지를 보여준다.

```swift
let url: URL = // Asset URL
 
var asset: AVAsset!
var player: AVPlayer!
var playerItem: AVPlayerItem!
 
// Key-value observing context
private var playerItemContext = 0
 
let requiredAssetKeys = [
    "playable",
    "hasProtectedContent"
]
 
func prepareToPlay() {
    // Create the asset to play
    asset = AVAsset(url: url)
 
    // Create a new AVPlayerItem with the asset and an
    // array of asset keys to be automatically loaded
    playerItem = AVPlayerItem(asset: asset,
                              automaticallyLoadedAssetKeys: requiredAssetKeys)
 
    // Register as an observer of the player item's status property
    playerItem.addObserver(self,
                           forKeyPath: #keyPath(AVPlayerItem.status),
                           options: [.old, .new],
                           context: &playerItemContext)
 
    // Associate the player item with the player
    player = AVPlayer(playerItem: playerItem)
}
```

prepareToPlay 메소드는 player item의 상태를 추적하기위해 `addObserver:forKeyPath:options:context:` 메소드를 사용한다. item의 모든 상태변화를 추적하기 위해 이 메소드를 반드시 player item과 player의 연결 전에 사용해야 한다. 상태 변화에 따른 알림을 받기 위해선 `observeValueForKeyPath:ofObject:change:context:` 메소드를 구현해야한다. 이 메소드는 상태 변화가 이루어질 때마다 호출되어 그에 따른 적절한 조치를 취할 수 있다.


apple의 예제 코드:

```swift
override func observeValue(forKeyPath keyPath: String?,
                           of object: Any?,
                           change: [NSKeyValueChangeKey : Any]?,
                           context: UnsafeMutableRawPointer?) {
 
    // Only handle observations for the playerItemContext
    guard context == &playerItemContext else {
        super.observeValue(forKeyPath: keyPath,
                           of: object,
                           change: change,
                           context: context)
        return
    }
 
    if keyPath == #keyPath(AVPlayerItem.status) {
        let status: AVPlayerItemStatus
        if let statusNumber = change?[.newKey] as? NSNumber {
            status = AVPlayerItemStatus(rawValue: statusNumber.intValue)!
        } else {
            status = .unknown
        }
        // Switch over status value
        switch status {
        case .readyToPlay:
            // Player item is ready to play.
        case .failed:
            // Player item failed. See error.
        case .unknown:
            // Player item is not yet ready.
        }
    }
}
```

## **Performing Time-Based Operations**

미디어 재생은 시간 기반 작업이고 `AVPlayer`와 `AVPlayerItem`의 대부분의 특징은 미디어의 시간을 관리하는 것과 연관되어있다. 이러한 특징들을 효율적으로 사용하기 위해선 `AVFoundation`에서 시간이 어떻게 표현되는지 이해해야 한다.

`AVFoundation`의 몇몇을 포함한 많은 Apple 프레임워크에선 시간을 초를 나타내는 부동소수점의 `NSTimeInterval` 값으로 표현하고. 많은 경우에는 이 값을 통해 시간을 표현하기가 쉽지만 디테일한 시간의 작업이 필요한 미디어 재생에서는 종종 문제가 생긴다. 이런 문제를 해결하기 위해 `AVFoundation`은 Core Media 프레임워크의 `CMTime` 데이터 타입을 사용해 시간을 표현한다.

```swift
public struct CMTime {
    public var value: CMTimeValue
    public var timescale: CMTimeScale
    public var flags: CMTimeFlags
    public var epoch: CMTimeEpoch
}
```
- CMTimeValue : 64 비트 정수로 분수로 표현되는 시간의 분자 값을 정의
- CMTimeScale : 32비트 정수로 분모 값을 정의

이 구조체는 시간을 유리수나 분수의 형태로 표현한다. `CMTime`의 두 가지 중요한 속성은 `value`와 `timescale` 이다. 이 구조체를 통해 미디어의 frame rate혹은 sample rate 측면에서의 시간을 쉽게 표현할 수 있다.


```swift
// 0.25 seconds
let quarterSecond = CMTime(value: 1, timescale: 4)
 
// 10 second mark in a 44.1 kHz audio file
let tenSeconds = CMTime(value: 441000, timescale: 44100)
 
// 3 seconds into a 30fps video
let cursor = CMTime(value: 90, timescale: 30)
```

### **Observing Time**

미디어의 재생 과정에서 재생 위치를 업데이트하거나 UI의 상태와 일치시키기 위해 재생 시간을 추적한다. KVO를 통해 재생 객체의 상태 변화를 관찰하는 것을 앞에서 살펴 봤지만 지속적인 상태 변화를 추적하는데는 적절한 기법이 아니다. 대신 `AVPlayer`는 player의 시간 변화를 추적하는 두 가지 방법을 제공한다: periodic observations, boundary observations.

- Periodic Observations

일정한 주기에 따라 시간 변화를 추적하는 기법이다. custom player를 만들 때 가장 흔하게 사용되는 경우는 주기적인 관찰을 통해 시간을 표시하는 UI를 업데이트 하는 경우다.

일정한 주기에 따라 시간 변화를 추적하기 위해선 player의 `addPeriodicTimeObserverForInterval:queue:usingBlock:` 메소드를 사용한다. 이 메소드는 `CMTime`으로 표현되는 시간 주기, serial dispatch queue, 일정 주기마다 실행될 call back 블럭을 사용한다. 다음 코드는 보통의 재생에서 0.5초 마다 호출되는 블럭을 어떻게 구성하는지 보여준다.

```swift
var player: AVPlayer!
var playerItem: AVPlayerItem!
var timeObserverToken: Any?
 
func addPeriodicTimeObserver() {
    // Notify every half second
    let timeScale = CMTimeScale(NSEC_PER_SEC)
    let time = CMTime(seconds: 0.5, preferredTimescale: timeScale)
    timeObserverToken = player.addPeriodicTimeObserver(forInterval: time,
                                                       queue: .main) {
        [weak self] time in
        // update player transport UI
    }
}
 
func removePeriodicTimeObserver() {
    if let timeObserverToken = timeObserverToken {
        player.removeTimeObserver(timeObserverToken)
        self.timeObserverToken = nil
    }
}

```

- Boundary Observations 

이 방법은 미디어의 타임라인 중 원하는 다양한 시점을 정의하고 이 지점에서 call back 되는 기법이다. 원하는 시점을 추적하기 위해서 `addBoundaryTimeObserverForTimes:queue:usingBlock:` 메소드를 사용한다. 이 메소드는 원하는 시점을 담은 `NSValue` 객체 배열과 serail diaptch queue, call back 블럭을 가진다. 

예제  코드:

```swift
var asset: AVAsset!
var player: AVPlayer!
var playerItem: AVPlayerItem!
var timeObserverToken: Any?
 
func addBoundaryTimeObserver() {
 
    // Divide the asset's duration into quarters.
    let interval = CMTimeMultiplyByFloat64(asset.duration, 0.25)
    var currentTime = kCMTimeZero
    var times = [NSValue]()
 
    // Calculate boundary times
    while currentTime < asset.duration {
        currentTime = currentTime + interval
        times.append(NSValue(time:currentTime))
    }
 
    timeObserverToken = player.addBoundaryTimeObserver(forTimes: times,
                                                       queue: .main) {
        // Update UI
    }
}
 
func removeBoundaryTimeObserver() {
    if let timeObserverToken = timeObserverToken {
        player.removeTimeObserver(timeObserverToken)
        self.timeObserverToken = nil
    }
}

```

### **Seeking Media**

사용자는  미디어  재생을  처음부터  순차적으로  재생할  뿐  아니라  원하는  다양한  지점에서  부터  재생하거나  특정  부분만을  재생하기도  한다. `AVKit`이  이를  위한  기능을  제공하긴  하지만, 커스텀 player을  만든다면  이  작업을  모두  구현해줘야  한다.

이를  위한  가장  흔한  방법은 `seekToTime:` 메소드를  사용하는  것이다. 이  메소드는  다음과  같이  이동하고자  하는  목적  시간을  전달한다.

```swift
// Seek to the 2 minute mark
let time = CMTime(value: 120, timescale: 1)
player.seek(to: time)
```

이  메소드는  빠르고  편리한  방법이긴  하지만  정확성보다는  빠르기에  초점을  맞춘다. 그렇기  때문에  이  메소드를  사용하면  원하는  시간대와는 차이가 조금 날 수 있다. 보다  정확한  방법을  원한다면 `seekToTime:toleranceBefore:toleranceAfter:`  메소드를  사용하면  된다. 이  메소드는  목적  시간대와  오차범위를 지정할  수  있다. 원하는  가장  정확한  목적  시간을  원하면  오차  범위를 0으로  주면  된다. 다만 이  함수는  추가적인 decoding을 위한 지연이 발생할  수  있다.

```swift
// Seek to the first frame at 3:25 mark
let seekTime = CMTime(seconds: 205, preferredTimescale: Int32(NSEC_PER_SEC))
player.seek(to: seekTime, toleranceBefore: kCMTimeZero, toleranceAfter: kCMTimeZero)
```

