---
layout: post
title: 고차함수 (Higher Order Function)
category: Swift
tags: [고차함수]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.   <br>

오늘은 고차함수에 대해서 알아보도록 하겠습니다. 고차함수에 대해서는 야곰닷넷에서 학습했는데요, 포스팅은 야곰닷넷의 내용과 공부한 내용을 토대로 작성합니다.

<br>

# 고차함수 (Higher Order Function)
고차함수는 ‘다른 함수를 전달인자로  받거나 함수실행의 결과를 함수로 반환하는 함수’를 뜻합니다. 스위프트의 문법이라고는 볼 수 없으나 유용한 기능들입니다. swift의 고차함수 **map, filter, reduce**에 대해서 알아보고 추가적으로 flatMap, compactMap 메서드에 대해서도 알아보도록 하겠습니다.
<br>

map, filter, reduce는 야곰 닷넷의 내용이 너무 좋아서 거의 그대로 가져왔습니다. 링크는 아래쪽에 첨부합니다 :)
<br>

### ✐ map
map 함수는 컨테이너 내부의 기존 데이터를 변형(tranform)하여 새로운 컨테이너를 생성합니다.     

```swift
// 변형하고자 하는 numbers, 변형 결과물을 담을 doubleNubers와 strings
let numbers: [Int] = [0, 1, 2, 3, 4]
var doubleNumbers: [Int]
var strings: [String]

// 기존의 for 구문 사용
doubleNumbers = [Int]()
strings = [String]()

for number in numbers {
    doubleNumbers.append(number * 2)
    strings.append("\(number)")
}

print(doubleNumbers) // [0, 2, 4, 6, 8]
print(strings) // ["0", "1", "2", "3", "4"]

// map 메서드 사용
// numbers의 각 요소를 2배하여 새로운 배열 반환
doubleNumbers = numbers.map({ (number: Int) -> Int in
    return number * 2
})

// numbers의 각 요소를 문자열로 변환하여 새로운 배열 반환
strings = numbers.map({ (number: Int) -> String in
    return "\(number)"
})

print(doubleNumbers) // [0, 2, 4, 6, 8]
print(strings) // ["0", "1", "2", "3", "4"]
```     
<br>

### ✐ filter
filter함수는 컨테이너 내부의 값을 걸러서 새로운 컨테이너로 추출합니다.

```swift
// 기존의 for 구문 사용
var filtered: [Int] = [Int]()

for number in numbers {
    if number % 2 == 0 {
        filtered.append(number)
    }
}

print(filtered) // [0, 2, 4]

// filter 메서드 사용
// numbers 요소 중 짝수를 걸러내어 새로운 배열로 반환
let evenNumbers: [Int] = numbers.filter({ (number: Int) -> Bool in
    return number % 2 == 0
})

print(evenNumbers) // [0, 2, 4]

// 매개변수, 반환 타입, 반환 키워드(return) 생략, 후행 클로저
let oddNumbers: [Int] = numbers.filter{ $0 % 2 != 0 }

print(oddNumbers) // [1, 3]
```     

<br>

### ✐ reduce
reduce함수는 컨테이너 내부의 콘텐츠를 하나로 통합합니다.

```swift
// 통합하고자 하는 someNumbers
let someNumbers: [Int] = [2, 8, 15]

// 기존의 for 구문 사용
// 변수 사용에 주목하세요
var result: Int = 0

// someNumbers의 모든 요소를 더합니다.
for number in someNumbers {
    result += number
}

print(result) // 25

// reduce 메서드 사용
// 초깃값이 0이고 someNumbers 내부의 모든 값을 더합니다.
let sum: Int = someNumbers.reduce(0, { (result: Int, currentItem: Int) -> Int in
    print("\(result) + \(currentItem)") // 어떻게 동작하는지 확인해보세요
    // 0 + 2, 2 + 8, 10 + 15
    return result + currentItem
})

print(sum) // 25

// 초깃값이 100이고 someNumbers 내부의 모든 값을 뺍니다.
var subtract: Int = someNumbers.reduce(0, { (result: Int, currentItem: Int) -> Int in
    print("\(result) - \(currentItem)") // 어떻게 동작하는지 확인해보세요
    // 0 - 2, -2 - 8, -10 - 15
    return result - currentItem
})

print(subtract) // -25

// 초깃값이 3이고 someNumbers 내부의 모든 값을 더합니다.
let sumFromThree = someNumbers.reduce(3) { $0 + $1 }

print(sumFromThree) // 28
```
<br>


# 고차함수를 사용할 때의 이점?
들어가기 앞서서.. 고차함수의 기능은 사실 swift에서 기본적으로 제공하는 문법인 **조건문, 반복문**으로 대체가 가능한 기능들인데요, 어떤 이점이 있길래 굳이 고차함수를 사용하는 것일까요?
<br>
####  ✐ 고차함수의 이점
- 코드가 간결해진다.
- chaining을 할 수 있다.
- 변수가 아닌 상수로 선언하여 해결해줄 수 있다.
- 성능상의 이점을 가질 확률이 더 크다.       
<br>

첫 번째, 두 번째 이점은 쉽게 확인이 가능합니다. 그렇다면 세 번째 이점은 어떤 내용일까요?

<br>

####  ✐ 변수가 아닌 상수로 선언하여 해결해줄 수 있다.
아래의 for문을 사용한 코드와 고차함수를 사용한 코드의 uppercaseAlphabets에 주목해봅시다.       

```swift
let lowercaseAlphabets = ["a", "b", "c", "d", "e"]
var uppercaseAlphabets = [String]()

for lowercaseAlphabet in lowercaseAlphabets {
    uppercaseAlphabets.append(lowercaseAlphabet.uppercased())
}
```

```swift
let lowercaseAlphabets = ["a", "b", "c", "d", "e"]
let uppercaseAlphabets = lowercaseAlphabets.map { (alphabet: String) -> String in
    return alphabet.uppercased()
}
```       

for문에서는 변수로, 고차함수에서는 상수로 선언해준 것을 볼 수 있습니다. 고차함수는 함수를 통해 변환을 마친 후, 상수에 **변환된 값을 한 번에** 꽂아주는 식인 반면, for문의 경우 **우선 변수로 선언**하여 for 문이 돌아가면서 차례대로 값이 추가, 제거 되는 식으로 변환하게 됩니다.  <br>

그런데 만약 for문을 사용하고 싶지만 추후에 값이 변할 수 없도록 상수로 선언해주고 싶다면 어떻게 해야할까요? <br>

**변수가 아닌 상수에 할당할 수 있다.** 이것이 고차함수를 사용하는 것에 대한 가장 큰 이점일 수 있습니다. 변수가 아닌 상수로 할당하는 것은 **컴파일러의 최적화와 성능상**으로도 훨씬 유리해진다는 이점이 있습니다.

<br>

# flatMap, compactMap
추가적으로 flatMap과 compactMap에 대해서도 알아보도록 하겠습니다. compactMap은 Swift 4.1버전에서 생겨난 메서드로 과거의 flatMap에서 하던 일정 기능을 수행하는 메서드입니다. 즉, flatMap이 flatMap과 compactMap으로 나누어진 것이라고 할 수 있습니다. <br>


#### ✐ flatMap
flatMap은 2차원 배열을 1차원 배열로 flat하게 만들어주는 메서드입니다. 예시를 보면 2차원 배열이 무엇인지는 바로 감이 오실겁니다. (2차원 배열은 배열 속의 배열로 구성된 배열입니다.)

```swift
let numbers = [1, 2, 3, 4]

let mapped = numbers.map { Array(repeating: $0, count: $0) }
// [[1], [2, 2], [3, 3, 3], [4, 4, 4, 4]]

let flatMapped = numbers.flatMap { Array(repeating: $0, count: $0) }
// [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
```

```swift
let someArray = [[1, 2, 3], [4, 5], [6, 7, 8, 9]]
let flatMap = someArray.flatMap { $0 }
print(flatMap) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```      
#### ✐ compactMap
compactMap은 배열 속의 nil을 제거해주는 함수입니다. 앞서 말했듯이 원래는 flatMap에 포함된 기능이었다고 하네요. compactMap은 2차원 배열과는 무관하며 1차원 배열에서 쓰이는 함수입니다.      

```swift
let possibleNumbers = ["1", "2", "three", "///4///", "5"]

let mapped: [Int?] = possibleNumbers.map { str in Int(str) }
// [1, 2, nil, nil, 5]

let compactMapped: [Int] = possibleNumbers.compactMap { str in Int(str) }
// [1, 2, 5]
```    

```swift
let someArray = [1, 2, 3, nil, 5, 6]
let compactMap = someArray.compactMap { $0 }
print(compactMap) // [1, 2, 3, 5, 6]
```    
<br>

그렇다면 2차원 배열에 속에 nil이 있다면 nil이 없는 1차원 배열로도 만들어 봅시다.      

```swift
let anyArray = [[1, 2], [nil, 3], [4, 5, nil, 7]]
let newArray = anyArray.flatMap { $0 }.compactMap { $0 }
print(newArray) // [1, 2, 3, 4, 5, 7]
```

<br>
<br>

#### 참고
- [https://blog.yagom.net/565/](https://blog.yagom.net/565/)
- [https://developer.apple.com/documentation/swift/sequence/2905332-flatmap](https://developer.apple.com/documentation/swift/sequence/2905332-flatmap)
- [https://developer.apple.com/documentation/swift/sequence/2950916-compactmap](https://developer.apple.com/documentation/swift/sequence/2950916-compactmap)

<br>
