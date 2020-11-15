---
layout: post
title:  "메모리와 디스크에 캐시해보자(feat. NSCache)"
date:   2020-11-15 08:43:59
author: delmaSong
categories: iOS
tags:	iOS
cover:  "/assets/instacode.png"

---



최근, 회사에서 서버로부터 받아온 데이터를 캐시하라는 명을 받아 작업을 했었습니다. 이슈를 성공적으로 마무리 한 기념으로 캐시가 무엇인지, 어떻게 사용하는 지에 대해 정리해보고자 합니다.



## 캐시(cache)란?

Cache는 자주 사용하는 데이터나 값을 미리 복사해 놓는 임시 장소를 가리킵니다. 캐시에 데이터를 미리 복사해 놓으면 계산이나 접근 시간 없이 더 빠른 속도로 데이터에 접근할 수 있고 효율적인 통신을 할 수 있습니다.

보통 **반복적으로 동일한 결과를 돌려주는 작업**이나 **각 작업의 시간이 오래 걸리던지 서버에 부담을 주는 경우(외부 API 호출 / DB 데이터 호출)**에 캐시를 사용합니다.

iOS에서는 두가지 종류의 캐시를 구현할 수 있습니다. **기기를 끄면 캐시 데이터가 사라지는 메모리 캐시**와 **캐시 데이터가 기기 안에 저장되어 있어, 껐다 켜도 남아있는 디스크 캐시**가 그것입니다.

<br>

## NSCache

Cocoa는 메모리 관리 문제를 해결하는 동시에 캐시할 항목을 위한 저장 컨테이너로 `NSCache`를 제공합니다. `NSCache`는 `NSDictionary`와 유사하며 둘 다 `key-value` 쌍을 보유합니다. 하지만 `NSCache`는 "Reactive Cache(반응 캐시)"로 시스템 메모리를 너무 많이 사용하지 않도록 하는 자동 제거 정책을 가지고 있습니다. 다른 앱에서 메모리가 필요한 경우 캐시에서 일부 항목을 제거해 메모리 공간을 최소화합니다. 

캐시 데이터를 읽고 쓰고 지울때마다 따로 lock을 할 필요가 없어 thread-safe합니다. 그리고 `NSMutableDictionary`는 Key값을 copy하지만 `NSCache`는 복사하지 않고 retain 카운트만 증가시킵니다.

이러한 장점 때문에 `Dictionary`를 이용해 캐시하는 것 보다 `NSCache`를 이용하는 것이 더 편리합니다. 그러나 `NSCache`는 `Objective-C` 기반 클래스이기 때문에 `NSCache`를 이용해 캐시하는 경우, 클래스 인스턴스만 캐시할 수 있습니다. 또한 `NSObject` 기반의 키하고만 호환됩니다. 

`NSCache`를 감싼 별개의 `Cache` 커스텀 클래스를 만든다면 구조체 및 기타 유형들을 저장할 수 있으며 `Hashable` 프로토콜을 채택한 모든 키를 사용할 수 있습니다. 



아래에 `Article`이라는 구조체가 있습니다. 이 구조체를 캐시하기 위해 NSCache를 래핑한 클래스를 만들어 봅시다.

```swift
struct Article {
  var title: String
  var subtitle: String
  var writtenDate: String
  var contents: String
}
```

```swift 
final class Cache<Key: Hashable, Value> {
  private let wrapped = NSCache<WrappedKey, Entry>()
}
```



메모리 캐시



### 참고

- [Effective Objective-C #50](https://aroundck.tistory.com/4717)

- [Dictionary vs NSCache](https://medium.com/@prasanna.aithal/dictionary-v-s-nscache-in-ios-a94f0e22602e)
- [DiskCache와 MemoryCache](https://nsios.tistory.com/58)

- [캐시란](https://jojoldu.tistory.com/57)

- [Caching in Swift](https://www.swiftbysundell.com/articles/caching-in-swift/)

- [NSCache.swift](https://github.com/apple/swift-corelibs-foundation/blob/bfe81824d9b8a98b84dcc5c022c5a9d567dbc224/Foundation/NSCache.swift)

- [Apple Developer Document - NSCache](https://developer.apple.com/documentation/foundation/nscache)

- [Swift의 캐싱 NSCache](https://gaki2745.github.io/swift/2019/10/10/Swift-Basic-08/)

