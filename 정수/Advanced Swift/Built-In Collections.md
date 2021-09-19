# Build-In Collections

<br>

이번 챕터에서는 Swift 내에서 사용되는 주요 콜렉션 타입들에 대하여 알아보도록 하겠습니다.

<br>

## Arrays

<br>

### Arrays and Mutability
---

Array(이하 배열)은 Swift내에서 가장 빈번히 사용되는 콜렉션 타입입니다.

배열은 동일한 타입의 요소들을 정렬된 형태로 갖는 컨테이너입니다.

예시로 하나의 정수 타입의 배열을 만들고자 한다면 아래와 같이 작성하여 생성할 수 있습니다.

```swift
// The Fibonacci numbers
let fibs = [0, 1, 1, 2, 3, 5]
```

만일 위 배열에서 `append(_:)` 함수를 사용하여 배열을 수정하고자 한다면 컴파일 에러가 발생합니다.

왜냐하면 위 배열은 `let` 키워드를 통해 생성되었기 때문에 변수가 아닌 상수 속성을 갖기 때문입니다.

이는 의도치않은 데이터의 변경을 막아줍니다.

만일 배열을 변수 타입으로 생성하고 싶다면 아래와 같이 `let` 키워드가 아닌 `var` 키워드를 통해 생성합니다.

```swift
var mutableFibs = [0, 1, 1, 2, 3, 5]
```

배열이 변수로 생성되었기 때문에 단일 요소 혹은 시퀀스를 덧붙일수 있습니다.

```swift
mutableFibs.append(8)
mutableFibs.append(contentsOf:[13, 21])
```

<br>

### Arrays and Optionals
---

Swift의 배열은 우리가 예상하는 대부분의 오퍼레이션을 제공합니다. 

또한 인덱스를 통한 서브스크립팅으로 특정 요소에 직접적인 접근도 가능합니다.

하지만 때에 따라 안정성을 위해 옵셔널 타입을 반환하는 경우가 존재합니다.

대표적인 예시로 배열의 `first` 와 `last` 속성의 경우에는 배열이 비어있을 경우 `nil` 값을 반환합니다.

`first` 프로퍼티는 내부에서 아래와 같은 로직으로 연산이 됩니다.

```swift
isEmpty ? nil : self[0]
```

`isEmpty` 로 현재 배열이 비어있는지를 판단하고 이 결과값에 따라 분기하여 결과값을 반환합니다.

<br>

## Transforming Arrays

<br>

### Map
---

배열 내 모든 요소들을 대상으로 동일한 연산을 이루고자 할 떄 사용하는 함수입니다.

map을 사용한 경우와 사용하지 않은 경우의 차이에 대해 살펴보도록 하겠습니다.

```swift
var squared: [Int] = []

for fib in fibs {
    squared.append(fib * fib)
}
```

위와 같이 `for` 문을 통해 배열 내 요소들을 순회하며 각각의 요소들에게 연산을 진행해줄수 있지만 `map` 을 이용하면 간편하며 더욱 가독성 좋은 코드를 작성할 수 있습니다.

```swift
let squared = fibs.map { $0 * $0 }
```

`map` 함수는 Sequence 타입에 대한 익스텐션으로 제공되며 아래와 같은 흐름으로 동작하게 됩니다.

```swift
extension Array {
    func jungsu_map<T>(_ transform: (Element) -> T) -> [T] {
        var result: [T] = []
        result.reserveCapacity(count)
        for x in self {
            result.append(transform(x))
        }

        return result
    }
}
```

(실제 Swift에서 제공하는 `map`은 위 연산에서 에러 처리를 위한 추가적인 로직이 존재합니다, 이는 추후 Error 챕터에서 다루도록 하겠습니다.)

<br>

### Filter
---

`map` 과 같이 매우 빈번히 사용되는 함수 중 하나인 `filter` 에 대해 알아보도록 하겠습니다.

`filter` 는 배열을 인자로 받아 특정 조건에 부합하는 원소들을 기반으로 생성된 배열을 반환합니다.

```swift
(1..<10).map { $0 * $0 }.filter { $0 % 2 == 0 }      //[4, 16, 36, 64]
```

범위 연산자를 통해 1~9까지의 숫자를 가지고, 이에 map을 통해 순차적으로 접근하여 제곱수를 구합니다. 이후 filter를 통해 짝수 값을 판별해냅니다.

`filter` 는 `map` 과 함께 사용할때 더욱 강력해집니다.


이를 통해 복잡한 로직의 코드를 더욱 간결하고 가독성이 좋게 표현할 수 있습니다.

`filter` 또한 `map` 과 아주 유사한 흐름으로 동작하게 됩니다.

위와 같이 동작하는 `filter` 을 배열 타입에 대한 익스텐션으로 정의해보겠습니다.

```swift
extension Array {
    func jungsu_filter(_ isIncluded: (Element) -> Bool) -> [Element] {
        var result: [Element] = []
        for x in self where isIncluded(x) {
            result.append(x)
        }

        return result
    }
}
```

보시다시피 for loop에 where 조건을 걸어 우리가 원하는 데이터만 쏙쏙 뽑아올수 있다는것을 제외하면 `map` 과 흐름이 매우 유사합니다.

<br>

### Reduce
---

`map` 과 `filter` 같이 배열을 기반으로 새로운 혹은 변경된 배열을 얻어내는 함수들도 있지만, 배열 내 모든 원소들을 기반으로 새로운 값을 생성해내는 함수도 있습니다.

예를 들어, 배열 내 모든 원소값을 더한 결과를 구해보겠습니다.

```swift
let fibs = [0, 1, 1, 2, 3, 5]
var total = 0

for num in fibs {
    total = total + num
}
```

단순 for loop을 이용한다면 위와 같이 작성하여서도 원하는 결과를 얻어낼 수 있습니다.

`reduce` 는 위 로직과 유사한 패턴을 가지지만 두 가지를 추상화하였습니다.

- 초기값(위 예시에서는 0이 됩니다.)
- 결과를 축적할 값(위 예시에서는 total입니다.)

위 로직을 `reduce`로 대체하면 아래와 같이 간결해집니다.

```swift
let total = fibs.reduce(0) { total, num in total + num }

let total = fibs.reduce(0, +)
```

<br>

### Flattening Map
---

코드를 작성하다보면 변환 클로저가 단일 원소를 반환하는 `map` 과는 달리 배열 자체를 반환하는 함수가 필요한 경우가 존재합니다.

예를 들어, 마크다운 파일 내 String을 읽어들여 내부에 기재되어 있는 모든 URL Link 들을 한데 모아 배열로 반환하는 함수를 작성한다고 가정해보겠습니다.

그렇다면 함수의 원형은 아래와 같이 작성되겠죠?

```swift
func extractLinks(markdownFile: String) -> [URL]
```

만일 Input으로 들어오는 마크다운 String이 여러개고 이를 순차적으로 접근하여 내부에 기재된 모든 Link를 추출하여 배열로 반환하고 싶은 경우는 어떻게 해야할까요?

이러한 경우 각각의 String에 대하여 `map` 을 수행한 결과를 가져오고 이들을 결합하여 최종적으로 단일 배열로 반환해야합니다.

하지만 `flatMap` 을 이용하면 더욱 간편하게 처리할 수 있습니다.

`flatMap` 은 변환 클로저의 반환값이 배열이라는 것을 제외하고는 `map` 과 매우 유사합니다.

```swift
extension Array {
    func jungsu_map<T>(_ transform: (Element) -> T) -> [T] {
        var result: [T] = []
        for element in self {
            result.append(transform(element))       // trnasform 클로저 타입: (Element) -> T
        }

        return result
    }

    func jungsu_flatMap(_ transform: (Element) -> [T]) -> [T] {
        var result: [T] = []
        for element in self {
            result.append(contentsOf: transform(element))       // transform 클로저 타입: (Element) -> [T]
        }

        return result
    }
}
```

두 함수간 차이가 보이시나요?

`map` 함수의 변환 클로저는 T 단일 원솔르 반환하지만 `flatMap` 함수의 변환 클로저의 경우에는 [T] 배열을 반환합니다.

이러한 특성으로 `RxSwift` 에서는 `flatMap` 을 통해 새로운 옵저버블 시퀀스를 생성할때에도 유용하게 사용됩니다.

<br>

## Dictionaries

<br>

이번에는 주요 자료구조중 하나인 Dictionary에 대해 알아보도록 하겠습니다.

Dictionary는 key와 이에 대응되는 value 값을 쌍으로 갖습니다.

key는 중복을 허용하지 않으며 key를 기반으로 value를 가져오는데에는 평균작으로 상수시간(O(1))이 소요됩니다.

Dictionary는 배열과는 달리 원소들이 정렬 되어있지 않습니다.

Dictionary를 이용하여 임의의 화면 셋팅값을 정의해보도록 하겠습니다.

```swift
enum Setting {
    case text(String)
    case int(Int)
    case bool(Bool)
}

let defaultSettings: [String: String] = [
    "Airplane Mode": .bool(false),
    "Name": .text("My iPhone")
]


defaultSettings["Name"]     // Optional(Setting.text("My iPhone"))
```

딕셔너리의 검색은 키 값이 존재하지 않을 경우 `nil` 값을 반환하기 때문에 optional 타입을 반환합니다. 

Array였다면 개발자의 실수로 OOB로 인한 크래쉬가 발생할 위험을 딕셔너리는 위와 같이 안전하게 처리합니다.

<br>

### Hashable Requirement
---

딕셔너리는 **해시테이블** 구조 입니다.

딕셔너리에 `key`의 해시값을 기반으로 위치를 분별하여 데이터를 저장합니다.

Swift 표준 라이브러리 내 기본적인 데이터 타입(ex: String, Int, Float, Bool...etc)들은 기본적으로 모두 `Hashable` 프로토콜을 채택하고 있으며 이는 딕셔너리의 `key` 값으로 사용이 가능합니다.

이것이 딕셔너리의 `key` 가 `Hashable` 프로토콜을 채택해야 하는 이유입니다.

만일 사용자가 정의한 데이터 타입을 딕셔너리의 `key`로 사용하고 싶다면, 반드시 해당 타입이 `Hashable` 프로토콜을 준수하도록 해야합니다.

`Hashable` 프로토콜은 `Equatable` 프로토콜을 준수하며 `hashValue`를 구현하는것을 강제합니다.

`Equatable` 프로토콜을 준수함으로써 사용자 정의 타입에 `==` 연산을 오버로드하여 동일한 해시값을 갖는지를 분별할 수 있도록 합니다.

<br>

