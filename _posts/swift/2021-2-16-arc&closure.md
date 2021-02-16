---
layout: post
title: ARC와 Closure
category: Swift
tags: [ARC, Closure]
comments: true
---

>이 글은 공부 중에 작성하는 글입니다.
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

지난번에 포스팅했던  [ARC, 강한 참조와  약한 참조](https://odong-tree.github.io/swift/2021/01/01/strong_weak/)에 이어서 ARC와 Closure에 대해서 포스팅하려고 합니다. [공식문서](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)를 토대로 작성합니다.

<br>
<br>

# Closure의 강한 참조
클로저에서 강한 참조는 인스턴스를 capture한 클로저가 인스턴스의 프로퍼티로 할당될 때 발생합니다. 예를 들어 `self.someProperty`나 `self.someMethod`처럼 인스턴스의 프로퍼티에 접근하거나 메서드를 호출할 때 강한 참조가 발생하는 것이지요. <br>

이러한 강한 참조가 발생하는 이유는 클로저가 class와 같은 **참조 타입**이기 때문입니다. 즉 참조 타입인 두 개의 클래스 사이에서 강한 참조가 발생하는 것과 같은 맥락인 것입니다. 아래의 코드로 강한 참조를 확인해보겠습니다. <br>

```swift
class Odong {
    let name: String
    let nickname: String?

    lazy var callName: () -> String = {
        if let nickname = self.nickname {
            return "Hello, Mr.\(nickname)"
        } else {
            return "Hello, Mr.\(self.name)"
        }
    }

    init(name: String, nickname: String? = nil) {
        self.name = name
        self.nickname = nickname
    }

    deinit {
        print("It is being deinitialized")
    }
}

let odong1 = Odong(name: "odongnamu")
print(odong1.callName()) // Hello, Mr.odongnamu

let odong2 = Odong(name: "odongnamu", nickname: "tree")
print(odong2.callName()) // Hello, Mr.tree
```

<br>
callName이라는 프로퍼티에 인스턴스의 프로퍼티에 접근하는 클로저를 할당하는 코드를  작성해보았습니다. <br>

```swift
var odongTest: Odong? = Odong(name: "odongnamu")
print(odongTest!.callName()) // Hello, Mr.odongnamu

odongTest = nil
// deinit이 출력되지 않는다.
```

이렇게 강한 참조가 발생하는 것을 확인할 수 있습니다. 그렇다면 클로저에서 발생하는 강한 참조는 어떻게 해결해줄 수 있을까요?

<br>

# Capture List
클로저에 **Capture List**를 정의하는 것으로 클로저-클래스의 강한 참조를 해결할 수 있습니다. 공식 문서의 예제를 빌려와보면

```swift
lazy var someClosure = {
    [unowned self, weak delegate = self.delegate]
    (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}

// 파라미터가 없을 경우
lazy var someClosure = {
    [unowned self, weak delegate = self.delegate] in
    // closure body goes here
}
```

앞서 작성했던 코드의 경우 파라미터가 없기 때문에 callName 코드 내부에 아래와 같이 Capture List를 작성해주면

```swift
lazy var callName: () -> String = {
    [unowned self] in
    if let nickname = self.nickname {
        return "Hello, Mr.\(nickname)"
    } else {
        return "Hello, Mr.\(self.name)"
    }
}
```
.
.
.


```swift
var odongTest: Odong? = Odong(name: "odongnamu")
print(odongTest!.callName()) // Hello, Mr.odongnamu

odongTest = nil
// It is being deinitialized
```


deinit 출력문이 출력되는 것을 확인할 수 있습니다.
<br>

**하지만 이때 unowned 대신에 weak를 써주니 self에 대해서 optional chaning이 필요하다고 에러가 발생했습니다. 어떻게 구분해서 써줘야하는 걸까요?**

<br>
<br>

###  ✐ unowned
unowned은 클로저와 캡처하는 인스턴스가 **항상 서로를 참조하고, 항상 동시에 할당 해제되는 경우에** unowned을 사용합니다.

<br>

###  ✐ weak
반대로 캡처된 참조가 미래의 어느 시점에 nil이 될 수 있는 경우에는 약한 참조로 정의합니다. 즉  할당 해제되는 시점이 다른 경우를 말하는 것 같습니다. 약한 참조는 항상 **optional**로  정의되어야 하며, 참조하는 인스턴스가 할당 해제되면 자동으로 nil이 됩니다.

<br>

따라서 위의 예시의 경우에는 할당 해제 시점이 동일하고, 항상 서로를 참조하기 때문에 unowned을 사용했습니다. 그리고 unowned을 weak으로 바꾸었을 때 항상 optional로 정의되어야 하므로 optional chaning이 요구되었던 것입니다.






<br>
<br>

#### 참고
- [https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)

<br>
<br>
