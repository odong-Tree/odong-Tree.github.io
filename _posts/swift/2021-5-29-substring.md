---
layout: post
title: Substring
category: Swift
tags: [Substring, String]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.        
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

Swift에는 Int, String, Double, Character... 등이 밥 먹듯이 사용하는 기본 타입들이 있습니다. 그중에서도 특히 Int와 String은 정말정말정말정말 많이 사용되는 것 같네요. 그런데 혹시 String 말고 **Substring**이라는 타입을 보신 적이 있으신가요?! 

![image](https://user-images.githubusercontent.com/73867548/120068459-c3b88800-c0bb-11eb-96d3-461caac17b8b.jpeg)

![image](https://user-images.githubusercontent.com/73867548/120068549-26aa1f00-c0bc-11eb-81fa-6da3d52e6395.jpeg)


저는 항상 무의식 속에서 String을 사용하다가 얼마전에 Substring이라는 타입을 처음 보게 되었습니다. `string의 한 부분`. String을 다루다보면 String의 부분에 대해서 다루는 경우도 꽤나 많습니다. 이때 당연히 String의 조각도 String 타입일 줄 알았는데 이 조각들을 Substring이라는 타입으로 구분해서 만들어두었더군요! 그래서 String의 몇몇 메서드를 보면 Substring을 반환하는 메서드들이 보입니다. <br>

그렇다면 Substring은 무엇이고 왜 필요할까요? 그냥 String만 있어도 될 것 같은데 말입니다! [공식문서](https://developer.apple.com/documentation/swift/substring/)와 [Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/StringsAndCharacters.html)를 토대로 작성합니다.

<br>
<br>

# Substring: A slice of a string

Substring은 언급했듯이 String의 한 부분입니다. **Substring 타입을 사용하는 이유는 더 빠르고 효율적이기 때문**입니다. <br>

Substring을 사용할 경우 원본인 String의 저장공간을 공유합니다. 무슨 말이냐면.. 만약 String의 부분을 String 타입으로 만든다면 그 새로운 String 부분에 대해서 하나의 인스턴스를 만들어 메모리를 할당해주어야 합니다. 하지만 부분을 Substring으로 만들게 될 경우 새로운 저장공간을 만들지않고 원본 Stirng의 저장공간을 사용합니다. <br>

또한 Substring과  String은 모두 [StringProtocol](https://developer.apple.com/documentation/swift/stringprotocol)을 준수하기 때문에 같은 메서드를 사용할 수 있습니다. (기능이 같습니다.) <br>

![image](https://user-images.githubusercontent.com/73867548/120070498-ce781a80-c0c5-11eb-8309-432814e1e0e1.jpeg)

 좀 더 정확하게 이야기하면 String으로 부터 Substring이 생겨나더라도 값을 복사하지 않고 사용할 수 있거나 값의 복사를 '미룰 수' 있습니다. Substring이 왜 효율적인지는 이해가 되려고 하는데 값의 복사를 **미룬다**는 말은 무슨 말일까요? <br>
 
Substring은 아래와 같은 조건에서 String 타입으로 바뀌게 되고 따라서 값을 복사하게 됩니다.
- Substring의 값을 저장해야할 때
- String 타입의 인스턴스를 요구하는 다른 함수에 값이 전달되어야 할 때

이때 `String(_:)` 이니셜라이저가 사용되면서 String으로 타입이 바뀌게 됩니다. 그러니까 임시적으로 사용하는 경우에는 값 복사가 일어나지 않기 때문에, 처음에는 원본의 저장공간을 사용하다가 따로 저장공간이 필요해질 때에 값을 복사해 보다 효율적으로 메모리를 사용할 수 있는 것입니다. 따라서 Substring은 최적화에 유리합니다.

<br>


> ### String.Subsequence
> ![image](https://user-images.githubusercontent.com/73867548/120070203-77be1100-c0c4-11eb-820d-7ee7ef51011a.jpeg)
split이나 suffix 메서드를 사용할 때 타입이 `String.SubSequence` 라고 나오기도 합니다. 공식문서에서는 분명 SubString을 반환한다고 했는데 말이죠. <br>
하지만 String.SubSequence 타입은 Substring을 typealias로 정의한 또 다른 이름입니다. 즉, Substring과 같은 타입입니다.   
![image](https://user-images.githubusercontent.com/73867548/120070324-e4d1a680-c0c4-11eb-8d61-91efc6067824.jpeg)


-----


<br>
<br>

#### 참고
- [Substring 공식문서](https://developer.apple.com/documentation/swift/substring/)
- [Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/StringsAndCharacters.html)
<br>
<br>
