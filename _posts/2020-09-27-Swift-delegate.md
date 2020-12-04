---
title: Delegate Pattern
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- delegate
last_modified_at: 2020-09-27 T13:45:00+08:00

---

Delegate Pattern에 대한 공부를 위해서 [LearnAppMacking](https://learnappmaking.com/delegation-swift-how-to/)의 Delegation에 대한 글을 번역 및 정리한 글입니다.

더 정확하고 디테일한 정보를 원하시면 사이트에 들어가셔서 원문을 모두 읽어보시는 것을 추천합니다.

잘못된 번역 및 표현 등이 있을 수 있으니, 자유롭게 피드백 주세요.

### **What is Delegation?**

Apple의 공식 개발자 문서는 *delegation*을 다음과 같이 정의한다:

Delegation은 class가 몇몇 class의 책임(작업을 말하는 듯)을 다른 클래스의 인스턴스에 위임하는 것을 할 수 있게 하는 디자인 패턴이다.

이를  현실 세계에서 일어나는 일로 가정해보자. 나와 친구는 초콜릿 쿠키를 어느 이벤트에 배달하는 팀이다. 나는 쿠키를 굽는 역할을 맡았고, 쿠키의 도우를 만드는 것을 친구에게 _위임했다._ 친구는 도우를 만들고 나에게 주기만 하면 되고, 나는 그것을 사용해서 쿠키를 굽기만 하면 된다.

이를 프로그래밍으로 생각해보면, 하나의 클래스가 어떤 작업을 다른 클래스에 위임하고, 그것의 책임을 전달하는 것으로 생각할 수 있다.

### **Delegation: A simple Example in Swift**

먼저, struct 타입의 Cookie와  Bakery class를 정의하자

```swift
struct Cookie {
    var size: Int = 5
    var hasChocolateChips: Bool = false
}

class Bakery {
    func makeCookie() {
        var cookie = Cookie()
        cookie.size = 6
        cookie.hasCholocateChips = true
    }
}
```

여기서 우리는 3가지 방식으로 쿠키를 팔고 싶다.

1\. 베이커리의 가게에서.

2\. 베이커리의 웹사이트에서

3\. 쿠키 유통업체에 도매

여기서 쿠키를 파는 것은 베이커리의 책임이 아니다. 베이커리는 쿠키를 만들기만 하면 된다.

그렇다면 우리는 쿠키가 만들어지면 쿠키를 배달할 방법이 필요한데, 이 작업의 코드를 Bakery class에 작성하지 않을 것이다.

이를 위해서 delegation이 필요한 것이다!

먼저 Bakery가 위임할 작업들을 정의한 프로토콜을 생성한다. 

\* delegation에서 프로토콜은 해야 할 작업의 목록들을 정의함.

그다음 delegate 프로퍼티를 Bakery class에 선언한다.

```swift
protocol BakeryDelegate {
    func cookieWasBaked(_ cookie: Cookie)
}

class Bakery {
    var delegate: BakeryDelegate?
    
    func makeCookie() {
        var cookie = Cookie()
        cookie.size = 6
        cookie.hasCholocateChips = true
        
        delegate?.cookieWasBaked(cookie)
    }
    
}
```

위임할 작업들을 정의했으니, 이제는 위임을 받을 delegate class를 만들어 보자!

CookieShop 클래스는 BakeryDelegate 프로토콜을 채택하고, cookieWasBaked(\_:) 함수를 conform 한다.

```swift
class CookieShop: BakeryDelegate {
    func cookieWasBaked(_ cookie: Cookie) {
        print("Yay! A new cookie was baked, with size \(cookie.size)")
    }
}
```

이제 마지막으로 이 모든 것들을 활용해보자.

CookieShop , Bakery 객체를 생성해서 각각 상수 shop, bakery에 할당하고 

bakery.delegate 에 shop을 할당한다. 이로 인해 shop이 bakery의 위임자가 되었다.

마지막으로, bakery는 쿠키를 만들고, 그 쿠키는 shop으로 전달된다.

```swift
let shop = CookieShop()

let bakery = Bakery()
bakery.delegate = shop

bakery.makeCookie()
// Prints Yay! A new cookie was baked, with size 6
```

이것이 delegation이다!! delegation의 힘은 bakery는 쿠키가 **결국에 어디로 갔는지 알 필요가 없다는 것이다.** bakery는 BakeryDeleate 프로토콜을 채택하는 모든 클래스에 쿠키를 제공할 수 있다. bakery는 프로토콜의 이행에 대해 알 필요 없이 필요할 때마다 cookieWasBaked(\_:)만 호출하면 된다.

### **Delegation in Practical iOS Development**

Delegation은 iOS 개발에서 가장 흔한 디자인 패턴 중 하나이다.

Delegation을 사용하는 iOS SDK 내에 클래스를 살펴보면:

1\. _UITableView_ 클래스는 table view의 상호작용, cell의 표시, table view의 laouy 변화를 위해서 _UITableViewDelegate_, _UITableViewDataSource_ 프로토콜을  사용한다.

2\. _CLLocationManager_는 아이폰의 GPS 좌표와 같은 위치와 관련된 값을 앱에 report 하기 위해서 _CLLocationManagerDelegate를_ 사용한다.

3\. _UITextView는_ 새로운 문자 삽입, text 편집의 멈춤과 같은 text view의 변화를 report 하기 위해서 _UITextViewDelegate를_ 사용한다.

3가지 delegate 프로토콜을 보면 공통된 패턴을 보여주는데:

모든 위임되는 이벤트들은 우리의 통제에서 벗어난 class, 사용자, OS, 하드웨어에 의해 초기화된다.

더 자세히 살펴보자:

\- GPS 좌표는 GPS 칩에 의해 제공되고 칩이 GPS 위치를 고정할 때마다 우리의 코드에 위임된다.

\- text view는 임의의 사용자의 입력에 반응하고 그에 따라 적절한 delegate 함수들을 호출한다.

요약하자면 **우리가 제어할 수 없는 event와 action과의 연결**을 위해서 delegation을 사용하는 것이다.

### **An Example with UITextView**

text view의 delegate pattern을 살펴보자

UIViewController의 서브 클래스인 NoteViewController는 UITextViewDelegate 프로토콜을 채택하고 있고, 간단한 textview 프로퍼를 가지고 있다. 여기서 textview는 적절하게 초기화되었다고 가정하자. 

```swift
class NoteViewController: UIViewController, UITextViewDelegate {
    var textView:UITextView = ...
    
    func viewDidLoad()
    {
        textView.delegate = self
    }
}
```

viewDidLoad() 함수에서 textview 프로퍼티의 delegate를 selft로 할당해 주었고, 이로 인해 NoteViewController는 textView의 위임자가 되었다.

UITextViewDelegate를 채택함에 따라서, 이제 textview에서 발생한 이벤트에 반응하는 몇 가지 delegate 함수를 수행할 수 있다.

다음은 delegate 함수의 일부이다:

-   textViewDidBeginEditing(\_:)
-   textViewDidEndEditing(\_:)
-   textView(\_:shouldChangeTextIn:replacementText:)
-   textViewDidChange(\_:)

### **Why Use Delegation?**

-   Delegation은 class 간의 **상호 작용과 처리 작업을 전달하기** 쉬운 가벼운 접근 방식이고.
    
-   클래스 간의 요구사항을 전달하기 위해 **단지 프로토콜만 필요하기 때문에 클래스 간의 결합도를 크게 줄일 수 있다.**
    
-   또한, 상호작용을 일으키는 클래스의 책임과 상호작용에 반응하는 클래스를 분리할 수 있다.
    

**결론적으로**Delegation은 내가 통제하지 않은 코드 내에서 발생하는 이벤트를 hook into 하는데 좋은 방법이다.
