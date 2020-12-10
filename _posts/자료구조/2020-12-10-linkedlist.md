---
layout: post
title: 연결 리스트(Linked List), Swift로 구현하기
category: 자료구조
tags: [자료구조]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다. 정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다. <br>

최근에 자료 구조에 대해서 처음 접하게 되었습니다.    
처음에는 양도 많아보이고 어려운 것 같아서 버겁게만 느껴졌는데요,    
오늘 열심히 공부를 해보니 굉장히 매력적인 분야더군요 !    <br>

오늘은 그 중에 **연결 리스트(Linked List)** 에 대해서 알아보고   
Swift로 구현까지 해보도록 하겠습니다.

<br>

#  연결 리스트 (Linked List) 

연결 리스트는 배열, 스택, 큐 등과 함께 선형구조의 자료 구조 중 하나입니다.
또는 트리, 그래프와 함께 연결 데이터 구조로 분류되기도 합니다.   
연결 리스트는 스택과 큐 등 다른 자료 구조를 이루는 베이스로도 많이 활용되기 때문에    
자료구조에서 중요한 개념이라고 볼 수 있겠네요. <br>

> *연결 데이터 구조*
>- *연결 데이터 구조는 데이터 타입과 이를 다른 데이터와 묶어주는 포인터로 구성되는 데이터 구조입니다.*
>- *포인터란 메모리상의 위치 주소를 말합니다.*
>- *C언어 같은 로우 레벨 프로그래밍 언어와 달리, 스위프트는 직접적으로 포인터에 접근하지 않고 포인터를 활용할 수 있는 별도의 추상 체계를 제공합니다.* <br>


<br>


####  ✐ 연결 리스트 <br>

![연결리스트1](/assets/post-img/자료구조/연결리스트1.jpg)

* 연결 리스트는 일련의 ‘노드’로 구성되어 있다.
* 이 노드는 링크필드를 통해 서로 연결되어 있다.
* 가장 간단한 형태의 연결 리스트는 데이터와 다음 노드에 연결할 수 있는 링크 정보를 포함한다.
* 좀 더 복잡한 경우, 추가 링크 정보를 통해 연결된 데이터에서 앞 또는 뒤로 이동할 수 있다.
* 간단하게 노드를 추가하거나 삭제할 수 있다. <br>

연결 리스트는 자체 참조 클래스인 노드로 구성되고,    
각각의 노드는 데이터와 전체 연결 데이터에서 다음 노드로 이동할 수 있는 링크 정보를 포함합니다.    
위 사진에서는 짙은 파란색 부분이 링크를 담는 부분, 연한 하늘색 부분이 정보를 담는 부분으로 이해할 수 있습니다. (참고로 위의 예시는 단일 연결 리스트입니다.) <br>

>**배열과 비교해보면**
배열은 데이터에 접근하는 것이 굉장히 쉽고 빠른 반면, 연결 리스트는 링크를 타고타고 데이터에 접근해야하기 때문에 배열에 비해 비효율적입니다. 하지만 연결 리스트는 노드를 추가/삭제 하는데에 용이하다는 장점이 있습니다. <br>

연결 리스트의 종류에는 3가지가 있습니다.
* 단일 연결 리스트
* 이중 연결 리스트
* 원형 연결 리스트 <br>

![연결리스트2](/assets/post-img/자료구조/연결리스트2.jpg)

<br>

글로만 보는 것 보다 코드를 작성해보면 더 이해가 빠릅니다.   
이 예시는 유튜브의 **Lets Build That App** 채널에서 가져온 예시이며   
링크는 맨 아래에 첨부합니다.

```Swift
class Node {
    let value: Int
    var next: Node?

    init(value: Int, next: Node?) {
        self.value = value
        self.next = next
    }
}

class LinkedList {

    var head: Node?

    // 연결 리스트 출력함수
    func displayListItems() {
        print("Herre is whats inside of your list:")
        var current = head
        while current != nil {
            print(current?.value ?? "")
            current = current?.next
        }
    }

    // 아이템 끝에 추가
    func insert(value: Int) {
        // empty list
        if head == nil {
            head = Node(value: value, next: nil)
            return
        }

        var current = head
        while current?.next != nil {
            current = current?.next
        }

        current?.next = Node(value: value, next: nil)
    }

    // 아이템 삭제
    func delete(value:  Int) {
        if head?.value == value {
            head = head?.next
        }

        var prev: Node?
        var current = head
        // prev가 필요한 이유는, 삭제할 아이템을 탐색하여 삭제하는 시점에서 그 전의 데이터와 그 다음의 데이터를 연결시켜 주어야하기 때문이다.

        while current != nil && current?.value != value {
            prev = current
            current = current?.next
        }

        prev?.next = current?.next
    }

    // Special Insert
    func inserInOrder(value: Int) {
        if head == nil || head?.value ?? Int.min >= value {
            let newNode = Node(value: value, next: head)
            head = newNode
            return
        }

        var current = head
        while current?.next != nil && current?.next?.value ?? Int.min < value {
            current = current?.next
        }

        current?.next = Node(value: value, next: current?.next)
    }
}
```

<br>

1. value와 next를 가지는 Node 타입을 정의
  단일 연결 리스트는 일방향성을 가지기 때문에 next 프로퍼티까지만 만들어 주어도 됩니다.
  2. 연결리스트 출력메서드 추가
  이는 연결 리스트를 출력하여 눈으로 확인하기 위한 메서드입니다. 다른 메서드를 정의하시다보면 자연스럽게 이해할 수 있습니다.
  3. insert 메서드 추가
  먼저 현재의 head에 값이 있는지 없는지를 확인해주어 nil인 경우 next가 아닌 head에 값이 들어가게 합니다. 그리고 그렇지 않은 경우, next에 값을 넣기 위해 while문으로 next의 값이 nil이 될때까지 반복문을 실행하고, nil이 나오면 next에 값을 넣어줍니다.
  4. delete 메서드 추가
  insert와  비슷한 논리입니다. 다만 prev라는 프로퍼티를 임시적으로 만들어주는데, 해당 아이템을 삭제할 때 그 전의 아이템과 다음 아이템을 연결해주기 위함입니다.
  5. inserInOrder
  끝이 아닌 중간에 값을 삽입하기 위한 메서드입니다. insert, delete를 이해하는 방식으로 이해할 수 있습니다.

<br>
#### 연결 리스트의 저장 공간?
연결 리스트를 구현해보면서 데이터를 저장하는 공간의 형체가 어디있는지 한참을 찾았습니다.    
왜냐면 눈에 보이지 않았기 때문이지요.   
연결 리스트는 최초의 데이터를 head에 저장한 후, 그 다음은 head.next에, 그 다음은 head.next.next에, 또 그 다음은 head.next.next.next에 저장하는 것입니다. 또 그 다음은...   <br>
그러니까 이건 배열처럼 가시적인 저장공간을 설정하는 것이 아니라      
꼬리에 꼬리를 무는 프로퍼티의 연속이라고 할 수 있습니다.

<br>
