---
layout: post
title: Split vs Components
category: Swift
tags: [split, components, string]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.        
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>
오늘 처음으로 코딩테스트 문제를 풀어보았습니다. `A x B`라는 간단한 문제를 풀어보다가 split과 components에 대해 찾아보게 되었습니다. 포스팅은 공식문서, stackoverflow를 보며 공부한 내용을 토대로 정리합니다.

<br>
<br>

# Split vs Components
 우선 공통점은 둘다 String을 분리하는 방법입니다. 저는 5가지 측면에서 비교해보도록 하겠습니다.
<br>

![image](https://user-images.githubusercontent.com/73867548/119248194-812e1180-bbca-11eb-9ad9-318f473f6b5a.jpeg)

![image](https://user-images.githubusercontent.com/73867548/119248125-11b82200-bbca-11eb-9015-e351ce327bd3.jpeg)

<br>

## 1. parameter
```swift
// split
func split(separator: Character, maxSplits: Int = Int.max, omittingEmptySubsequences: Bool = true) -> [Substring]

// components
func components(separatedBy separator: CharacterSet) -> [String]
```

먼저 두 메서드의 파라미터를 살펴보면 split이 더 다양한 파라미터를 받을 수 있습니다.split의 파라미터를 먼저 살펴보도록 하겠습니다. 공식문서의 예제를 그대로 가져왔습니다.

- **separator**     
문자열을 나누는 기준입니다. 

- **maxSplits**   
문자열을 split하는 횟수를 전달하는 파라미터입니다.

- **ommittingEmptySubsequences**   
빈 시퀀스를 생략할지에 대한 Bool 값입니다. false를 전달할 경우 빈 시퀀스를 생략하지 않고 배열을 반환합니다.


```swift
// 1. separator
let line = "BLANCHE:   I don't want realism. I want magic!"
print(line.split(separator: " "))
// Prints "["BLANCHE:", "I", "don\'t", "want", "realism.", "I", "want", "magic!"]"

// 2. maxSplits
print(line.split(separator: " ", maxSplits: 1))
// Prints "["BLANCHE:", "  I don\'t want realism. I want magic!"]"

// 3. ommittingEmptySubsequences
print(line.split(separator: " ", omittingEmptySubsequences: false))
// Prints "["BLANCHE:", "", "", "I", "don\'t", "want", "realism.", "I", "want", "magic!"]"

```

<br>

#### **Character vs CharacterSet**    
그런데 공식문서를 살펴보면 components의 separator는 split의 separator와 조금 다른 것을 알 수 있습니다. split의 separator는 `Character` 타입을 전달받고, components의 separatedBy는 `CharacterSet` 타입을 전달받습니다. Charactor는 뭔지 알겠는데 CharacterSet은 무엇일까요? <br>
우선 Charactor는 Swift Standard Library에 속해있곡 CharacterSet은 Foundation 프레임워크에 속해있습니다. ChracterSet은 Objective-C의 NSCharacterSet 클래스를 브릿징한 것으로 따지고보면 Character보다 String과 유사합니다. <br>

>자세히는 모르겠으나, 저는 String과 거의 같은 것으로 이해가 되네요. 그런데 왜 String을 사용하지 않고 CharacterSet를 사용할까? 하는 의문이 들었는데요, 문서로는 잘 찾아지지 않아서 'Swift Algorithm Club' 카톡방에 질문을 해본 결과, 사용하는 이유는 예전부터 사용해오던 코드들, API들과의 호환성을 유지하기 위해서라고 하는 답변을 얻을 수 있었습니다 :)



그러므로 아래와 같이 **여러개의 Character를 기준으로 문자열을 나누어야할 경우** components를 사용하여 구현해줄 수 있습니다. Character 타입을 파라미터로 받는 split을 사용할 경우 에러가 발생합니다. "ng"는 Character 타입이 아니기 때문입니다.


```swift
import Foundation

let str = "odongnamu"
let newStrArray = str.split(separator: "ng") // error!!! 
print(newStrArray)

let newStrArray2 = str.components(separatedBy: "ng")
print(newStrArray2)
```


<br>

## 2. Availability
공식문서에 따르면 components는 iOS 2.0부터 split은 iOS 8.0부터 지원하고 있습니다.

<br>

## 3. return Type
components와 split은 반환하는 타입 역시 다릅니다. components는 `[String]` 타입을, split은 `[SubString]`을 반환합니다. SubString에 대해서는 따로 포스팅해서 링크를 남겨두도록 하겠습니다.

<br>

## 4. Framework
components는 Foundation 프레임워크에 속해있으므로 사용하기 위해서는 `import Foundation` 코드를 항상 작성해주어야 합니다. 반면 split은 Swift Standard Library에 속하기 때문에 Foundation이나 다른 프레임워크를 따로 import해주지 않아도 사용할 수 있습니다. <br>

 .+ 참고로 백준에서 문제를 풀 때 components를 사용하면 12초가 걸리던 코드가 `import Foundation`을 해주지 않는 split을 사용할 경우 8초로 단축되더군요. 여러 가지 테스트 결과 components, split의 차이로 시간이 단축되는 것은 아닌 것 같았습니다 :)

<br>

## 5. 성능
성능상 split이 components보다 효율적이라고 합니다. 그 이유로는 두 가지 정도를 찾았는데요, 아래와 같습니다.
1. split은 빈 시퀀스의 생략하기 때문에 더 효율적으로 작업을 처리하게 된다. (하지만 ommittingEmptySubsequences를 false로 해도 split이 더 빠른 걸로 측정이 되었습니다.)
2. split은 String에 속해있고, Components는 NSString에 속해있다. 따라서 Components의 경우 NSString과 브릿징을 하는 작업을 내부에서 해주고 있으므로 브릿징 작업을 하지 않는 split에 비해 퍼포먼스가 떨어집니다.

<br>

성능에 대해서는 [StackOverFlow](https://stackoverflow.com/questions/46344649/componentseparatedby-versus-splitseparator)에서 테스트 코드를 찾을 수 있었습니다.

```swift
var str = """
                One of those refinements is to the String API, which has been made a lot easier to use (while also gaining power) in Swift 4. In past versions of Swift, the String API was often brought up as an example of how Swift sometimes goes too far in favoring correctness over ease of use, with its cumbersome way of handling characters and substrings. This week, let’s take a look at how it is to work with strings in Swift 4, and how we can take advantage of the new, improved API in various situations. Sometimes we have longer, static strings in our apps or scripts that span multiple lines. Before Swift 4, we had to do something like inline \n across the string, add an appendOnNewLine() method through an extension on String or - in the case of scripting - make multiple print() calls to add newlines to a long output. For example, here is how TestDrive’s printHelp() function (which is used to print usage instructions for the script) looks like in Swift 3  One of those refinements is to the String API, which has been made a lot easier to use (while also gaining power) in Swift 4. In past versions of Swift, the String API was often brought up as an example of how Swift sometimes goes too far in favoring correctness over ease of use, with its cumbersome way of handling characters and substrings. This week, let’s take a look at how it is to work with strings in Swift 4, and how we can take advantage of the new, improved API in various situations. Sometimes we have longer, static strings in our apps or scripts that span multiple lines. Before Swift 4, we had to do something like inline \n across the string, add an appendOnNewLine() method through an extension on String or - in the case of scripting - make multiple print() calls to add newlines to a long output. For example, here is how TestDrive’s printHelp() function (which is used to print usage instructions for the script) looks like in Swift 3
          """

    var newString = String()

    for _ in 1..<9999 {
        newString.append(str)
    }

    var methodStart = Date()

 _  = newString.components(separatedBy: " ")
    print("Execution time Separated By: \(Date().timeIntervalSince(methodStart))")

    methodStart = Date()
    _ = newString.split(separator: " ")
    print("Execution time Split By: \(Date().timeIntervalSince(methodStart))")
```

저의 경우 Xcode Command Line Tool 프로젝트에서 테스트해보았고, 그 결과 split은 약 2.5초, components는 약 3.7초 정도가 소요되는 것을 확인할 수 있었습니다. <br>

그리고 메모리는 split의 경우 약 150MB 미만의 메모리를 사용하였고, components의 경우 300MB에 가까운 메모리를 사용하는 것으로 측정이 되었습니다.


<br>
<br>

> 작성에 도움을 주신 Mumani(vivi)에게 감사드립니다 :)






<br>
<br>

#### 참고
- https://developer.apple.com/documentation/swift/string/2894564-split
- https://developer.apple.com/documentation/foundation/nsstring/1410120-components/
- https://stackoverflow.com/questions/46344649/componentseparatedby-versus-splitseparator
- https://github.com/apple/swift/blob/7123d2614b5f222d03b3762cb110d27a9dd98e24/stdlib/public/core/Sequence.swift
- https://github.com/apple/swift/blob/7123d2614b5f222d03b3762cb110d27a9dd98e24/stdlib/public/Darwin/Foundation/NSStringAPI.swift

<br>
<br>