---
title: Child View Controllers
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- View Controller
last_modified_at: 2020-09-30 T19:00:00+08:00

---

`UIView` 를 다른 `UIView` 에 추가하는 것처럼 view controller 역시 다른 view controller의 *child* 가 될 수 있다.  Child view controller는 앱의 window 크기에 따라 자동적으로 자신의 크기를 조절하지만, child view controller의 view는 frame과 autolayout을 이용해서 사이즈와 위치가 재조정 할 수 있다.

View controller를 child로 추가하기 위해선 다음과 같은 과정을 거친다.

```swift
let parent = UIViewController()
let child = UIViewController()

// 먼저, child의 view를 parent의 view에 추가한다.
parent.view.addSubview(child.view)

// 그리고, chlid를 parent에 추가한다.
parent.addChild(child)

// 마지막으로, child에게 parent로 이동했다는 것을 알린다.
child.didMove(toParent: parent)
```
<br>

추가된 child view controller를 parent로부터 삭제하기 위한 코드:

```swift
// 먼저, child에게 제거되었다고 알린다.
child.willMove(toParent: nil)

// 그리고, parent로 부터 child를 지운다. 
child.removeFromParent()

// 마지막으로, parent의 view에서 child의 view를 제거한다.
child.view.removeFromSuperview()
```
<br>

이 과정을 사용할 때마다 거치는 것은 번거롭기 때문에 extension을 활용한다.

```swift
extension UIViewController {
    func add(_ child: UIViewController) {
        addChild(child)
        view.addSubview(child.view)
        child.didMove(toParent: self)
    }

    func remove() {
        // Just to be safe, we check that this view controller
        // is actually added to a parent before removing it.
        guard parent != nil else {
            return
        }

        willMove(toParent: nil)
        view.removeFromSuperview()
        removeFromParent()
    }
}
```
<br>

Child view controller는 프로젝트 진행 과정에서 재사용이 가능한 UI를 구성하는데 유용하다. 예를 들어, 로딩 화면을 보여주고 싶을 때, child view controller를 사용할 수 있다.

loading spinner가 view 가운데에 있는 `LoadingViewController` 를 생성하고,

```swift
class LoadingViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        let spinner = UIActivityIndicatorView(style: .gray)
        spinner.translatesAutoresizingMaskIntoConstraints = false
        spinner.startAnimating()
        view.addSubview(spinner)

        // Center our spinner both horizontally & vertically
        NSLayoutConstraint.activate([
            spinner.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            spinner.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
}
```
<br>

content을 불러오는 view controller에 `LoadingViewController` 를 child view controller로 추가하여 loading spinner를 쉽게 추가하고 제거할 수 있다.

```swift
class ContentViewController: UIViewController {
    private let loader = ContentLoader()

    func loadContent() {
        let loadingVC = LoadingViewController()
        add(loadingVC)

        loader.load { [weak self] content in
            loadingVC.remove()
            self?.render(content)
        }
    }
}
```
<br>

그렇다면 왜 `UIView` 를 사용하지 않고 view controller를 사용할까? 여기 몇가지 이유가 있다:

- View controller는 다양한 UI 관련 코드를 작성할 수 있는  `viewDidLoad`,  `viewWillAppear` 를 통해 다양한 이벤트를 처리할 수 있는데,  child view controller 역시 가능하다.
- View controller가 더 self-contained 하다. View controller는 UI와 UI를 위해 필요한 로직을 모두 포함할 수 있다.
- View controller가 child로 추가되면 자동적으로 화면을 채우기 때문에,  full screen을 위한 추가적인 layout 코드를 작성할 필요가 줄어든다.
- UI의 일부를 view controller로 실행하면 다양한 상황에서 사용할 수 있다. - navigation controller에 push 될 수도 있고, 위에서 본 것처럼 child로 추가될 수 있다.
<br>

## Reference
[Swift by Sundell](https://www.swiftbysundell.com/basics/child-view-controllers/)
