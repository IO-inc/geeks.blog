---
title: RxSwift 기본 익히기 (1) — 10 분짜리
permalink: startrxswift1
---

![main image](https://cdn-images-1.medium.com/max/2000/1*i8d0WlTwViNNn4xiOGdLHQ.jpeg)


### Intro
개발자라면 Rx에 대해서 한번쯤은 들어봤을 거에요. 나도 1주일 전까지는 그냥 들어만 봤을 뿐이에요. 공부를 해보고 실제 사용해 봤습니다. 그리고 그 내용을 공유합니다.


##### Rx 장점은 이렇게 나열됩니다.

### Benefits
In short, using Rx will make your code:
- Composable <- Because Rx is composition’s nickname
- Reusable <- Because it’s composable
- Declarative <- Because definitions are immutable and only data changes
- Understandable and concise <- Raising the level of abstraction and removing transient states
- Stable <- Because Rx code is thoroughly unit tested
- Less stateful <- Because you are modeling applications as unidirectional data flows
- Without leaks <- Because resource management is easy


### 시작해볼까요?

먼저 Reactive Programming은 어떤건가 한번 봅시다.

> In computing, reactive programming is an asynchronous programming paradigm oriented around data streams and the propagation of change.
This means that it should be possible to express static (e.g. arrays) or dynamic (e.g. event emitters) data streams with ease in the programming languages used, and that the underlying execution model will automatically propagate the changes through the data flow. — wikipedia

> 리액티브 프로그래밍은 데이터 스트림과 변화의 증식에 기반한 비동기 프로그래밍 패러다임이다.

> Reactive Extensions (aka ReactiveX, Rx) is an API for asynchronous programming with observable streams.

> Rx는 Reactive programming을 위한 도구입니다.

> ReactiveX is a combination of the best ideas from
  the Observer pattern, the Iterator pattern, and functional programming. — http://reactivex.io/


**이 글만 가지고는 먼소리인지 잘 모르겠어요. 자세히 하번 살펴봅시다.** 

---
**Swift Version 의 Rx 인 RxSwift 를 가지고 설명해 보도록 할게요.**

# 1. Observables aka Sequences

Sequence는 프로그래밍에서 기본중에 기본 아닙니까? Sequence는 순차적이고 반복적으로 각각의 element에 접근 가능하도록 디자인된 데이터 타입입니다. 쉽게 Sequence 는 list 와 같이 반복문을 사용할 수 있는 데이터 타입을 말합니다.

```
let oneTwoThree = 1...3 
for number in oneTwoThree {     
  print(number) 
} 
// Prints "1" 
// Prints "2" 
// Prints "3"
```

RxSwift에서 Observables은 Sequences입니다. Arrays, Strings 와 같은 Sequences는 RxSwift에서 Observable이 됩니다. Swift Sequence Protocol을 따르는 모든 Object들의 Observable을 만들 수 있어요.


#### 한번 만들어 봅시다.

```
let stringSequence = Observable.just("this is string yo")
let oddSequence = Observable.from([1, 3, 5, 7, 9])
let dictSequence = Observable.from([1:"Rx",2:"Swift"])
```
#### 이렇게 만든 Observables를 subscribe 할 수 있습니다.

```
/*
subscribe(<#T##on: (Event<String>) -> Void##(Event<String>) -> Void#>)
*/
let subscription = stringSequence.subscribe { (event: Event<String>) in
  print(event)
}
// -- print --
// next(this is string observable)
// completed
```

##### 다른 예


```
let subscription = oddSequence.subscribe { (event: Event<Int>) in
  print(event)
}
// -- print --
// next(1)
// next(3)
// next(5)
// next(7)
// next(9)
// completed
let subscription = dictSequence.subscribe { (event: Event<(key: Int, value: String)>) in
  print(event)
}
// -- print --
// next((2, "Swift"))
// next((1, "Rx"))
// completed
```
어떤가요? Sequence처럼 출력 되지 않나요? Observables는 0 혹은 그보다 많은 event를 발생시킬 수 있습니다. RxSwift에서 Event는 enum 타입이고 3가지 상태를 가지고 있습니다.

```
// Event의 정의 입니다. 3가지 상태를 가지고 있어요.
enum Event<Element>  {
    case next(Element)      // next element of a sequence
    case error(Swift.Error) // sequence failed with error
    case completed          // sequence terminated successfully
}
class Observable<Element> {
    func subscribe(_ observer: Observer<Element>) -> Disposable
}

protocol ObserverType {
    func on(_ event: Event<Element>)
}
```

# 2. Disposing

Observables의 사용이 끝나면 메모리를 해제해야 합니다. 그 때 사용할 수 있는것이 Dispose입니다. RxSwift에서는 DisposeBag을 사용하는데 DisposeBag instance의 deinit() 이 실행 될 때 모든 메모리를 해제합니다.

이렇게요.

```
let disposeBag = DisposeBag()
let stringSequence = Observable.just("RxSwift Observable")
let subscription = stringSequence.subscribe { (event) in
  print(event)
}
// subscription 을 disposeBag에 넣어 메모리를 해제합니다.
subscription.addDisposableTo(disposeBag)
// 빠르게 비워주고 싶을때는 disposeBag을 새로 만들면 됩니다.
self.disposeBag = DisposeBag()
```

직접 dispose()를 호출 할 수 있습니다. subscription.dispose() 이렇게요.
하지만 직접 호출하는 것은 bad code smell 입니다. Thread가 다를 때 Observable을 사용하기도 전에 메모리를 비워주는 일이 발생할 수 있음.

메모리를 해제 하는 다른 방법으로 Take Until 따위가 있어요.

```
// 요런식으로 사용합니다.
sequence
    .takeUntil(self.rx.deallocated)
    .subscribe {
        print($0)
    }
```

# 3. Observable(aka observable sequence)을 만들어 봅시다.

처음에 우린

```
let stringSequence = Observable.just("this is string yo")
let oddSequence = Observable.from([1, 3, 5, 7, 9])
```

이런 방법으로 just, from 등을 이용해 Observable을 만들었어요. just, from을 사용하지 않고 직접 Observable을 만들어 봅시다.

Observable.create 함수를 사용해 Observable Sequence 를 쉽게 만들 수 있습니다.

```
func myJust<E>(element: E) -> Observable<E> {
    return Observable.create { observer in
        observer.on(.next(element))
        observer.on(.completed)
        return Disposables.create()
    }
}

myJust(element: 0)
    .subscribe(onNext: { n in
      print(n)
    })
```

create 함수는 Swift 의 closure를 사용해 우리가 위에서 사용한 subscribe 메서드를 쉽게 구현할 수 있도록 합니다.

# 4. Subjects

Rx 에서 Subject는 Observable 과 Observer 둘 다 될 수 있는 특별한 형태입니다. Subject는 Observables을 subscribe(구독) 할 수 있고 다시 emit(방출)할 수 도 있습니다. 혹은 새로운 Observable을 emit 할 수 있습니다.
그래서 이게 왜 필요할 까요?

일단 RxSwift에는 4가지 타입의 Subject가 있습니다.

- PublishSubject
- BehaviorSubject
- ReplaySubject
- Variable

### 4.1 PublishSubject

일단 사용해 보죠.
onNext() 함수로 새로운 값을 추가할 수 있습니다.

```
let bag = DisposeBag()
var publishSubject = PublishSubject<String>()
// Subject를 구독합니다.
publishSubject.subscribe { (event) in
  print(event)
}
// onNext로 값을 추가하면 event가 발생합니다.
publishSubject.onNext("first value")
// --- 출력 ---
// next(first value)
```
onNext로 두번째 값을 추가합니다.

```
publishSubject.onNext("second value")
// --- 출력 ---
// next(first value)
// next(second value)
```

onCompleted() 이벤트를 종료시킵니다.
```
publishSubject.onCompleted()
// --- 출력 ---
// next(first value)
// next(second value)
// completed
```

마지막으로 예제를 한번 보시죠.

```
let bag = DisposeBag()
var publishSubject = PublishSubject<String>()
publishSubject.onNext("very first value")
publishSubject.subscribe { (event) in
  print(event)
}
publishSubject.onNext("first value")
publishSubject.onNext("second value")
publishSubject.onError(NSError(domain: "", code: 1, userInfo: nil))
publishSubject.onNext("very last value")
publishSubject.onCompleted()
// --- 출력 ---
// next(first value)
// next(second value)
// error(Error Domain= Code=1 "(null)")
```

PublishSubject는 subscribe 전의 이벤트는 emit하지 않습니다. subscribe한 후의 이벤트만을 emit합니다. 그리고 에러 이벤트가 발생하면 그 후의 이벤트는 emit 하지 않습니다.


### 4.2 BehaviorSubject

BehaviorSubject는 PublishSubject와 거의 같지만 BehaviorSubject는 반드시 값으로 초기화를 해줘야 합니다. 즉 BehaviorSubject는 Observer에게 subscribe하기전 마지막 이벤트 혹은 초기 값을 emit합니다.

이렇게요.

```
var behaviorSubject = BehaviorSubject(value: "initial value")
behaviorSubject.subscribe { (event) in
  print(event)
}
behaviorSubject.onNext("first value")
// --- 출력 ---
// next(initial value)
// next(first value)
// completed
```

### 4.3 ReplaySubject

ReplaySubject는 미리 정해진 사이즈 만큼 가장 최근의 이벤트를 새로운 Subscriber에게 전달 합니다.

```
let bag = DisposeBag()
var replaySubject = ReplaySubject<String>.create(bufferSize: 1)
replaySubject.onNext("before subscribe first value")
replaySubject.onNext("before subscribe second value")
replaySubject.subscribe { (event) in
  print(event)
}
replaySubject.onNext("first value")
replaySubject.onCompleted()
// --- 출력 ---
// next(before subscribe second value)
// next(first value)
// completed
```

버퍼 사이즈가 1이기 때문에 가장 최근의 이벤트인 “before subscribe second value”를 새로운 구독자에게 전달합니다.

### 4.4 Variable

Variable 은 PublishSubject의 Wrapper 함수입니다. PublishSubject처럼 작동하며 더 익숙한 이름으로 사용하기 위해 만들어 졌습니다. 저도 Variable 많이 사용합니다.

# 5. Marble Diagram

Marble Diagram 은 Observable Sequence의 transformation을 시각화 해 보여줍니다. 시각화된 transformation은 이해를 돕죠. 한번 보시죠.

- [Web-app](http://rxmarbles.com) 
- [iOS-App](https://itunes.apple.com/com/app/rxmarbles/id1087272442)
- [Android](https://goo.gl/b5YD8K) 

---
> 쓰다보니 길어졌네요. 아무래도 다음이 필요할 것 같아요. 가능한 빠르게 다음 내용을 정리해 볼게요. 그럼 이만.

> 공유하거나 하트 누르거나 뭐 그냥 아무 것도 하지 않거나. 하지만 여기 까지 읽었으면 Rx 개념은 잡으셨죠?

[Why Rx](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Why.md)

