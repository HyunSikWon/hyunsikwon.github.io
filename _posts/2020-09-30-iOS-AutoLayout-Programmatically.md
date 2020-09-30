---
title: AutoLayout - Programmatically Creating Constraints
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- AutoLayout
last_modified_at: 2020-09-30 T15:00:00+08:00

---


AutoLayout을 코드로 작성하는 방법을 공부하며  [Apple 공식 문서](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/ProgrammaticallyCreatingConstraints.html#//apple_ref/doc/uid/TP40010853-CH16-SW1)를 번역하고 정리해봤습니다. 

잘못된 번역 및 표현 등은 자유롭게 피드백해주세요.

### **Programmatically Creating Constraints**

제약 조건을 코드로 작성하기 위한 방법은 총 3가지가 존재한다: layout anchor을 사용하거나, NSLayoutContraint 클래스를 사용하거나, Visual Format Language를 사용하거나.

또한 코드로 제약조건을 주기 위해선 반드시

**translatesAutoresizingMaskIntoConstraints의 값을 false로 지정해주어야 한다.  **

### **LayoutAnchor**

`NSLayoutAnchor` 를 사용하기 위해서는 제약을 주고싶은 아이템의 anchor 프로퍼티에 접근해야 한다. 예를들어, view controller의 top과 bottom layout guides는 topAnchor, bottomAnchor과 heightAnchor 프로퍼티가 있다. 반면에 View는 edge, center, size, baseline anchor을 노출시킨다.

Layout Anchor는 가독성이 좋고 컴팩트한 형식으로 제약조건을 생성한다. 아래 코드처럼 각기 다른 제약조건들을 생성하는 다양한 메소드들이 존재한다.

```swift
// Get the superview's layout
let margins = view.layoutMarginsGuide
 
// Pin the leading edge of myView to the margin's leading edge
myView.leadingAnchor.constraint(equalTo: margins.leadingAnchor).isActive = true
 
// Pin the trailing edge of myView to the margin's trailing edge
myView.trailingAnchor.constraint(equalTo: margins.trailingAnchor).isActive = true
 
// Give myView a 1:2 aspect ratio
myView.heightAnchor.constraint(equalTo: myView.widthAnchor, multiplier: 2.0).isActive = true
```

![autulayout1](https://user-images.githubusercontent.com/48352065/94648912-89c99300-032e-11eb-85a2-8d8ec24b5878.png)

Layout Anchor는 제약조건들을 생성하는 다양한 메소드들을 가지고, 각각의 메소드들은 **결과에 영향을 주는 방정식의 요소들만** parameter로 전달된다. 

```swift
myView.leadingAnchor.constraint(equalTo: margins.leadingAnchor).isActive = true
```

위 코드와 방정식을 대응하면 다음과 같다:

<table style="border-collapse: collapse; width: 77.4419%; height: 517px;" border="1" data-ke-style="style13"><tbody><tr style="height: 16px;"><td style="height: 16px; width: 49.8837%;">Equation</td><td style="height: 16px; width: 50%;">Symbol</td></tr><tr style="height: 52px;"><td style="height: 52px; width: 49.8837%;"><p>Item 1</p></td><td style="height: 52px; width: 50%;"><p>myView</p></td></tr><tr style="height: 52px;"><td style="height: 52px; width: 49.8837%;"><p>Attribute 1</p></td><td style="height: 52px; width: 50%;"><p>leadingAnchor</p></td></tr><tr style="height: 52px;"><td style="height: 52px; width: 49.8837%;"><p>Relationship</p></td><td style="height: 52px; width: 50%;"><p>constraintEqualToAnchor</p></td></tr><tr style="height: 52px;"><td style="height: 52px; width: 49.8837%;"><p>Multiplier</p></td><td style="height: 52px; width: 50%;"><p>None (defaults to 1.0)</p></td></tr><tr style="height: 52px;"><td style="height: 52px; width: 49.8837%;"><p>Item 2</p></td><td style="height: 52px; width: 50%;"><p>margins</p></td></tr><tr style="height: 52px;"><td style="height: 52px; width: 49.8837%;"><p>Attribute 2</p></td><td style="height: 52px; width: 50%;"><p>leadingAnchor</p></td></tr><tr style="height: 52px;"><td style="height: 52px; width: 49.8837%;"><p>Constant</p></td><td style="height: 52px; width: 50%;"><p>None (defaults to 0.0)</p></td></tr></tbody></table>

또한, Layout Anchor는 추가적인 type safety를 제공하는데 이로인해 잘못된 제약조건이 우연적으로 발생하는 것을 예방한다.

예를들어 horizontal anchor는 오직 horizontal과 대응되며, multiplier는 size 제약조건만을 위해 제공된다.

### **NSLayoutConstraint Class**

제약조건을 생성하기 위해서 `NSLayoutConstraint` 클래스의 `constraintWithItem:attribute:relatedBy:toItem:attribute:multiplier:constant:` 메소드를 사용할 수 있다. 이 메소드는 제약 방정식을 명시적으로 코드로 변환해준다. 각 파라미터는 방정식과 대응된다. Layout Anchor와 모든 결과에 영항을 주지 않는 파라미터도 값을 명시해야 한다. 따라서 코드의 가독성이 떨어지는 특징이 있다. 또 특정 제약조건의 중요한 특성들을 강조하지 않기 때문에 중요한 디테일을 놓칠 가능성이 있다. 따라서 Layout Anchor API를 사용하는 것을 추천한다.

```swift
NSLayoutConstraint(item: myView, 
                   attribute: .leading, 
                   relatedBy: .equal, 
                   toItem: view, 
                   attribute: .leadingMargin,
                   multiplier: 1.0, 
                   constant: 0.0).isActive = true
 
NSLayoutConstraint(item: myView, 
                   attribute: .trailing, 
                   relatedBy: .equal, 
                   toItem: view, 
                   attribute: .trailingMargin, 
                   multiplier: 1.0, 
                   constant: 0.0).isActive = true
 
NSLayoutConstraint(item: myView, 
                   attribute: .height, 
                   relatedBy: .equal, 
                   toItem: myView, 
                   attribute:.width, 
                   multiplier: 2.0, 
                   constant:0.0).isActive = true
```

### **Visual Format Language**

이 방법은 문자열과 ASCII-art 을 사용하여 제약조건을 정의하는 방법이다. 제약조건의 표현을 시간적인 묘사로 표현한다.

다음은 Visual Format Language의 장점과 단점이다:

\- AutoLayout은 Visual Format Language를 이용하여 제약조건들을 콘솔에 출력한다. 이러한 이유로 디버깅 메시지가 제약조건을 생성하기 위한 코드와 매우 유사하다.

\- Visual Format Language는 여러 제약조건들을 한번에 생성하고 매우 컴팩트한 표현을 사용한다.

\- Visual Format Language는 오직 유요한 제약조건들만 생성할 수 있게 한다.

\- 이 표현 방법은 완전성보다 좋은 시각화에 초점을 두기 때문에 Visual Format Language가 표현할 수 없는 제약조건들이 존재한다.

\- 컴파일러는 문자의 유효성을 검사하지 않기 때문에 오류는 런타임 테스팅에서만 발견된다.

Visual Format Language을 이용한 제약조건 생성 코드.

이 코드는 leading과 trailing 제약을 생성하고, visual format language는 default sapcing을 사용하면 슈퍼뷰의 margin에 0-point 제약을 생성하기 때문에 앞서 보았던 예시들과 동일한 제약조건을 생성한다.

```swift
let views = ["myView" : myView]
let formatString = "|-[myView]-|"
 
let constraints = NSLayoutConstraint.constraints(withVisualFormat: formatString, 
                                                 options: .alignAllTop, 
                                                 metrics: nil, 
                                                 views: views)

NSLayoutConstraint.activate(constraints)
```

그러나 위 코드에서 가로 세로 비율 제약조건을 생성할 수 없는데, 슈퍼 뷰를 제외한 뷰가 두개 이상 존재해야 수직, 수평 제약조건을 줄 수 있다. 이러한 이유로 `.alignAllTop`은 layout에 영항을 주지 않는다.

Visual Format Language를 사용해서 제약조건을 생성하려면:

1\. view들로 이루어진 dictionary를 생성한다. 이 dictionary는 반드시 `[String:view object]` 형태로 구성되어야 한다. key값은 view에 대한  format string을 식별한다.

2\. (Optional) metrics dictionary를 생성한다. 이 dictionary는 반드시 `[String: NSNumber]` 형태로 구성되어야 한다. key는 format string의 constant 값을 대표한다.

3\. item의 단일 행과 열을 배치함으로써  format string을 생성한다.

4\. `NSLayoutContraint` 클래스의 `constraintsWithVisualFormat:options:metrics:views:` 메소드를 호출한다.이 메소드는 모든 제약조건을 담고있는 배열을 반환한다.

5\. `NSLayoutContraint` 클래스의 `activateConstraint:` 메소드를 호출해서 모든 제약조건은 활성화 한다.
