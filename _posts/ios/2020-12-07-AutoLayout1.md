---
layout: post
title: AutoLayout 1, 개념다지기
category: iOS
tags: [AutoLayout]
comments: true
---
안녕하세요 오동나무입니다.<br>

&nbsp;오늘은 오토레이아웃에 대해 공부해보려 합니다. 오토레이아웃은 스토리보드를 사용한다면 거쳐가지 않을 수 없는 아주 강력한 기능인데요, 간단한 것 같으면서도 맘대로 안되는게 참 답답하네요.. ㅠ 그래서 오토레이아웃을 공부해보려니 문서도 꽤나 읽어야겠더군요. 오늘부터 차근차근 오토레이아웃에 대해서 공부해봅시다!

<br>
# 오토레이아웃 (AutoLayout)

&nbsp;오토레이아웃은 왜 필요할까요? 오토레이아웃은 스크린의 크기가 다르다거나, 스크린의 비율이 다양하다는 점에서 발생하는 레이아웃의 문제를 해결해줍니다. 예를 들어서 가로모드와 세로모드를 모두 지원하는 경우, 혹은 아이폰6와 아이폰X처럼 스크린의 크기와 비율이 다른 경우, 모든 스크린에 대한 레이아웃을 만들지 않고 오토레이아웃으로 한 번에 구현할 수가 있습니다.<br>
&nbsp;이처럼 뷰는 외부적 요인과 내부적 요인에 따라 구성이 변할 수 있는데요, 이때 구성요소들이 적당한 자리를 찾아갈 수 있도록 절대적 위치가 아닌 상대적 위치를 정해주는 것이 오토레이아웃입니다. 그렇다면 상대적 위치는 어떻게 정할 수  있을까요? 오토레이아웃을 이해하기 위해서는 제약 조건(Constraints)을 잘 이해해야 합니다.
 <br>


### 제약 (Constraints)
&nbsp;오토레이아웃은 제약 조건(Constraints)을 이용하여 뷰의 위치를 정합니다. 제약 조건은 뷰의 구성요소들 간의 약속, 혹은 관계입니다. 즉 단독적일 수 없는 상대적인 값이 됩니다. <br>
<img src = "/assets/post-img/ios/2020-12/auto1.png" width = "90%">   
오토레이아웃은 이러한 수식으로 나타낼 수 있습니다. 이 수식은 스토리보드에서는
<br>
<img src = "/assets/post-img/ios/2020-12/auto2.jpg" width = "70%">   

이렇게 확인할 수 있습니다. 이 경우 파란 상자와 빨간 상자 사이에는 8만큼의 거리가 있다는 뜻이겠네요. 여기서 8의 단위는 point입니다. pixel이 아닙니다. <br>

&nbsp;오토레이아웃을 사용할 때에는 조건을 충족시키지 못하면 그 구성요소는 자기 자리를 찾아가지 못합니다. 여기서 충족되어야하는 조건에는 **x축의 너비와 위치, y축의 너비와 위치** 이렇게 4가지가 됩니다.    <br>

<img src = "/assets/post-img/ios/2020-12/auto3.png" width = "80%">

&nbsp;이 경우에는 위치는 정해주었지만 View의 사이즈를 정해주지 않았기 때문에 발생하는 에러입니다. Auto라면서 자동이 아니네요..? 일일이 다 명령을 해주어야 자리를 찾아갑니다. 그 이유는 컴퓨터는 자의적 해석을 하지 못하기 때문인데요, 사람의 경우 '저기에다 놔둬.'라고 하면 적당하게 놓아둘 수 있지만 컴퓨터의 경우에는 세세하게 명령을 해주어야 수행할 수 있습니다. <br>
&nbsp;그렇기 때문에 조건을 모두 충족시킬 수 있도록 제약을 추가해주어야 합니다. 위의 경우에는 x와 y의 너비에 대한 제약을 추가해주면 되겠네요. 그렇다면 제약은 어떻게 추가해줄 수 있을까요?
<br>

### 오토레이아웃 인터페이스
![auto4](/assets/post-img/ios/2020-12/auto4.jpg)
![auto5](/assets/post-img/ios/2020-12/auto5.jpg) <br>
&nbsp;오토레이아웃 인터페이스는 우측 하단에 5개가 모여있습니다.
- Update Frame: 구성요소를 제약사항에 맞게 위치시킵니다. (사진없음)
- Align: 구성요소의 배열를 정하는 메뉴
- Pin: 구성요소의 위치를 정하는 메뉴
- Resolve AutoLayout Issue: 제약사항에 대한 이슈를 해결하는 메뉴
- Embed In: 구성요소들을 원하는 뷰에 같이 넣어주는 메뉴
<br>
<img src = "/assets/post-img/ios/2020-12/auto6.jpg" width = "70%">   

인터페이스 말고도 Control을 누른채 드래그로 구성요소들을 이어주면 제약을 추가할 수 있습니다.
<br>

### Intrinsic Content Size
Intrinsic Content Size는 고유 콘텐츠 사이즈입니다. 공식문서는 아래와 같네요. <br>
![auto7](/assets/post-img/ios/2020-12/auto7.png) <br>

&nbsp;이처럼 button이나 Label,  switch, textfield 등의 경우에는 내부 콘텐츠의 사이즈를 고정할 수 있기 때문에 Intrinsic Content Size가 존재합니다. 즉 구성요소 자체가 크기를 가지기 때문에 사이즈를 특정하지 않아도 되는 요소가 됩니다. 따라서 내부 콘텐츠의 사이즈를 특정해놓는다면 제약조건 중 x, y의 크기, 위치 중에서 위치 값만 설정해주면 알아서 자리를 찾을 수 있는 요소들입니다.

<br>

### Priority
&nbsp;제약에는 우선도가 있습니다. 만약 제약이 중복되면 어떻게 될까요? 충돌이 일어납니다. 그렇기 때문에 컴퓨터는 어떤 명령을 우선적으로 따라야할지 정하지 못해서 에러를 발생시킵니다. 이렇게 어떤 명령을 먼저 따라야할지를 정해주는 것이 우선도입니다. <br>

![auto8](/assets/post-img/ios/2020-12/auto8.png)    
이 경우를 보면 하나의 제약이 점선으로 표시되어 있는데요, 이는 제약의 우선도가 낮아서 숨어버린 것입니다.   <br>
&nbsp;제약의 우선도는 기본적으로 1000의 값을 가지게 되며 숫자가 높은 순서대로 우선됩니다. 우선도는 1부터 1000까지의 양의 정수로 설정해줄 수 있는데요, 기본적으로 Xcode에서는 Low(250), High(750), Required(1000) 3가지가 제공됩니다. <br>

 <img src = "/assets/post-img/ios/2020-12/스크린샷%202020-12-07%20오후%208.30.57.jpg" width = "70%">


### Hugging, Compression
![autu9](/assets/post-img/ios/2020-12/autu9.png)   

&nbsp;Hugging과 Compression Resistance는 위치를 잡는 힘으로 우선도로서 그 순서가 결정됩니다. Hugging은 몸집에 맞게 사이즈를 유지하는 힘, Compression은 외부에서 콘텐츠를 누를때 버티는 힘인데요, 뭔가 알듯말듯하네요. <br>

![auto11](/assets/post-img/ios/2020-12/auto11.png)   

지금 3개의 레이블을 만들어 상하좌우에 제약을 모두 추가해주었지만 레이블들이 자리를 못잡고 있습니다. 빈 공간을 어떻게 해주어야할지를 컴퓨터가 결정하지 못하기 때문입니다. 이럴때 hugging으로 우선도를 정하여 해결해줄 수 있습니다. <br>

#### Hugging
Hugging은 **짬처리반**입니다. 우선도가 낮은 녀석이 남은 공간을 책임져야하는 운명이 되는 것이지요. 빨강에 1000, 파랑에 750, 노랑에 250의 우선도를 할당해볼까요? 어떻게 될까요? <br>

![auto10](/assets/post-img/ios/2020-12/auto10.png)   

이렇게 노랑이 남는 모든 공간을 처리하는 일이 발생합니다. hugging 우선도가 낮은 노랑이 짬처리를 했으니 해결된 것 같은데요. <br>

![auto12](/assets/post-img/ios/2020-12/auto12.png)    

하지만 빨강의 텍스트를 길게 써주니 다른 레이블들의 자리를 침범하는 것을 볼 수 있습니다. 남는 공간에 대해서는 우선도로 규칙이 정해졌지만, 어떤 레이블이 우선적으로 자리를 차지할지에 대해서는 약속되지 않았기 때문입니다. 이것을 정하는 것이 Compression Resistance 우선도 입니다.
<br>

#### Compression
Compression Resistance의 우선도를 빨강 1000, 파랑 750, 노랑 250으로 세팅한 후 빨강 레이블의 길이를 늘려보도록 하겠습니다. <br>

![auto13](/assets/post-img/ios/2020-12/auto13.png)   

앗. 이번에는 빨강이 다른 레이블들을 밖으로 밀어버리는 일이 발생했습니다. 빨강이 자리를 차지하는 힘이 가장 강하기 때문에 자기 욕심만큼 자리를 모두 차지해버렸네요. 좀 밸런스를 맞추어 보면
 <br>

![auto14](/assets/post-img/ios/2020-12/auto14.png)    

뭔가 평화로워보이네요. 하지만 노랑이 가장 힘이 약하기 때문에 원하는 만큼의 자리를 차지하지 못하고 ...으로 생략이 되어있는 것을 확인할 수 있습니다.
<br>
- Hugging: 우선도가 낮으면 남는 자리를 메꿔야하는 운명. 짬처리의 개념.

- Compression: 우선도가 높을 수록 우선적으로 자리를 잡는 개념.
<br>
>- 빨,파,노  순서로 hugging, resistatance가 높다?
>   - 레이블 총 길이 합이 전체 가로보다 크거나 같을 떄
        -> 빨, 파, 노 순서로 자리를 잡는다.
>    - 레이블 총 길이 합이 전체 가로보다 작을 떄
        -> 노랑이 짬처리 한다.
        <br>
>- 빨,파,노 순서로 hugging이 낮고, resistance가 높다?
>   - 레이블 총 길이 합이 전체 가로보다 크거나 같을 떄
        -> 빨, 파, 노 순서로 자리를 잡는다.
>   - 레이블 총 길이 합이 전체 가로보다 작을 떄
        -> 빨강이 짬처리 한다.
> <br>

<br>
