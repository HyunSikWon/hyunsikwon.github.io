---
title: PhotoKit 튜토리얼!
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- PhotoKit
last_modified_at: 2021-01-06 T21:00:00+08:00
---


`Photokit`은 iCloud 사진, 라이브 포토(Live Photos)를 포함한 사진 앱에서 관리하는 모든 사진 및 비디오 에셋(asset)을 사용하여 작업할 수 있는 프레임워크이다. `PhotoKit`은 컨텐츠를 표시 및 재생하기위해 에셋을 가져(fetch)오고 캐시(cache) 할 수 있다. 뿐만 아니라 사진 및 비디오 컨텐츠를 편집할 수 있고, 앨범, Moments, Shared Albums 같은 에셋들을 관리할 수도 있다. 간단한 튜토리얼 프로젝트를 통해서 `PhotoKit`을 공부해보자.

## Getting Started

튜토리얼 프로젝트의 기본적인 레이아웃은 다음과 같이 아주 간단한 형태이다. ( `AlbumCollectionViewController`, `PhotosCollectionViewController`, `PhotoViewController` )

![noirit-storyboard-650x309](https://user-images.githubusercontent.com/48352065/103767207-919e7f00-5063-11eb-9179-2052903608a1.png)


## Getting PhotoKit Permissions

다른 iOS API와 같이 `PhotoKit`도 승인(permissions) 모델을 사용한다. 이는 사용자에게 앱이 그들의 사진에 접근한다는 승인 요청을 보여준다. 사진에 접근하고 수정하기 위해서는 반드시 사용자의 승인을 얻어야한다. `PHPhotoLibrary`를 사용하여 사진첩 접근 관리를 수행한다.

### Modifying Info.plist

먼저 `Info.plist`에 사진첩에 왜 접근해야 하는지 설명하는 키(key)를 추가한다.

![noirit-infoplist-650x274](https://user-images.githubusercontent.com/48352065/103767204-9105e880-5063-11eb-825e-f231ed4c9edb.png)

### Requesting Authorization

먼저 `AlbumCollectionViewController.swift` 파일의 `getPermissionIfNecessary(completionHandler:)` 메소드를 살펴보자

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    getPermissionIfNecessary { granted in
      guard granted else { return }
      self.fetchAssets()
      DispatchQueue.main.async {
        self.collectionView.reloadData()
      }
    }
  }

...

func getPermissionIfNecessary(completionHandler: @escaping (Bool) -> Void) {
    guard PHPhotoLibrary.authorizationStatus() != .authorized else {
      completionHandler(true)
      return
    }
    
    PHPhotoLibrary.requestAuthorization { status in
      completionHandler(status == .authorized)
    }
  }

```

1. 가장 먼저 현재 권한(authorization) 상태를 `PHPhotoLibrary`를 통해 확인한다. 이미 권한 승인이 된 상태라면, `true` 값과 함께 컴플리션 핸들러를 호출한다. 
2. 만약 이전에 요청이 없었다면, 권한에 대한 요청을 진행한다. `PHAuthorizationStatus` 객체가 권한 상태를 나타내고, 이를 통해서 컴플리션 핸들러를 호출한다. 만약 상태 값이 `.authorized` 면 `true`, 그 외에는 `false` 가 리턴된다.

> `PHAuthorizationStatus`는 열거형이고, `notDetermined`, `restricted`, `denied`, `limited` 값도 가진다.

## Understanding Assets

Photos 앱의 사진은 favorites과 위치정보 같은 메타데이터를 가진다. 그리고 Photos 앱은 사진뿐 아니라 라이브포토와 영상을 포함한다. 이 모든 것들을 `UIImage`로 관리하는 것은 불가능 하기 때문에 우리는 `PHAsset`을 사용하는 것이다.

`PHAsset`은 이미지, 라이브포토 영상을 설명하는 메타데이터이다. `PHAsset`은 값이 변하지도 않고 이미지 자체를 가지고 있지도 않지만 우리가 이미지를 얻을 수 있는데 필요한 정보들을 제공한다. `PHAsset`은 생성일, 수정일, 위치 데이터 등 많은 정보를 포함한다. 

### Asset Data Models

```swift
private var allPhotos = PHFetchResult<PHAsset>()
private var smartAlbums = PHFetchResult<PHAssetCollection>()
private var userCollections = PHFetchResult<PHAssetCollection>()
```

`PHFetchResult`는 배열이라고 생각하면 된다. `count()`, `index(of:)` 같은 메소드를 가지기도 하고, 데이터 불러오기(fetching), 캐싱을 다루기도 한다. 좀 더 기능이 좋은 에셋 혹은 컬렉션의 배열이라고 생각하자.

### Fetching Assets and Asset Collections

다음은 에셋을 불러오는(fetcing) 작업이다.

```swift
// 1
let allPhotosOptions = PHFetchOptions()
allPhotosOptions.sortDescriptors = [
  NSSortDescriptor(
    key: "creationDate",
    ascending: false)
]
// 2
allPhotos = PHAsset.fetchAssets(with: allPhotosOptions)
// 3
smartAlbums = PHAssetCollection.fetchAssetCollections(
  with: .smartAlbum,
  subtype: .albumRegular,
  options: nil)
// 4
userCollections = PHAssetCollection.fetchAssetCollections(
  with: .album,
  subtype: .albumRegular,
  options: nil)
```

1. 에셋을 불러올때, 결과에 옵션들(sorting, filtering)을 적용할 수 있다. 여기서는 생성 일을 기준으로 정렬하는 옵션을 적용했다.
2. `PHAsset`은 에셋을 불러오고 그 결과를 `PHFetchResult`로 반환하는 기능을 제공한다. 여기서는 옵션과 함께 에셋을 불러와서 그 결과를 `allPhotos`에 할당했다.
3. Photos 앱은 자동으로 스마트 앨범을 생성한다. 앨범은 에셋의 그룹이며 `PHAssetCollection` 객체로 표현된다. 여기서는 스마트 앨범 컬렉션을 불러왔고 옵션은 따로 지정하지 않았다.
4. 사용자가 생성한 앨범에 접근하는 것도 비슷하다, 다만 불러올때 타입을 `.album` 으로 지정했다.

이제 데이터를 가져왔으니 이를 이용해 UI를 업데이트 하자.

### Prepping the Collection View

```swift
override func collectionView(
  _ collectionView: UICollectionView,
  numberOfItemsInSection section: Int
) -> Int {
  switch sections[section] {
  case .all: return 1
  case .smartAlbums: return smartAlbums.count
  case .userCollections: return userCollections.count
  }
} 
```

다음은 각각의 섹션에 표시할 아이템의 개수를 반환한다. `PHFetchResult`를 배열처럼 사용할 수 있다는 것을 보여준다.

### Updating the Cell

`collectionView(_:cellForItemAt:)`에 다음을 추가하자

```swift
// 1
guard let cell = collectionView.dequeueReusableCell(
  withReuseIdentifier: AlbumCollectionViewCell.reuseIdentifier,
  for: indexPath) as? AlbumCollectionViewCell
  else {
    fatalError("Unable to dequeue AlbumCollectionViewCell")
}
// 2
var coverAsset: PHAsset?
let sectionType = sections[indexPath.section]
switch sectionType {
// 3
case .all:
  coverAsset = allPhotos.firstObject
  cell.update(title: sectionType.description, count: allPhotos.count)
// 4
case .smartAlbums, .userCollections:
  let collection = sectionType == .smartAlbums ? 
    smartAlbums[indexPath.item] : 
    userCollections[indexPath.item]
  let fetchedAssets = PHAsset.fetchAssets(in: collection, options: nil)
  coverAsset = fetchedAssets.firstObject
  cell.update(title: collection.localizedTitle, count: fetchedAssets.count)
}
// 5
guard let asset = coverAsset else { return cell }
cell.photoView.fetchImageAsset(asset, targetSize: cell.bounds.size) { success in
  cell.photoView.isHidden = !success
  cell.emptyView.isHidden = success
}
return cell 
```

1. `AlbumCollectionViewCell` 타입으로 셀을 조작한다.
2. 앨범 커버 이미지, 섹션 타입으로 사용될 변수들을 생성한다.
3. "all photos" 섹션의 커버 이미지를 `allPhotos`의 첫번째 에셋으로 설정한다. 그 후 섹션 이름과 개수를 업데이트 한다.
4. `smartAlbums` 과 `userCollections`는 모두 컬렉션 타입이므로 비슷한 방식을 사용한다. 먼저 셀과 섹션 타입을 위해 불러온 결과(fetch result)로부터 컬렉션을 가져온다. 컬렉션의 첫번째 에셋을 커버 에셋으로 사용하고, 앨범 제목과 에셋 개수와 함께 셀을 업데이트 한다.
5. 만약 커버 에셋이 없다면 셀을 그대로 반환하고, 있다면 에셋으로부터 이미지를 가져온다. 이미지를 가져오는 컴플리션 블록(클로저)에서 반환된 성공 상태(success state)를 사용하여 셀의 `photoView` 와 `emptyView`를 설정한다.

빌드하고 실행해보자. 적절한 레이아웃은 구성되었으나 커버 이미지가 생성되지 않았다. 제대로 고쳐보자.

<img src="https://user-images.githubusercontent.com/48352065/103767202-8fd4bb80-5063-11eb-903a-bad84bdcd2a9.png" width="40%">

## Fetching Images from Assets

위에서 이미지를 불러오기 위해 `fetchImageAsset(_:targetSize:contentMode:options:completionHandler:)`를 사용했다. 이는 `UIImage`의 `extension`에 추가한 커스텀 메소드이며 아직 이미지를 가져오는 로직이 정의되어있지 않다. `PHImageManager`를 사용하여 로직을 작성해보자. 이미지 매니저는 에셋으로부터 이미지를 불러오고(fetch) 추후에 빠르게 복구하기 위해 결과를 캐싱하는 작업을 다룬다.

```swift
// 1
guard let asset = asset else {
  completionHandler?(false)
  return
}
// 2
let resultHandler: (UIImage?, [AnyHashable: Any]?) -> Void = { image, info in
  self.image = image
  completionHandler?(true)
}
// 3
PHImageManager.default().requestImage(
  for: asset,
  targetSize: size,
  contentMode: contentMode,
  options: options,
  resultHandler: resultHandler)
```

1. 만약 `asset`이 `nil`이라면 `false`를 리턴하고 아니면 계속 진행한다.
2. 결과 핸들러(result handler)를 생성한다. 이미지 매니저는 이미지 요청이 성공하면 이 핸들러를 호출할 것이다. 반환된 이미지를 `UIImageView`의 `image`프로퍼티에 할당한다. `true` 값과 함께 컴플리션 핸들러를 호출한다. 이는 요청이 성공적임을 나타낸다.
3. 마지막으로 이미지 매니저로부터 이미지를 요청한다. 에셋, 크기, 컨텐츠 모드, 옵션, 결과 핸들러를 제공한다. `resultHandler`를 제외한 모든 것은 호출하는 코드에서 제공된다.

이제 커버 이미지가 나타난다!

<img src="https://user-images.githubusercontent.com/48352065/103767193-8d726180-5063-11eb-8943-8599a6728403.png" width="40%">
 

## Displaying Album Assets

앨범 내의 이미지를 모두 표시하기위한 세그웨이 작업을 다음과 같이 수행해보자.`AlbumCollectionViewController.swift`의 `makePhotosCollectionViewController(_:)` 에 다음 코드를 추가한다.

```swift
@IBSegueAction func makePhotosCollectionViewController(_ coder: NSCoder) -> PhotosCollectionViewController? {
    // 1
    guard
      let selectedIndexPath = collectionView.indexPathsForSelectedItems?.first
    else { return nil }
    
    // 2
    let sectionType = sections[selectedIndexPath.section]
    let item = selectedIndexPath.item
    
    // 3
    let assets: PHFetchResult<PHAsset>
    let title: String
    
    switch sectionType {
    // 4
    case .all:
      assets = allPhotos
      title = AlbumCollectionSectionType.all.description
    // 5
    case .smartAlbums, .userCollections:
      let album =
        sectionType == .smartAlbums ? smartAlbums[item] : userCollections[item]
      assets = PHAsset.fetchAssets(in: album, options: nil)
      title = album.localizedTitle ?? ""
    }
    
    // 6
    return PhotosCollectionViewController(assets: assets, title: title, coder: coder)
    
  }
```

1. 선택된 셀의 index path를 가져온다
2. 섹션 타입과 선택된 아이템을 위한 `item`을 생성한다.
3. `PhotosCollectionViewController`는 에셋 리스트와 제목이 필요하다.
4. 만약 사용자가 "all photos" 섹션을 선택한다면, `allPhotos`를 사용하고 제목을 지정해주면 된다.
5. 만약 사용자가 앨범 혹은 사용자 컬렉션을 선택하면, 선택된 앨범을 얻기 위해 섹션과 아이템을 사용하고, 앨범의 모든 에셋을 불러오면(fetch) 된다. `PHAssetCollection` 타입이 `PHAssetReuslt<PHAsset>`이 되는 것이다.
6. 이동할 뷰 컨트롤러를 생성하여 리턴해주자.

결과!



<img src="https://user-images.githubusercontent.com/48352065/103767187-8a777100-5063-11eb-8330-8c618b8606e8.png" width="40%">  <img src="https://user-images.githubusercontent.com/48352065/103767174-83e8f980-5063-11eb-9a35-03d7937a88fa.png" width="40%">



## Reference

[Raywenderich](https://www.raywenderlich.com/11764166-getting-started-with-photokit)
