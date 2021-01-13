---
layout: post
title: GCD, QoS와 DispatchGroup
category: iOS
tags: [GCD, DispatchQueue, DispatchGroup, QoS]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

[GDC, DispatchQueue 이해하기](https://odong-tree.github.io/ios/2021/01/12/dispatchqueue/) 포스팅에 이어서 QoS와 DispatchGroup에 대해서 알아보도록 하겠습니다.


# QoS (Quality of service)
QoS는 작업에 적용할 서비스의 Quality 또는 실행 우선 순위를 말합니다. 우선 순위?? 라고 해서 단순하게 순서만을 정하는 것은 아닙니다. QoS는 **앱의 효율적인 에너지 사용**과 밀접하게 관련됩니다. 오늘은 너무 깊지 않게 공부하도록 하고 추후에 다시 이 주제에 대해서 다루어 보도록 하겠습니다. 보다 자세한 내용은 [Prioritize Work with Quality of Service Classes](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html#//apple_ref/doc/uid/TP40015243-CH39-SW1) 문서에서 확인 할 수 있습니다 :)

<br>

### ✐ DispatchQoS
```swift
DispatchQueue.global(qos: DispatchQoS)
```
DispatchQueue의 global 메서드의 파라미터로  DispatchQoS 우선 순위를 할당할 수 있습니다. 그리고 시스템은 QoS 정보를 통해 스케쥴링, CPU 및 I/O 처리량, 타이머 대기 시간 등의 우선 순위를 조정합니다. DispatchQoS는 열거형 타입으로, 총 6개의 QoS 클래스가 있으며 4개의 주요 유형와 다른 2 개의 특수 유형으로 구분이 가능합니다. (높은 순서대로, userInteractive, userInitiated, default, utility, background, unspecified 입니다.)
<br>

### ✐ Primary QoS Classes

우선 순위가 높을 수록 더 빨리 수행되고, 더 많은 전력을 소모합니다. 그러므로 수행 작업에  적절한  QoS를 할당하면 앱이 더 **반응적(responsive)이고 보다 효율적인 에너지 사용**이 가능해집니다.

- **User-interactive**           
main thread에서 작업하며, 사용자 인터페이스 새로고침, 애니메이션  등 사용자와 상호작용하는 작업에  할당합니다. 작업이  빠르게 수행되지  않으면, 유저 인터페이스는 멈추게 됩니다. 반응성(responsiveness)과 성능(performance)에 중점을 둡니다.

- **User-initiated**         
문서를 열거나 버튼을 클릭해 액션을 수행하는 것처럼 빠른 결과를 요구하는 유저와의 상호작용 작업에 할당합니다. 몇 초  이내의 짧은  시간 내에 수행해야하는  작업으로 반응성과 성능에 중점을 둡니다.

- **Utility**          
데이터를 읽거나 다운로드하는 작업처럼 작업이 완료되는데에 어느정도  시간이 걸리거나 즉각적인 결과가 요구되지 않는 작업에 할당합니다. 반응성, 성능, 에너지 효율의 밸런스에 중점을 둡니다.

- **Background**          
index 생성, 동기화,  백업 등 사용자가 볼 수  없는, 백그라운드의 작업에 할당합니다. 에너지 효율에 중점을 둡니다.

<br>

### ✐ Special QoS Classes
- **Default**        
QoS를 할당해주지 않을 경우 기본값으로 사용되며 User Initiate와 Utility의 중간 수준의 레벨입니다.

- **Unspecified**          
Unspecified는 QoS의 정보가 없음을 나타내며, 시스템이 QoS를 추론해야 합니다.


<br>
<br>

# DispatchGroup
DispatchGroup은 비동기적으로 처리되는 작업들을 그룹으로 묶어, 그룹 단위로 작업 상태를 추적할 수 있는 기능입니다. 그러니까 DispatchGroup을 사용하면 async들을 묶어서 그룹의 작업이 끝나는 시점에 어떠한 동작을 수행시킬 수 있습니다. DispatchGroup의 기능에는 notify, wait, enter, leave 등이 있습니다. (DispatchGroup은 async에서만 유효하네요!)
<br>

### ✐ enter, leave
enter와 leave는 각각 DispatchGroup이 시작된다(등록된다), 끝난다(해제된다)는 의미입니다. DispatchGroup은 enter, leave를 사용는 방법과 파라미터로 그룹을 지정해주는 방법, 2가지가 있습니다.

```swift
// enter, leave를 사용하는 경우
group.enter()
DispatchQueue.main.async {}
DispatchQueue.global().async {}
group.leave()

// enter, leave를 사용하지 않는 경우
DispatchQueue.main.async(group: group) {}
DispatchQueue.global().async(group: group) {}
```
<br>

### ✐ notify
notify는 DispatchGroup의 업무 처리가 끝나는 시점에 원하는 동작을 수행하기 위한 메서드입니다.

```swift
let group = DispatchGroup()

DispatchQueue.global().async(group: group) {
    print("odongnamu")
}

DispatchQueue.global().async(group: group) {
    Thread.sleep(forTimeInterval: 2)
}

DispatchQueue.global().async(group: group) {
    for i in 1...3 {
        print(i)
    }
}

group.notify(queue: .main) {
    print("Finish")
}

/* 출력
odongnamu
1
2
3
(2초 후)
Finish
*/
```

notify의 파라미터 queue는 코드블록을 실행시킬 queue를 말합니다.

<br>

### ✐ wait
wait는 DispatchGroup의 수행이 끝나기를 기다리는 메서드입니다.

```swift
let group = DispatchGroup()

DispatchQueue.global().async(group: group) {
    print("odongnamu")
}

DispatchQueue.global().async(group: group) {
    Thread.sleep(forTimeInterval: 2)
}

DispatchQueue.global().async(group: group) {
    for i in 1...3 {
        print(i)
    }
}

group.wait()
print("Finish")

/* 출력
odongnamu
1
2
3
(2초 후)
Finish
*/
```

<br>
<br>


#### 참고
- [https://developer.apple.com/documentation/dispatch/dispatchqos](https://developer.apple.com/documentation/dispatch/dispatchqos)
- [https://developer.apple.com/documentation/dispatch/dispatchgroup](https://developer.apple.com/documentation/dispatch/dispatchgroup)
- [https://jinshine.github.io/2018/07/09/iOS/GCD(Grand%20Central%20Dispatch)/](https://jinshine.github.io/2018/07/09/iOS/GCD(Grand%20Central%20Dispatch)/)
- [https://developer.apple.com/library/.../EnergyGuide-iOS/PrioritizeWorkWithQoS.html#/](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html#//apple_ref/doc/uid/TP40015243-CH39-SW1)

<br>
<br>
