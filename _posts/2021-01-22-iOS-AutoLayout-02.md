---
title: Intrinsic Content Size를 알아보자 (Content Hugging & Compression Resistance)
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- AutoLayout
last_modified_at: 2021-01-22 T22:13:00+08:00
---

## Intrinsic Content Size

몇몇 뷰는 자신의 컨텐츠에 따라 고유의 크기를 가진다. 우리는 이를 뷰의 컨텐츠 본질적인 사이즈(Intrinsic content size)라고 부른다. 예를들어 버튼은 타이틀의 크기 + 약간의 마진(margin)을 Intrinsic content size로 가진다. 그렇다고 해서 모든 뷰가 Intrinsic content size를 가지는 것은 아니다. 

Intrinsic content size를 가지는 뷰는 Intrinsic content size가 뷰의 높이, 너비, 혹은 둘 모두를 정의할 수 있다. 몇가지 예를 살펴보자.

- `UIView`, `NSView`는 Intrinsic content size를 가지지 않는다.
- `UISlider` - 너비만 Intrinsic content size로 정의한다.
- `UILabel`, `UIButton`, `UISwitch`, `UITextField` - 높이와 너비 모두 Intrinsic content size로 정의한다.
- `UITextView`, `UIImageView` - Intrinsic content size가 상황에 따라 다르다.(vary)

Intrinsic content size는 뷰의 현재 컨텐츠에 기반한다. 레이블과 버튼의 Intrinsic content size는 텍스트의 양과 폰트에 따라 결정된다. 다른 뷰의 경우, Intrinsic content sizes는 더 복잡하다. 예를 들어, 빈(empty) 이미지 뷰는 Intrinsic content size를 가지지 않다가 이미지를 추가하면 Intrinsic content size가 이미지의 크기로 설정된다.  

텍스트 뷰의 Intrinsic content size는 컨텐츠, 스크롤 가능 여부, 뷰에 적용된 다른 제약조건에 따라 바뀐다. 예를 들어 스크롤이 가능하다면, 뷰는 Intrinsic content size를 가지지 않는다. 반대의 경우 뷰의 Intrinsic content size는 자동 줄바꿈(line wrapping)을 고려하지 않은 텍스트의 크기로 결정된다. 만약 뷰의 너비 제약조건을 추가한다면, Intrinsic content size는 주어진 너비를 고려한 높이를 정의한다.

## Content Hugging & Compression Resistance

AutoLayout은 각 차원(dimension)에 대해 한 쌍의 제약 조건을 사용하여 Intrinsic content size를 정의한다. Content Hugging은 뷰를 안쪽 방향으로(inward) 당겨서 뷰가 컨텐츠를 잘 감쌀 수 있도록 하고, Compression Resistance는 뷰를 바깥 방향으로(outward) 밀어서 뷰가 컨텐츠를 침범하지(clip) 않도록 한다.

![intrinsic_content_size_2x](https://user-images.githubusercontent.com/48352065/105494824-b4d15b80-5cfe-11eb-8eb9-209920ffc36f.png)

이 제약조건들은 다음과 같은 부등식을 사용하여 정의된다. `IntrinsicHeight`과 `IntrinsicWidth` 상수는 뷰의 Intrinsic content size의 높이와 너비를 나타낸다.

```swift
// Compression Resistance
View.height >= 0.0 * NotAnAttribute + IntrinsicHeight
View.width >= 0.0 * NotAnAttribute + IntrinsicWidth
 
// Content Hugging
View.height <= 0.0 * NotAnAttribute + IntrinsicHeight
View.width <= 0.0 * NotAnAttribute + IntrinsicWidth
```

이 제약 각각은 우선순위를 가질 수 있는데, 기본적으로 뷰는 Content Hugging에 대해 250의 우선순위 그리고 Compression Resistance에 대해 750의 우선순위를 가진다.이는 뷰가 오그라드는(shrink) 것보다 늘어나는 것이 낫기 때문이다.

뷰의 Intrinsic content size 사용하면 뷰의 컨텐츠에 맞게 레이아웃을 적용할 수 있다.또한 AutoLayout을 위한 제약의 수를 줄일 수도 있다. 하지만 이를 위해서는 Content Hugging과 Compression Resistance(CHCR) 우선순위 값을 신경써 주어야한다. Intrinsic content size를 다루기 위한 몇 가지 가이드라인을 살펴보자.

- 여러 뷰를 이용해서 공간을 채울때 모든 뷰가 동일한 content-hugging priority를 가지면 안된다. 이런 상황에선 AutoLayout은 어떤 뷰가 늘어나야 하는지 알 수 없다.

    예를 들어 레이블과 텍스트 필드 쌍을 생각해보자. 레이블은 자신의 intrinsic content size를 유지하고 남은 공간은 모두 텍스트 필드가 채우기를 원하는 것이 일반적일 것이다. 이 경우 텍스트 필드의 content-hugging priority가 더 낮아야 한다.

- 뷰가 의도치 않게 확장되어서 레이아웃이 깨지는 경우가 종종 있다. 이런 경우 content-hugging priority를 높여주면 된다.
- 몇몇 뷰는 항상 자신의 Intrinsic content size를 유지해야 한다. CHCR 우선순위를 높여줌으로써 이를 크기가 확장되거나 줄어드는 것을 막을 수 있다.

## Reference

[Apple developer](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/AnatomyofaConstraint.html#//apple_ref/doc/uid/TP40010853-CH9-SW21)
