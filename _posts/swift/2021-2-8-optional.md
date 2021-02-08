---
layout: post
title: Optional  이해하기
category: Swift
tags: [Optional]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.         
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 optional에 대해서 정리해보려고 합니다. 처음에 옵셔널을 접했을 때에는 정말  너무 어렵다가도.. 계속 쓰다보니 많이 익숙해지더군요. 과거에 정리해뒀던 내용이지만, 블로그에 모아두기 위해 포스팅합니다!

<br>
<br>

# ☞ Swift는 Optional이 왜 필요할까요?

Swift의 옵셔널은 **안정성** 을 보장하기 위하여 사용하는 개념입니다. <br>

예를 들어서 어떤 변수에는 값이 할당될 수도 있고, 그렇지 않을 수도 있습니다.       
만약 코드가 실행되는 도중에, 값이 없는(할당되지 않은) 변수를 만나게 되면 오류가 발생하죠. 이러한 경우를 방지하기 위해 사용되는 것이 Optional입니다.       
**즉, nil이라는 없는 것에 대한 표현을 하기 위한 것이기도 하지요.** <br>


처음 옵셔널을 접했을 때, 그렇게 안정적인 것이라면, 모두 Optional로 하면 되지 않을까? 라고 생각을 했었습니다. <br>
하지만 모든 값을 옵셔널 타입으로 선언하게 되면, 모든 값에서 일일이 옵셔널에 대한 체크를 해야하기 때문에 프로그래밍이 복잡하고 무거워진다고 합니다. 그러므로 꼭 필요한 경우에만 제한적으로 사용하는 것이 좋다고 하네요 :)       

![스크린샷 2021-02-28 오후 1 35 50](https://user-images.githubusercontent.com/73867548/109408136-ec9c8480-79c9-11eb-9734-66f7f9fbb6dd.jpg)       

참고로 optional은 enum으로 이루어져  있습니다. xcode에서 `control + 클릭, 'Jump to definition'` 에서 확인할 수 있습니다.

<br>
<br>

# nil
nil은 위에서 말한 '값이 없음'을 나타냅니다.       
그리고 이 nil 값을 처리해주기 위한 것이 Optional 이라고 생각할 수 있습니다.  

```swift
let num = Int("123")
print(num) // Optional(123)

let num2 = Int("Hello")
print(num2) // nil
```

예를 들어 이렇게 Int에 String 타입을 넣을 경우,      
스위프트는 똑똑하게 "123"같이 숫자로 이루어진 문자열이라면 옵셔널 Int 값으로 변환해주고,           
그렇지 않은 "Hello"에는 nil 값을 부여합니다. <br>

그러니까, String 타입을 Int 타입으로 형변환을 시켜주어야 하는데 Int가 아닌 값을 넣어버리니 들어가지를 않는 겁니다. <br>

```swift
let num: Int?
```

그리고 num의 타입을 확인해보면 **Int?** 라고 되어있는 것을 확인할 수 있습니다. Optional은 이렇게 타입어노테이션에 ?를 붙여서 선언해줄 수 있습니다.   <br>

*물음표 ? 를 사용하니까 꼭*        
*"혹시 nil인가요? 값이 있나요?"*       
*물어보는 것 같네요 :D* <br>

<br>


# Optional Wrapping
그런데 옵셔널로 선언한 변수에 값을 할당하고 출력해보면 뭔가 이상합니다.

```swift
let num: Int? = 100
print(num) // Optional(100)
```

이렇게 할당한 값에 Optioanl이 감싸져서 나오는 것을 볼 수 있습니다. <br>

옵셔널 타입으로 반환된 값을 열어보면 실제 값이 옵셔널 타입으로 둘러싸여 있는 것을 볼 수 있습니다. 이것을 **옵셔널 래핑(Optional Wrapping)** 이라고 부릅니다. 야곰의 강의에서는 Optional을 **상자를 열어보는 것** 으로 비유를 해주었는데 이해가 잘 되더군요!      

![스크린샷 2021-02-28 오후 1 49 08](https://user-images.githubusercontent.com/73867548/109408319-d099e280-79cb-11eb-96d2-1ac3db6839f6.jpg)     
![스크린샷 2021-02-28 오후 1 49 25](https://user-images.githubusercontent.com/73867548/109408320-d1327900-79cb-11eb-9be4-ffdfe86a3df7.jpg)       


옵셔널 언래핑은 위와 같은 상자를 여는 것과 같습니다. 열어보았을 때 어떤 값이 있는지를 확인하고 그 값을 가져와야 합니다. 하지만 이때 옵셔널을 그대로 가져와버리면 위의 코드처럼 Optional(100)이라는 포장된 값을 얻어오게 됩니다. <br>

하지만 우리에게 필요한 값은 이렇게 옵셔널로 포장된 값이 아니라, 100이라는 실제 값이 필요합니다! 이러한 옵셔널 타입을 해제하는 것을 **옵셔널 언래핑(Optional Unwrapping)** 이라고 합니다.

<br>

# Optional Unwrapping
옵셔널 언래핑은 실제 값에 둘러싸인 옵셔널을 벗겨내고, **실제 값을 얻기위한 작업입니다.**

- **Forced unwrapping** - 박스를 강제로 까보자.
- **Optional binding** - 박스를 조심스럽게 까보자.
- **Nil coalescing** - 박스를 까봤는데 nil이라면 default 값을 할당하자.
- **Implicitly Unwrapped Optional** - 암시적 추출

<br>

### ✐ Forced unwrapping

Forced unwrapping은 강제로 언래핑을 시켜주는 방식입니다.       
어떤 값이 들어있건, 강제로 Optional을 벗겨낸 값을 얻을 수 있습니다.       

```swift
let num: Int? = 100
print(num!) // 100
```

이렇게 변수에다가 **!** 를 붙여주면 강제 추출이 가능합니다. <br>

강제 추출과 느낌표 !는 느낌이 비슷하네요.       
"나와!!" 하면서 막무가내로 꺼내는 느낌이랄까..? <br>

강제해제 방식은 단순한 방식이라 편하게 사용할 수 있다고 생각하실 수 있는데요,      
**하지만 편하다고 모든 경우에 사용할 수는 없습니다!!** <br>

nil 값이 할당된 변수를 강제 추출하게 되면 에러가 발생하다는 점을 유의해야 합니다. 그렇기 때문에 좀 더 조심스럽게 언래핑을 하는 방법으로는 아래와 같은 **옵셔널 바인딩** 이 있습니다.


<br>

### ✐ Optional binding

#### ▪️ if let

옵셔널 바인딩은 강제가 아닌, 조심스럽게 언래핑을 하는 방식입니다.       
if let을 사용하는 구문과 guard를 사용하는 구문으로 나눌 수 있는데요, <br>

먼저 if let을 사용해보기 전에,     
if 구문으로 옵셔널 바인딩을 해볼 수 있습니다.

```swift
var number = nil
let unwrappedNumber: Int

if number != nil {
	unwrappedNumber = number
    print(unwrappedNumber)
} else {
    print("number의 값이 없습니다.")
}
```

이런식으로 먼저 number의 값이 nil인지의 여부를 따져보고,      
nil일 때와 nil이 아닐 때의 경우로 나누어 값을 출력할 수 있습니다. 조심스럽게요. <br>


if let 구문은 위의 if 구문을 더 간단하게 만든 것입니다.       

```swift
var number = nil

func printNumber() {
  if let unwrappedNumber = number {
      print(unwrappedNumber)
  } else {
      print("number의 값이 없습니다.")
  }
}
```

if let 을 사용하면 이렇게 더 간단하게 코드를 작성할 수 있습니다.  <br>

*위의 코드는*       
*"만약 number가 nil이 아니라면, unwrappedNumber에 number 값을 할당하고, 출력한다. 만약 number가 nil이라면 "number의 값이 없습니다."를 출력한다." 이렇게 해석할 수 있겠네요.*

<br>

#### ▪️ guard
if let을 좀 더 간단하게 쓸 수 있는 것이 guard 구문입니다.

```swift
var number = nil

func printNumber() {
  guard let unwrappedNumber = number else {
      print("number의 값이 없습니다.")
      return
  }
  print(unwrappedNumber)
}
```

guard 구문을 사용하면 이렇게 작성할 수 있습니다. guard의 예시도 위의 if let 구문의 예시와 같은 의미를 가집니다. if let이나 guard나 else뒤에 오는 코드블록이 nil 일 때 실행되는 코드가 되는군요. <br>

하지만 if 구문과 guard 구문은 비슷하지만 다른 의미를 가지는데요,      
if 구문은 단순히 옵셔널 값 처리 결과에 따라 서로 다른 피드백을 주고 싶을 때 사용하지만,              
guard 구문은 조건에 맞지 않으면 무조건 함수의 실행을 종료시키는 특성이 있기 때문에             
실행 흐름상 옵셔널 값이 해제되지 않으면 더 이상 진행이 불가능할 정도로 큰일이 생길 때에만 사용하는 것이 좋습니다.

<br>

### ✐ nil coalescing
nil coalescing 도 if let 과 비슷한 의미로,       
할당하고자 하는 값이 nil일 경우, default 값을 할당하는 구문입니다.     

```swift
let myDogName: String = dogName ?? "바둑이"
```

*이렇게 작성된 구문은, myDogName에 dogName을 할당하는데, dogName이 nil이라면 "바둑이"를 할당한다. 라는 의미가 되겠네요!* <br>

왠지 삼항연산자와 비슷하게 생긴 것 같기도 합니다.


<br>

### ✐ Implicitly Unwrapped Optional
마지막으로 옵셔널 암시적 추출입니다.      

```swift
let num: Int! = 100
print(num) // 100
```

옵셔널 타입을 선언할 때 ? 대신 !를 쓰는 경우를 볼 수 있습니다.      
이러한 경우를 옵셔널 묵시적 해제라고 합니다. <br>

말 그대로, 직접 옵셔널 언래핑 작업을 하지 않아도     
알아서 옵셔널 언래핑을 해주는 것입니다. <br>

하지만 역시 추출할 값이 nil이라면 에러가 발생합니다.따라서 암시적 추출은 형식상 Optional로 정의해야하지만,절대 nil 값이 대입될 가능성이 없는 변수 일 때에만 사용해야 합니다. <br>

이러한 암시적 추출은 UI를 구성할 때, @IBOutlet에서 주로 사용이 됩니다.







<br>
<br>

#### 참고
- [https://www.youtube.com/watch?v=YBofMKyfDaQ](https://www.youtube.com/watch?v=YBofMKyfDaQ)

<br>
<br>
