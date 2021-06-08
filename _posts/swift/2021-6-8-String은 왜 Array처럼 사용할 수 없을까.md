---
layout: post
title: String은 왜 Array처럼 사용할 수 없을까?
category: Swift
tags: [String, Array, Unicode]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.        
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

요즘 코딩 테스트 문제를 풀어보면서 String을 다루는 경우가 많아졌습니다. 그 중에서 String의 특정한 순서에 있는 Character에 접근하고 싶어서 배열처럼 접근을 해보았는데 에러가 나더군요.

```swift
let str = "Odongnamu"
let someChar = str[3] 
// error: 'subscript(_:)' is unavailable: cannot subscript String with an Int, use a String.Index instead.
```

저는 String이 Character의 집합으로 배열과 비슷하게 접근이 가능할줄 알았는데 그렇지 않았습니다. String은 왜 배열처럼 사용할 수 없을까요? 그 내용은 [Swift Language Guide - Strings And Characters](https://docs.swift.org/swift-book/LanguageGuide/StringsAndCharacters.html#ID293)와 각종 공식문서에서 확인할 수 있었습니다.

<br>
<br>

# String은 Array인가?
결론부터 이야기하면 String 은 Array가 아닙니다! 저도 문서를 읽기전에는 Array와 사용하는 메서드도 비슷하고.. Character들이 나열된걸 보면 내부적으로는 배열로 구성되지 않았을까? 하는 생각이 들기도 했습니다. <br>

String과 Array가 비슷한 메서드를 사용하는 이유는 둘다 Collection 프로토콜을 채택하고 있기 때문입니다. Collection 프로토콜은 Sequence를 상속받고 있으니 String과 Array는 둘 다 Sequence, Collection의 종류라고 말할 수 있겠네요. <br>

![image](https://user-images.githubusercontent.com/73867548/119816257-760b1680-bf27-11eb-9739-7f9562eee83c.jpeg)   


그림을 보면 String은 `BidirectionalCollection - Collection - Sequence`를, Array는 `MutableCollection - Collection - Sequence`를 채택하고 있습니다. (모두 프로토콜입니다.)  <br>

> #### Collection Type       
>String은 Collection 프로토콜을 채택하고 있지만 Collection Type이라고 하는 것은 틀린 것 같습니다. Swift에는 3가지의 Collection Type (Array, Dictionary, Set)이 있습니다.

<br>

# String은 Index로 접근이 불가능한가?
String도 Collection을 채택하고 있기 때문에 first, last, count, split 등의 메서드를 사용할 수 있습니다. 하지만 위에서 Array처럼 특정 인덱스에 접근하려니까 에러가 발생했는데요, String은 Indext로 접근이 불가능한 것일까요? <br>

![image](https://user-images.githubusercontent.com/73867548/119818138-cbe0be00-bf29-11eb-9ebc-4626b9849f1c.jpeg)   

[Collection](https://developer.apple.com/documentation/swift/collection) 공식문서를 보면 Collection 프로토콜에 subscript가 구현되어 있습니다. 그리고 무려 Required 메서드입니다. 음.. 그렇다면 String에도 Index로 접근이 가능하도록 구현되어 있어야 하는게 아닐까요? <br>

#### String도 Index로 접근이 가능합니다. 다만 배열처럼 Int 타입의 인덱스로 접근할 수 없는 것이지요. String은 [String.Index](https://developer.apple.com/documentation/swift/string/index/) 타입으로 인덱스 접근이 가능합니다!

```swift
let str = "Odongnamu"
let someChar = str[str.index(str.startIndex, offsetBy: 4)]
print(someChar) // print: g
```

<br>

# String은 왜 Int 타입 Index로 접근할 수 없을까?
 그 이유는 String은 ASCII가 아닌 Unicode를 준수하고 있으므로 **각 Character는 각각 다른 크기의 메모리를 차지할 수 있기 때문입니다.** Unicode와 ASCII가 궁금하시다면 [ASCII, Unicode(유니코드), UTF-8 이해하기](https://odong-tree.github.io/cs/2021/06/01/ASCII_Unicode_UTF/) 포스팅을 가볍게 읽어보시는 것을 추천합니다 :) 
 
 > As mentioned above, different characters can require different amounts of memory to store, so in order to determine which Character is at a particular position, you must iterate over each Unicode scalar from the start or end of that String. For this reason, Swift strings can’t be indexed by integer values. - *[Swift Programming Language](https://docs.swift.org/swift-book/LanguageGuide/StringsAndCharacters.html#ID293)*


ASCII의 경우 각 Charcter는 모두 1byte의 크기를 가집니다. 만약 모든 글자의 크기가 1btye로 동일하다면 n번째 글자를 알고싶다면 'n'byte를 돌면서 조회를 해주면 됩니다. 하지만 글자마다 크기가 다른 Unicode의 경우에는 정수 단위로 Index를 조회할 수가 없는 것이지요. <br>

또한 유니코드의 경우 같은 글자라도 표시하는 방법이 여러개가 될 수 있습니다. 공식 문서의 예를 좀 보면,

```swift
let eAcute: Character = "\u{E9}"                         // é
let combinedEAcute: Character = "\u{65}\u{301}"          // e followed by ́
// eAcute is é, combinedEAcute is é

let precomposed: Character = "\u{D55C}"                  // 한
let decomposed: Character = "\u{1112}\u{1161}\u{11AB}"   // ᄒ, ᅡ, ᆫ
// precomposed is 한, decomposed is 한

let str = "\u{1112}\u{1161}\u{11AB}국"
let str2 = "한국"
```

이런 식으로 말입니다.    


따라서 Index를 조회하기 위한 새로운 단위로서 `String.Index`가 필요한 것입니다.

<br>

# 그렇다면 배열(Array)은 왜 Int 타입 접근이 가능할까?
Swift의 String이 각 글자의 크기가 불규칙해서 정수 타입의 인덱스를 사용할 수 없는 것이라면.. Array는 왜 정수 타입의 인덱스로 접근할 수 있는 걸까요? 각 요소의 크기가 당연히 다른게 아닐까요?? (이 부분에 대해서 알기위해 C 언어와 포인터에 대해서 공부를 하고 왔습니다! 그래서 포스팅이 조금 늦어진 것 같네요 ..ㅎ) <br>

Array는 메모리의 덩어리입니다. 즉, Array를 구성하는 요소들을 직접 담고 있는 것이 아니라 요소들이 저장되어 있는 메모리의 주소들을 가지고 있는 것입니다. 예를 들면 `0x00001234, 0x00001235, 0x00001236 ...` 이런 식으로 말입니다. 따라서 Array의 각 요소들의 크기는 다른게 맞지만 **실질적으로 Array가 담고있는 포인터들은 모두 같은 크기를 가지기 때문에 Int 타입의 인덱스로 접근이 가능합니다.**  <br>

참고로 포인터의 크기는 환경에 따라 다를 수 있지만 같은 환경 내에서는 크기가 모두 동일합니다. 컴파일러의 환경이 32비트에서는 4byte, 64bit에서는 8byte라고 하네요 :)







<br>
<br>

#### 참고
- [Apple 공식문서](https://developer.apple.com/documentation/swift/string)
- [Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/StringsAndCharacters.html)
- [Swift github](https://github.com/apple/swift/blob/main/stdlib/public/core/String.swift)
- [Unicode 네이버 백과](https://terms.naver.com/entry.naver?docId=2270340&cid=51173&categoryId=51173)
- [StackOverFlow](https://stackoverflow.com/questions/61122298/why-should-we-use-string-index-instead-of-int-as-index-of-character-in-string)
- [Sungdoo Yoo님 블로그](https://medium.com/@esung/swift의-문자열과-유니코드-af37a5d503a4)
- [포인터의 크기](https://m.blog.naver.com/123gtf/220905979083)

<br>
<br>