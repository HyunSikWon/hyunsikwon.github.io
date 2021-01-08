---
title: PhotoKit 튜토리얼!
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- PhotoKit
last_modified_at: 2021-01-08 T16:00:00+08:00
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


## Modifying Asset Metadata

### Change Requests

`PHAssetChangeRequest`는 에셋의 생성, 수정, 삭제를 도와준다. `toggleFavorite()`에 다음의 코드를 추가한다.

```swift
// 1
let changeHandler: () -> Void = {
  let request = PHAssetChangeRequest(for: self.asset)
  request.isFavorite = !self.asset.isFavorite
}
// 2
PHPhotoLibrary.shared().performChanges(changeHandler, completionHandler: nil)
```

1. 변경 사항을 캡슐화하기 위해서 코드 블럭을 생성한다. 먼저 에셋에 대한 변경 요청(`request`)을 생성하고, 요청의 `isFavorite` 프로퍼티를 현재 값의 반대로 설정한다.
2. 변경 요청 블럭을 전달해서 사진 라이브러리가 변경 사항을 수행하도록 지시한다. 여기서 컴플리션 핸들러는 필요하지 않다.

UI를 위한 코드도 추가한다.

```swift
if asset.isFavorite {
  favoriteButton.image = UIImage(systemName: "heart.fill")
} else {
  favoriteButton.image = UIImage(systemName: "heart")
}
```

### Photo View Controller Change Observer

PhotoKit은 더 좋은 퍼포먼스를 위해서 불러오기(fetch) 요청의 결과를 저장한다(caches). 사진의 하트 버튼(favorite)을 눌렀을때, 라이브러리의 에셋은 업데이트 되지만, 뷰 컨트롤러에 있는 에셋의 복사본은 업데이트되지 않는다. 따라서 컨트롤러는 라이브러리의 업데이트 사항에따라서 필요에 따라 자신의 에셋을 업데이트 할 필요가 있다. 컨트롤러가 `PHPhotoLibraryChangeObserver`를 준수(conform)하면 이를 수행할 수 있다.

```swift
extension PhotoViewController: PHPhotoLibraryChangeObserver {
  func photoLibraryDidChange(_ changeInstance: PHChange) {
    // 2
    guard
      let change = changeInstance.changeDetails(for: asset),
      let updatedAsset = change.objectAfterChanges
      else { return }
    // 3
    DispatchQueue.main.sync {
      // 4
      asset = updatedAsset
      imageView.fetchImageAsset(
        asset, 
        targetSize: view.bounds.size
      ) { [weak self] _ in
        guard let self = self else { return }
        // 5
        self.updateFavoriteButton()
        self.updateUndoButton()
      }
    }
  }
}
```

1. 변경 옵저버는 오직 하나의 메소드만 가진다. 라이브러리가 변할 때마다 이 메소드를 호출한다. `photoLibraryDidChange(:)`
2. 변경 사항이 우리의 에셋에 영향을 주었는지 먼저 확인한다. 라이브러리 변화를 설명하는(describe) `changeInstance` 프로퍼티를 사용한다. 이 프로퍼티의 `changeDetails(for:)`를 호출하고 우리의 에셋을 전달한다. 만약 에셋이 변경 사항에 영향을 받지 않았다면 `nil`을 반환하고, 영향을 받았다면 갱신된 버전의 에셋을 `objectAfterChanges`를 호출하여 받을 수 있다.
3. 이 메소드는 백그라운드에서 실행되기 때문에, UI를 업데이트하는 나머지 로직들은 메인 스레드에서 수행해야 한다.
4. 컨트롤러의 에셋 프로퍼티를 업데이트 된 에셋으로 바꾸고, 새로운 이미지를 불러온다.
5. UI를 갱신(refresh) 한다.

### Registering the Photo View Controller

옵저버를 등록(register)해보자. `viewDidLoad()`에 다음을 추가한다. 업데이트 사항을 받기 위해선 반드시 등록해야 한다.

```swift
PHPhotoLibrary.shared().register(self)
```

또한 작업이 모두 끝나면 등록 해제(unregister)도 해줘야 한다.

```swift
deinit {
  PHPhotoLibrary.shared().unregisterChangeObserver(self)
}
```

결과 화면

<img src="https://user-images.githubusercontent.com/48352065/103984593-bf520800-51ca-11eb-8ce0-a1a9a3283e0d.png" width="40%"> 

여기에 한 가지 문제가 있다. 하트 버튼을 눌러서 선호하는 사진을 등록하고, "All Photos"로 돌아간 뒤, 다시 같은 사진을 선택하면, 하트는 빈 하트가 되어있고, 다시 하트를 눌러도 하트가 채워지지 않는다. 

### Photos View Controller Change Observer

이 문제는 이전 화면(`PhotosCollectionViewController`)가 변경 사항에 대한 관찰(oberve)을 하지 않기 때문에 발생한다. 따라서 `PhotosCollectionViewController`도 `PHPhotoLibraryChangeObserver`를 준수하도록 해야한다. 

```swift
extension PhotosCollectionViewController: PHPhotoLibraryChangeObserver {
  func photoLibraryDidChange(_ changeInstance: PHChange) {
    // 1
    guard let change = changeInstance.changeDetails(for: assets) else {
      return
    }
    DispatchQueue.main.sync {
      // 2
      assets = change.fetchResultAfterChanges
      collectionView.reloadData()
    }
  }
} 
```

`PhotoViewController` 에서 한 것과 비슷하다.

1. 이 뷰(`PhotosCollectionViewController`)는 여러 에셋을 표시하므로, 이 에셋들에대한 모든 변경사항을 확인한다.
2. 갱신된 결과를 `assets`에 할당하고, 컬렉션 뷰를 리프레시 한다.

### Registering the Photos View Controller

`viewDidLoad()`에 다음 코드를 추가한다.

```swift
PHPhotoLibrary.shared().register(self)
```

등록 해제도 추가.

```swift
deinit {
  PHPhotoLibrary.shared().unregisterChangeObserver(self)
}

```

### Album View Controller Change Observer

`AlbumCollectionViewController`에도 같은 작업을 추가한다.

```swift
extension AlbumCollectionViewController: PHPhotoLibraryChangeObserver {
  func photoLibraryDidChange(_ changeInstance: PHChange) {
    DispatchQueue.main.sync {
      // 1
      if let changeDetails = changeInstance.changeDetails(for: allPhotos) {
        allPhotos = changeDetails.fetchResultAfterChanges
      }
      // 2
      if let changeDetails = changeInstance.changeDetails(for: smartAlbums) {
        smartAlbums = changeDetails.fetchResultAfterChanges
      }
      if let changeDetails = changeInstance.changeDetails(for: userCollections) {
        userCollections = changeDetails.fetchResultAfterChanges
      }
      // 4
      collectionView.reloadData()
    }
  }
}
```

여기서는 약간의 차이가 있다. 다양한 fetch result가 존재하기 때문에 모든 경우에 변경 사항을 확인해야 한다.

### Album View Controller Registration

역시 같은 작업.

```swift
// viewDidLoad()에 추가
PHPhotoLibrary.shared().register(self)

deinit {
  PHPhotoLibrary.shared().unregisterChangeObserver(self)
}
```

## Editing a Photo

사진을 편집하는 기능을 추가해보자. 먼저 수정한 에셋을 저장할 컨테이너인 `PHContentEditingOutput`를 선언한다.

```swift
private var editingOutput: PHContentEditingOutput?
```

이후에는 `applyFilter()` 메소드에 다음 로직을 추가한다.

```swift
// 1
asset.requestContentEditingInput(with: nil) { [weak self] input, _ in
  guard let self = self else { return }

  // 2
  guard let bundleID = Bundle.main.bundleIdentifier else {
    fatalError("Error: unable to get bundle identifier")
  }
  guard let input = input else {
    fatalError("Error: cannot get editing input")
  }
  guard let filterData = Filter.noir.data else {
    fatalError("Error: cannot get filter data")
  }
  // 3
  let adjustmentData = PHAdjustmentData(
    formatIdentifier: bundleID,
    formatVersion: "1.0",
    data: filterData)
  // 4
  self.editingOutput = PHContentEditingOutput(contentEditingInput: input)
  guard let editingOutput = self.editingOutput else { return }
  editingOutput.adjustmentData = adjustmentData
  // 5
  let fitleredImage = self.imageView.image?.applyFilter(.noir)
  self.imageView.image = fitleredImage
  // 6
  let jpegData = fitleredImage?.jpegData(compressionQuality: 1.0)
  do {
    try jpegData?.write(to: editingOutput.renderedContentURL)
  } catch {
    print(error.localizedDescription)
  }
  // 7
  DispatchQueue.main.async {
    self.saveButton.isEnabled = true
  }
}
```

1. 편집은 컨테이너 안에서 완료된다. 입력(input) 컨테이너는 이미지에 접근할 수 있도록 해준다. 편집 로직은 컴플리션 핸들러 안에 위치한다.
2. 계속 진행하기 위해서는 번들 식별자, 컴플리션 핸들러의 입력 컨테이너 그리고 필터 데이터가 필요하다.
3. 수정(adjustment) 데이터는 에셋에게 변경사항을 설명하는 방법이다. 이 데이터를 생성하기 위해서는 변경사항을 식별한 고유의 식별자를 사용한다. 번들 아이디를 사용하는 것이 좋다. 또한 버전 넘버와 이미지를 수정하는데 사용되는 데이터도 사용한다.
4. 또한 최종 수정된 이미지를 위한 출력(output) 컨테이너도 필요하다. 입력 컨테이너를 전달하여 이를 생성한다.
5. 이미지에 필터를 적용한다. 동작 방식은 튜토리얼의 주 내용이 아니지만 `UIImage+Extensions.swift`에서 관련 코드를 확인할 수 있다.
6. 이미지를 위해서 JPEG 데이터를 생성하고, 출력 데이터에 저장(write) 한다.
7. 마지막으로 저장 버튼을 활성화 한다.

 
<img src="https://user-images.githubusercontent.com/48352065/103984587-bd884480-51ca-11eb-96f2-669a77600454.png" width="40%"> 

## Saving Edits

이제 활성화된 저장 버튼에 기능을 추가하자. 변경사항을 라이브러리에 저장하기 위해서 위에서 사용한 출력 컨테이너를 사용한다. 위에서 메타 데이터를 변경했던 것처럼 `PHAssetChangeRequest`를 사용한다.

`saveImage()`에 로직을 추가하자.

```swift
// 1
let changeRequest: () -> Void = { [weak self] in
  guard let self = self else { return }
  let changeRequest = PHAssetChangeRequest(for: self.asset)
  changeRequest.contentEditingOutput = self.editingOutput
}
// 2
let completionHandler: (Bool, Error?) -> Void = { [weak self] success, error in
  guard let self = self else { return }

  guard success else {
    print("Error: cannot edit asset: \(String(describing: error))")
    return
  }
  // 3
  self.editingOutput = nil
  DispatchQueue.main.async {
    self.saveButton.isEnabled = false
  }
}
// 4
PHPhotoLibrary.shared().performChanges(
  changeRequest,
  completionHandler: completionHandler)
```

1. 변경 사항은 코드 블럭안에서 다룬다. 에셋을 위한 `PHAssetChangeRequest`를 생성하고 출력 컨테이너를 적용한다(apply).
2. 컴플리션 핸들러를 생성한다. 이는 변경이 완료되면 수행된다. 결과의 성공 여부를 확인하고 실패하면 오류를 출력한다.
3. 만약 변경이 성공적이라면 더이상 필요 없는 컨테이너에 `nil`을 할당한다. 또한, 저장할 이미지가 더이상 존재하지 않으므로 저장 버튼을 비활성화 시킨다.
4. 라이브러리의 `performChanges(:completionHandler:)`를 호출한다.
5. 

저장하기를 누르면, iOS는 사진을 수정해도 되는지 물어보는 다이얼로그 박스를 표시할 것이다.

<img src="https://user-images.githubusercontent.com/48352065/103984581-bb25ea80-51ca-11eb-89c4-f561ababd043.png" width="40%"> 


## Undoing Edits

마지막으로 되돌리기(undo) 버튼의 기능을 추가하자. 이 기능 역시 `PHAssetChangeRequest`를 사용한다. 변경된 데이터의 존재 여부에 따라서 되돌리기 버튼의 활성화를 결정한다.

`updateUndoButton()`에 다음을 추가한다.

```swift
let adjustmentResources = PHAssetResource.assetResources(for: asset)
  .filter { $0.type == .adjustmentData }
undoButton.isEnabled = !adjustmentResources.isEmpty 
```

한 에셋에 대한 각각의 수정사항은 `PHAssetResource` 객체를 생성한다. 그리고 `assetResources(for:)`는 주어진 에셋에 대한 리소스 배열을 반환한다. 필터를 통해서 수정된 데이터를 걸러낸다. 수정 데이터에 존재 여부에 따라서 되돌리기 버튼의 활성화가 결정된다.

이제 되돌리기 기능을 위한 로직을 `undo()`에 추가한다.

```swift
// 1
let changeRequest: () -> Void = { [weak self] in
  guard let self = self else { return }
  let request = PHAssetChangeRequest(for: self.asset)
  request.revertAssetContentToOriginal()
}
// 2
let completionHandler: (Bool, Error?) -> Void = { [weak self] success, error in
  guard let self = self else { return }

  guard success else {
    print("Error: can't revert the asset: \(String(describing: error))")
    return
  }
  DispatchQueue.main.async {
    self.undoButton.isEnabled = false
  }
}
// 3
PHPhotoLibrary.shared().performChanges(
  changeRequest,
  completionHandler: completionHandler)
```

1. 변경 로직을 가지는 변경 요청 블록을 생성한다. 여기서는 `revertAssetContentToOriginal()` 를 호출하고, 이는 기본 상태로 다시 되돌리는 요청이다. 메타 데이터에는 영향을 주지 않는다.
2. 컴플리션 핸들러는 성공 여부를 검사한 뒤, 성공하면 되돌리기 버튼을 비활성화 한다.
3. 마지막으로 라이브러리가 변경사항을 수행하도록 지시한다.

되돌리기 버튼을 누르면 위에서 본 것처럼 iOS가 모든 변경사항을 되돌릴지 사용자에게 물어본다.

<img src="https://user-images.githubusercontent.com/48352065/103984564-b5c8a000-51ca-11eb-94f2-2d445f520be4.png" width="40%"> 


## Reference

[Raywenderich](https://www.raywenderlich.com/11764166-getting-started-with-photokit)
