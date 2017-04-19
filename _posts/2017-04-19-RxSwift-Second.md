---
title: title: RxSwift 기본 익히기 (2) — 9.59초 짜리
permalink: startrxswift2
---

![example](https://cdn-images-1.medium.com/max/2000/1*GFksbQsZ_t8Wqhk9Cq83og.png)

##6. 고차함수 사용하기

먼저 함수형 프로그래밍에 대해 잠깐 살펴볼까요? 그 유명한 밥아저씨가 말씀하시기를 Functional Programming은 대입문 없이 프로그래밍 하는것 이라고 했어요. 어떤 특징들 때문일까요?
순수 함수
f (x)라는 함수가 있습니다. f는 항상 입력된 인수에만 의존합니다. 함수 f는 달에서 인수 x를 넣고 함수 f(x)를 호출 하는것이나 지구에서 같은 인수 x를 넣고 f(x)를 호출하는 것이나 항상 같은 결과가 나옵니다. 같은 인수를 넣으면 언제나 같은 결과를 내는 함수를 순수함수라고 합니다. 지극히 수학적입니다. 상태에 의존적이지 않기 때문에 side effect가 없습니다.
고차 함수
함수형 프로그래밍에서는 함수도 값으로 취급합니다. 함수를 인수로 넘길 수도 있고 함수를 리턴할 수도 있습니니다.
``
```
func f(_ g: (() -> Void) ) -> (() -> Void) {
  let h = g
  return h
}
```

이런 식의 코딩이 가능합니다. 위 함수에서 인자로 넘겨지는 함수 g 를 callback 함수라고 부릅니다. 함수 f 에 의해 불려지기 때문이죠. 아래 처럼 사용 될 수 있습니다.

```
let list = 1...100
let result = list.reduce(0) { (r, element) -> Int in
  return r + element
}
print(result)
// --- 출력 ---
// 5050
```

함수형 프로그래밍에도 단점이 있습니다. 그 중하나는 상태가 없기때문에 변수 수정이 어려운 것입니다. Swift에서 타입 키워드로 let 만 사용한다고 생각하면 될것 같아요. 그래서 함수형 프로그래밍의 좋은 점을 추가하고 OOP 의 장점을 추가한 프로그래밍 언어들이 나오게 됐고 그중 하나가 Swift입니다.
Swift와 같은 언어들을 Multi Paradigm Language라고 합니다.
Swift Standard Library 에는 Sequences 를 다루는 고차함수들이 이미 구현 돼 있습니다. 그냥 사용하기만 하면 되죠.
- Map
- Reduce
- Filter
- FlatMap
- Foreach

등이 있습니다.
앞서서 첫번째 글에서 Observables 는 Sequence라고 했습니다. Rx는 비동기 이벤트를 마치 iterable 컬렉션처럼 간단하게 다루는 것입니다.
우린 RxSwift에서 위와 같은 고차 함수들을 가지고 Observable이 emit한 이벤트를 subscriber가 받기 전에 Transformation 혹은 combination 할 수 있습니다.

#### 시작해볼까요?

---

## 7. Operators By Category


_겁나 많은 Operators 하나씩 봅시다_

#### 7.1 Creating Observables

7.1 Creating Observables
- ### Create
1편에서도 설명했지만 Observable을 처음 부터 만들 때 사용하는 Operator입니다.
```
func myJust<E>(element: E) -> Observable<E> {
  return Observable.create { observer in
    observer.on(.next(element))
    observer.on(.completed)
    return Disposables.create()
  }
}
myJust(element: 0).subscribe(onNext: { n in
  print(n)
})
//--- 출력 ---
// 0
```
- ### Defer
defer는 observer가 subscribe할 때까지 기다리며 subscribe하면 그때 Observable을 생성합니다. defer(미루다) subscribe할 때까지 Observable 생성을 미룹니다.
```
let disposeBag = DisposeBag()
var count = 1
let deferredSequence = Observable<String>.deferred {
  print("\(count) 번째")
  count += 1
  return Observable.create { observer in
    print("Emitting...")
    observer.onNext("📱")
    return Disposables.create()
  }
}
deferredSequence.subscribe(onNext: { print($0) }).disposed(by: disposeBag)
deferredSequence.subscribe(onNext: { print($0) }).disposed(by: disposeBag)
// --- 출력 ---
// 1 번째
// Emitting...
// 📱
// 2 번째
// Emitting...
// 📱
```
- ### From
from은 자료구조화 된 데이터 타입들 Array, Dictionary따위 들을 Observable로 만들때 사용할 수 있습니다.
```
let disposeBag = DisposeBag()
let arrayType = ["📱", "⌚️", "💻", "🖥"]
Observable.from(arrayType)
 .subscribe(onNext: { print($0) })
 .disposed(by: disposeBag)
// --- 출력 ---
// 📱
// ⌚️
// 💻
// 🖥
```
- ### Of
Of는 같은 타입의 여러개의 element들을 observable로 만들 수 있습니다.
From 과 거의 비슷하지만 다른점이 있습니다. From은 데이터 타입과 자료구조들만 가능하고 Of는 같은 타입의 여러개를 Parameter로 받을 수 있어요.
// func of(_ elements: E ...) // 구현이 이렇게 돼 있습니다.
- ### Interval
Integer를 정해진 time interval마다 emit합니다.
- ### Just
가장 간단하게 Observable을 만들 수 있습니다. 한개나 여러개의 객체들을 Observable로 만들 수 있습니다.
```
Observable.just(1).subscribe { print($0) }.disposed(by: disposeBag)
// --- 출력 ---
// next(1)
// complete
```
- ### Range
정해진 Range에 따른 Integer를 방출합니다.
```
let disposeBag = DisposeBag()
Observable.range(start: 1, count: 3).subscribe { print($0) }.disposed(by: disposeBag)
// --- 출력 ---
// next(1)
// next(2)
// next(3)
// complete
```
- ### Generate
generate는 parameter로 C 타입 for 문이 들어갈 수 있어요.
```
Observable
.generate(initialState: 0, condition: {$0 < 3}, iterate: {$0 + 1})
.subscribe(onNext: { print($0) })
.disposed(by: disposeBag)
// --- 출력 ---
// 0
// 1
// 2
// C type for loop 
// for ( int i = 0; i< 3; i++) {}
```
- ### Repeat
생성한 Observable을 무기한 반복해서 방출합니다. take(_ count: Int) 메서드로 제한 할 수 있습니다.
```
Observable.repeatElement("🖥 사줘")
 .subscribe(onNext: { print($0) })
 .disposed(by: disposeBag)
// --- 출력 ---
// 🖥 사줘
// 🖥 사줘
// 🖥 사줘
...
```
- ### doOn
doOn은 Observer가 구독하기 전에 핸들링 할 수 있습니다.
```
Observable.of("📱", "⌚️", "💻", "🖥")
 .do(onNext: { print("Intercepted:", $0) },
     onError: { print("Intercepted error:", $0) },
     onCompleted: { print("Completed")  })
 .subscribe(onNext: { print($0) })
 .disposed(by: disposeBag)
// --- 출력 ---
// Intercepted: 📱
//📱
//Intercepted: ⌚️
//⌚️
//Intercepted: 💻
//💻
//Intercepted: 🖥
//🖥
//Completed
```
- ### Empty, Never, Throw
Empty는 Observable을 emit(방출)하지 않고 종료바로 종료하는 Observable을 만듭니다.
Never로 생성한 Observable은 emit하지도 않고 terminate(종료)하지도 않습니다.
Throw로 생성한 Observable은 emit하지 않고 error와 함께 종료합니다.

--- 

### 7.2 Combination Operators

Source Observable여러개를 하나의 Observable로 만듭니다.
- ### StartWith
Obervable이 보내는 데이터의 처음에 데이터를 추가합니다.
```
Observable.of("📱", "⌚️", "💻", "🖥")
 .startWith("startWith")
 .subscribe(onNext: { print($0) })
 .disposed(by: disposeBag)
// --- 출력 --- 
// startWith
// 📱
// ⌚️
// 💻
// 🖥
```
- ### Merge
여러개의 Observable sequence를 하나로 만듭니다.
```
let subject1 = PublishSubject<String>()
let subject2 = PublishSubject<String>()
Observable.of(subject1, subject2)
  .merge()
  .subscribe(onNext: { print($0) })
  .disposed(by: disposeBag)
subject1.onNext("1")
subject1.onNext("2")
subject2.onNext("A")
subject2.onNext("B")
subject2.onNext("3")
// --- 출력 ---
// 1
// 2
// A
// B
// 3
```
- ### Zip
최대 8개의 타입 까지 하나의 Observable Sequence의 element로 만들어 emit합니다.
```
let stringSubject = PublishSubject<String>()
let intSubject = PublishSubject<Int>()
Observable
.zip(stringSubject, intSubject) { stringElement, intElement in
  "\(stringElement) \(intElement)"
}
.subscribe(onNext: { print($0) })
.disposed(by: disposeBag)
stringSubject.onNext("A")
stringSubject.onNext("B")
intSubject.onNext(1)
intSubject.onNext(2)
stringSubject.onNext("AB")
intSubject.onNext(3)
// --- 출력 ---
// A 1
// B 2
// AB 3
```
그 외에도 combineLatest, switchLatest 등이 있습니다.
---


### 7.3 Transforming Operators
emit되는 Observable를 어떤 변형을 거친 후에 subscriber에게 보내도록 합니다.
- ### Map
emit된 Observable을 어떤 함수를 적용시켜 매핑한 후에 Observer에게 전달 합니다.
```
Observable.of(1, 2, 3)
.map { $0 * $0 }
.subscribe(onNext: { print($0) })
.disposed(by: disposeBag)
// --- 출력 ---
// 1
// 4
// 9
```
- ### FlatMap
Observable안에 또 다른 Observable이 있는 경우에 새로운 Observable 만들어야 합니다. 그때 사용할 수 있습니다. Observable Sequence 들로 만들어진 Observable Sequence를 하나의 Observable Sequence로 만들어 emit합니다.
```
let disposeBag = DisposeBag()
let sequence1  = Observable<Int>.of(1,2)
let sequence2  = Observable<Int>.of(1,2)
let sequenceOfSequences = Observable.of(sequence1,sequence2)
sequenceOfSequences
.flatMap{ return $0 }
.subscribe(onNext:{
  print($0)
}).disposed(by: disposeBag)
// --- 출력 ---
// 1
// 2
// 1
// 2
// flatMap을 빼고 그냥 subscribe하면 아래와 같이 Observable이 출력됩니다.
// --- 출력 ---
// RxSwift.ObservableSequence<Swift.Array<Swift.Int>>
// RxSwift.ObservableSequence<Swift.Array<Swift.Int>>
```
- ### Scan
Swift의 reduce와 비슷합니다.
정해 진 초기값(seed value)에 클로저를 적용시켜 하나의 Observable로 만듭니다.
```
Observable.of(10, 100, 1000)
 .scan(0) { aggregateValue, newValue in
 aggregateValue + newValue
}
.subscribe(onNext: { print($0) })
.disposed(by: disposeBag)
// --- 출력 ---
// 10
// 110
// 1110
```

### 7.4 Filtering and Conditional Operators
Source Observable로 부터 emit되는 item을 필터링 하거나 조건에 따라 emit하도록 합니다. 아래와 같은 Operator들이 존재합니다. 이거 예제를 추가해야하는데…
- ### filter
Filter는 Swift의 그것과 같이 Observable이 emit하는 item 을 필터링 해 subscriber에 보내기 위해 사용합니다.
```
Observable.of("📱", "⌚️", "💻", "🖥")
.filter { $0 == "🖥" }
.subscribe(onNext: { print($0) })
.disposed(by: disposeBag)
// --- 출력 ---
// 🖥
```
- ### distinctUntilChanged
이건 말이죠. Observable이 연속 같은 item 을 방출하면 무시합니다. subscriber는 연속해서 같은 item을 받지 않습니다.
```
Observable.of("📱", "⌚️", "⌚️", "💻", "📱", "💻", "🖥", "🖥")
  .distinctUntilChanged()
  .subscribe(onNext: { print($0) })
  .disposed(by: disposeBag)
// --- 출력 ---
//📱
//⌚️
//💻
//📱
//💻
//🖥
```
- ### elementAt
Observables는 Sequenecs 처럼 다룰 수 있습니다. 특정 순서에 나오는 item을 구독할 수 있습니다.
```
Observable.of("📱", "⌚️", "💻", "🖥")
  .elementAt(3)
  .subscribe(onNext: { print($0) })
  .disposed(by: disposeBag)
// --- 출력 ---
// 💻
```
- ### single
Swift Sequence 의 .first와 같습니다. 대신 매개변수로 조건을 넣을 수 있습니다.
```
Observable.of("📱", "⌚️", "💻", "🖥")
  .single()
  .subscribe { print($0) }
  .disposed(by: disposeBag)
// --- 출력 ---
//📱
```
- ### take
매개변수로 subscriber에게 넘겨줄 item의 개수를 정할 수 있습니다.
```
Observable.of("📱", "⌚️", "💻", "🖥")
  .take(2)
  .subscribe(onNext: { print($0) })
  .disposed(by: disposeBag)
// --- 출력 ---
//📱
//⌚️
takeLast
takeWhile
takeUntil
skip
skipWhile
skipWhileWithIndex
skipUntil
```

## 8. Schedulers
Operator들은 subscription이 생성된 스레드에서 돌아갑니다. 그리고 RxSwift를 사용할 때 특정한 스레드를 강제할 수 있습니다. operation-queue나 dispatch-queue에 익숙하시다면 껌입니다.
observeOn, 이나 subscribeOn메서드로 Thread를 지정할 수 있습니다.

- MainScheduler
- CurrentThreadScheduler
- SerialDispatchQueueScheduler
- ConcurrentDispatchQueueScheduler
- OperationQueueScheduler

위와 같은 Thread들이 있고 다음과 같이 사용할 수 있습니다.
```
Observable.of("📱", "⌚️", "💻", "🖥")
  .observeOn(CurrentThreadScheduler.instance)
  .subscribe(onNext: { print($0) })
  .disposed(by: disposeBag)
```

축하드려요. 이제 Rx시작하기를 끝내셨네요. 시간이 날때 마다 예제 코드나 설명을 수정 보완하도록 하겠습니다. 다음에는 RxCocoa에 대해서 다뤄보도록 하죠.
여기까지 읽으셨으면 피드백 꼭 주세요. ⚔️
그럼 이만…