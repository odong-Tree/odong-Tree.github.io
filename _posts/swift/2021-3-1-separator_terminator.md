---
layout: post
title: separator와 terminator
category: Swift
tags: [separator, terminator]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.        
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

print 구문 속에서 separator와 terminator를 보신적 있으신가요?         
separator와 terminator는 출력을 도와주는 parameter입니다.


# ✐ separator : 분리 기호

```
"1 더하기 2 더하기 3 더하기 4 더하기 5 더하기 6 더하기 7 더하기 8"
```

이런 문구를 출력하기 위해서는 어떻게 해야할까요? <br>

**separator**를 사용하면 출력문 사이사이에 원하는 문자열을 넣어줄 수 있습니다. 필요에 따라 유용한 기능일 것 같네요.

```swift
print(1, 2, 3, 4, 5, 6, 7, 8, separator: " 더하기 ")
// 출력 : 1 더하기 2 더하기 3 더하기 4 더하기 5 더하기 6 더하기 7 더하기 8
```

<img width="484" alt="8AA7D81D-83EB-4775-89B2-7800E6343154" src="https://user-images.githubusercontent.com/73867548/109509274-0f22c080-7ae4-11eb-9c0b-278f3861d022.png">

**separator의 default 값은 " " 입니다.** 출력할 item 사이사이에 스페이스바 한 번, 띄워쓰기가 들어가는 것이지요. <br>

따라서 separator를 따로 입력하지 않을 경우,

```swift
print(1, 2, 3, 4, 5, 6, 7, 8)
// 출력 : 1 2 3 4 5 6 7 8
```

이렇게 출력됩니다.


<br>

# ✐ terminator : 종결자

**terminator는 '종결'** 즉, 프린트의 마지막 부분에 추가되는 String 입니다.        

<img width="490" alt="1B35C9B9-40A2-4F11-A1FC-7CFD2303E451" src="https://user-images.githubusercontent.com/73867548/109509599-645ed200-7ae4-11eb-8d5e-9e83a056e1f2.png">


terminator의 default 값은 "\n", 줄바꿈 기호인데요, 그렇기 때문에 보통 terminator를 생략하고 프린트를 하게 되면       

```swift
print("안녕하세요.")
print("오동나무입니다.")

// 안녕하세요.
// 오동나무입니다.
```

이렇게 줄이 바뀐채로 프린트가 됩니다. 그렇다면 terminator에 " ", 스페이스바를 하나 넣으면 어떻게 될까요?

```swift
print("안녕하세요.", terminator: " ")
print("오동나무입니다.")

// 안녕하세요. 오동나무입니다.
```

이렇게 줄 바꿈이 일어나지 않고, 이어서 프린트되는 것을 확인할 수 있습니다. <br>

좀 더 명확하게는 for in 구문으로 확인해볼 수 있습니다.

```swift
for n in 1...5 {
    print(n, terminator: "")
}
// 출력 : 12345
```

원래 라면,         
1      
2       
3      
4      
5   <br>

이렇게 출력이 되었을 숫자들이 한 줄에 출력되는 것을 확인할 수 있습니다.
