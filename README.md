# 📘 RxSwift Main Concepts Cheat Sheet

A comprehensive guide to RxSwift main concepts with definitions, lifecycle explanations, use cases, and practical code examples.

## Table of Contents
- [Observable](#1-observable)
- [Observer](#2-observer)
- [Disposable](#3-disposable)
- [Subjects](#4-subjects)
- [Relays (RxCocoa)](#5-relays-rxcocoa)
- [Special Observable Types](#6-special-observable-types)
- [Transformation Operators](#7-transformation-operators)
- [Filtering Operators](#8-filtering-operators)
- [Combining Operators](#9-combining-operators)
- [Error Handling](#10-error-handling)
- [Schedulers](#11-schedulers)
- [DelegateProxy (RxCocoa)](#12-delegateproxy-rxcocoa)
- [Binder (RxCocoa)](#13-binder-rxcocoa)
- [RxTest](#14-rxtest)
- [Quick Reference](#quick-reference)

---

## 1. Observable

**Definition**: A sequence of events that can send values (onNext), errors (onError), or completion (onCompleted).

**Lifecycle**:
- Starts when someone subscribes
- Emits onNext values
- Ends with onCompleted or onError

**Use case**: Represent async data like API responses, text field changes, timers.

```swift
let numbers = Observable.of(1, 2, 3)

numbers.subscribe(
    onNext: { print("Value:", $0) },
    onCompleted: { print("Done") }
)
// Output: Value: 1, Value: 2, Value: 3, Done
```

**Common Creation Methods**:
```swift
// Create from array
Observable.of(1, 2, 3)

// Create from single value
Observable.just("Hello")

// Create from array
Observable.from([1, 2, 3])

// Create empty
Observable.empty()

// Create never ending
Observable.never()

// Create with error
Observable.error(MyError())
```

---

## 2. Observer

**Definition**: The "listener" that reacts to Observable events.

**Lifecycle**: Active until disposed, completed, or error happens.

**Use case**: Handle values from Observables.

```swift
numbers.subscribe(onNext: { value in
    print("Observer received:", value)
})
```

**Observer Pattern**:
```swift
let observer = AnyObserver<String> { event in
    switch event {
    case .next(let value):
        print("Next:", value)
    case .error(let error):
        print("Error:", error)
    case .completed:
        print("Completed")
    }
}

Observable.of("A", "B", "C").subscribe(observer)
```

---

## 3. Disposable

**Definition**: A token that represents a subscription you can cancel.

**Lifecycle**: Active until `.dispose()` is called or added to DisposeBag.

**Use case**: Prevent memory leaks.

```swift
let disposeBag = DisposeBag()

Observable.of("A", "B")
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```

**Manual Disposal**:
```swift
let subscription = Observable.of(1, 2, 3)
    .subscribe(onNext: { print($0) })

// Later...
subscription.dispose()
```

---

## 4. Subjects

**Definition**: Special observables that can also accept new values.

**Types & Lifecycle**:
- **PublishSubject** → emits new items after subscription
- **BehaviorSubject** → keeps the latest value + new items
- **ReplaySubject** → replays a buffer of old items
- **AsyncSubject** → only emits the last value after completion

**Use case**: When you want to push values into a stream.

### PublishSubject
```swift
let subject = PublishSubject<String>()
subject.onNext("Before subscription") // not seen

subject.subscribe(onNext: { print("Subscriber:", $0) })
subject.onNext("Hello")
subject.onNext("RxSwift")
// Output: Subscriber: Hello, Subscriber: RxSwift
```

### BehaviorSubject
```swift
let behaviorSubject = BehaviorSubject(value: "Initial")
behaviorSubject.onNext("Updated")

behaviorSubject.subscribe(onNext: { print("Behavior:", $0) })
// Output: Behavior: Updated
```

### ReplaySubject
```swift
let replaySubject = ReplaySubject<String>.create(bufferSize: 2)
replaySubject.onNext("A")
replaySubject.onNext("B")
replaySubject.onNext("C")

replaySubject.subscribe(onNext: { print("Replay:", $0) })
// Output: Replay: B, Replay: C
```

---

## 5. Relays (RxCocoa)

**Definition**: Similar to Subjects but never end with error/completion.

**Lifecycle**: Always alive, updated with `.accept()`.

**Use case**: Manage app state (e.g., text fields, toggles).

```swift
import RxCocoa

let relay = BehaviorRelay(value: "Initial")
relay.accept("Updated")

relay.asObservable()
    .subscribe(onNext: { print("Relay:", $0) })
    .disposed(by: disposeBag)
// Output: Relay: Updated
```

### PublishRelay
```swift
let publishRelay = PublishRelay<String>()
publishRelay.accept("Hello")

publishRelay.subscribe(onNext: { print($0) })
publishRelay.accept("World")
// Output: World
```

---

## 6. Special Observable Types

**Single**: Emits one value or error.
**Maybe**: Emits one value, or none, or error.
**Completable**: Only completes or errors.
**Driver (RxCocoa)**: UI-safe (no error, main thread, replay latest).
**Signal (RxCocoa)**: For UI events (no replay).

**Use case**:
- Single → API calls
- Driver → UI bindings
- Signal → Button taps

```swift
// Single
Single.just("Loaded")
    .subscribe(onSuccess: { print($0) })
    .disposed(by: disposeBag)
// Output: Loaded

// Maybe
Maybe.just("Maybe value")
    .subscribe(
        onSuccess: { print("Success:", $0) },
        onCompleted: { print("Completed with no value") }
    )
    .disposed(by: disposeBag)

// Driver
let driver = BehaviorRelay(value: "Driver")
    .asDriver()

driver.drive(onNext: { print("Driver:", $0) })
    .disposed(by: disposeBag)
```

---

## 7. Transformation Operators

**Definition**: Modify values from a stream.

**Lifecycle**: Keep emitting until source completes.

**Use case**: Change data before passing it on.

### map
```swift
Observable.of(1, 2, 3)
    .map { $0 * 10 }
    .subscribe(onNext: { print($0) })
// Output: 10, 20, 30
```

### flatMap
```swift
Observable.of("A", "B")
    .flatMap { letter in
        Observable.of("\(letter)1", "\(letter)2")
    }
    .subscribe(onNext: { print($0) })
// Output: A1, A2, B1, B2
```

### compactMap
```swift
Observable.of("1", "two", "3")
    .compactMap { Int($0) }
    .subscribe(onNext: { print($0) })
// Output: 1, 3
```

### scan
```swift
Observable.of(1, 2, 3)
    .scan(0) { accumulator, value in
        accumulator + value
    }
    .subscribe(onNext: { print($0) })
// Output: 1, 3, 6
```

---

## 8. Filtering Operators

**Definition**: Control which values are allowed through.

**Lifecycle**: Emit only matching values.

**Use case**: Filter noisy streams (like search input).

### distinctUntilChanged
```swift
Observable.of(1, 1, 2, 3, 3)
    .distinctUntilChanged()
    .subscribe(onNext: { print($0) })
// Output: 1, 2, 3
```

### filter
```swift
Observable.of(1, 2, 3, 4, 5)
    .filter { $0 % 2 == 0 }
    .subscribe(onNext: { print($0) })
// Output: 2, 4
```

### take
```swift
Observable.of(1, 2, 3, 4, 5)
    .take(3)
    .subscribe(onNext: { print($0) })
// Output: 1, 2, 3
```

### skip
```swift
Observable.of(1, 2, 3, 4, 5)
    .skip(2)
    .subscribe(onNext: { print($0) })
// Output: 3, 4, 5
```

---

## 9. Combining Operators

**Definition**: Merge values from multiple observables.

**Lifecycle**: Emits combined values based on operator.

**Use case**: Sync multiple inputs.

- **combineLatest**: Latest values of each
- **zip**: Pairs by index
- **merge**: Emits all as they come

### combineLatest
```swift
let first = PublishSubject<String>()
let second = PublishSubject<String>()

Observable.combineLatest(first, second) { "\($0) \($1)" }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

first.onNext("Hello")
second.onNext("World")
// Output: Hello World
```

### zip
```swift
let letters = Observable.of("A", "B", "C")
let numbers = Observable.of(1, 2, 3)

Observable.zip(letters, numbers) { "\($0)\($1)" }
    .subscribe(onNext: { print($0) })
// Output: A1, B2, C3
```

### merge
```swift
let first = Observable.of(1, 3, 5)
let second = Observable.of(2, 4, 6)

Observable.merge(first, second)
    .subscribe(onNext: { print($0) })
// Output: 1, 2, 3, 4, 5, 6 (order may vary)
```

---

## 10. Error Handling

**Definition**: Catch or replace errors.

**Lifecycle**: If not handled, stream stops at onError.

**Use case**: Prevent crashes, retry operations.

### catchErrorJustReturn
```swift
Observable.of("1", "two", "3")
    .map { Int($0)! }
    .catchErrorJustReturn(0)
    .subscribe(onNext: { print($0) })
// Output: 1, 0
```

### retry
```swift
var attempt = 0
Observable<String>.create { observer in
    attempt += 1
    if attempt < 3 {
        observer.onError(NSError(domain: "Test", code: 0))
    } else {
        observer.onNext("Success!")
        observer.onCompleted()
    }
    return Disposables.create()
}
.retry(3)
.subscribe(
    onNext: { print($0) },
    onError: { print("Failed:", $0) }
)
```

### catchAndReturn
```swift
Observable.error(NSError(domain: "Test", code: 1))
    .catchAndReturn("Default value")
    .subscribe(onNext: { print($0) })
// Output: Default value
```

---

## 11. Schedulers

**Definition**: Control threads where work runs.

**Lifecycle**: 
- `subscribeOn` = where it works
- `observeOn` = where it delivers results

**Use case**: Background work + UI updates on main thread.

```swift
Observable.of(1, 2, 3)
    .subscribeOn(ConcurrentDispatchQueueScheduler(qos: .background))
    .observeOn(MainScheduler.instance)
    .subscribe(onNext: { print("On main:", $0) })
    .disposed(by: disposeBag)
```

**Common Schedulers**:
```swift
// Main thread (UI updates)
MainScheduler.instance

// Background concurrent
ConcurrentDispatchQueueScheduler(qos: .background)

// Serial queue
SerialDispatchQueueScheduler(qos: .default)

// Current thread
CurrentThreadScheduler.instance
```

---

## 12. DelegateProxy (RxCocoa)

**Definition**: Converts delegate callbacks into observables.

**Lifecycle**: Active until disposed.

**Use case**: React to UIKit events without writing delegate code.

```swift
import RxCocoa

scrollView.rx.contentOffset
    .subscribe(onNext: { print("Offset:", $0) })
    .disposed(by: disposeBag)

tableView.rx.itemSelected
    .subscribe(onNext: { indexPath in
        print("Selected row:", indexPath.row)
    })
    .disposed(by: disposeBag)
```

**Common RxCocoa Extensions**:
```swift
// UITextField
textField.rx.text.orEmpty
    .subscribe(onNext: { print("Text:", $0) })

// UIButton
button.rx.tap
    .subscribe(onNext: { print("Button tapped") })

// UITableView
tableView.rx.modelSelected(MyModel.self)
    .subscribe(onNext: { model in
        print("Selected:", model)
    })
```

---

## 13. Binder (RxCocoa)

**Definition**: Type-safe binding to UI, always on main thread.

**Lifecycle**: No errors, no completion.

**Use case**: Bind text, visibility, etc.

```swift
let label = UILabel()
let textStream = BehaviorRelay(value: "Hello RxSwift")

textStream.bind(to: label.rx.text).disposed(by: disposeBag)
```

**Custom Binder**:
```swift
extension Reactive where Base: UIView {
    var isHidden: Binder<Bool> {
        return Binder(self.base) { view, hidden in
            view.isHidden = hidden
        }
    }
}

// Usage
let isHidden = BehaviorRelay(value: false)
isHidden.bind(to: myView.rx.isHidden).disposed(by: disposeBag)
```

---

## 14. RxTest

**Definition**: A test framework for RxSwift using virtual time.

**Lifecycle**: Simulates events with a scheduler.

**Use case**: Test debounce, throttle, merge, etc.

```swift
import RxTest

let scheduler = TestScheduler(initialClock: 0)
let observer = scheduler.createObserver(Int.self)

let observable = scheduler.createColdObservable([
    .next(10, 1),
    .next(20, 2),
    .completed(30)
])

observable.bind(to: observer).dispose()
scheduler.start()

print(observer.events)
// Output: [next(10, 1), next(20, 2), completed(30)]
```

**Testing Example**:
```swift
func testDebounce() {
    let scheduler = TestScheduler(initialClock: 0)
    
    let input = scheduler.createColdObservable([
        .next(10, "a"),
        .next(20, "b"),
        .next(30, "c"),
        .next(100, "d")
    ])
    
    let result = scheduler.createObserver(String.self)
    
    input
        .debounce(.seconds(50), scheduler: scheduler)
        .bind(to: result)
        .dispose()
    
    scheduler.start()
    
    XCTAssertEqual(result.events, [
        .next(80, "c"),  // "c" after 50ms delay
        .next(150, "d")  // "d" after 50ms delay
    ])
}
```

---

## Quick Reference

### Observable Creation
```swift
Observable.just(value)           // Single value
Observable.of(1, 2, 3)          // Multiple values
Observable.from([1, 2, 3])      // From array
Observable.empty()              // Empty sequence
Observable.never()              // Never emits
Observable.error(error)         // Error immediately
```

### Subject Types
```swift
PublishSubject<T>()             // New subscribers get future events
BehaviorSubject(value: initial) // New subscribers get latest + future
ReplaySubject.create(bufferSize: n) // Replay last n events
```

### Relay Types (RxCocoa)
```swift
PublishRelay<T>()               // Like PublishSubject but no error/complete
BehaviorRelay(value: initial)   // Like BehaviorSubject but no error/complete
```

### Common Operators
```swift
.map { }                        // Transform values
.filter { }                     // Filter values
.flatMap { }                    // Transform to new Observable
.distinctUntilChanged()         // Skip consecutive duplicates
.debounce(.milliseconds(300))   // Wait for pause in events
.throttle(.seconds(1))          // Limit rate of events
.take(3)                        // Take first 3
.skip(2)                        // Skip first 2
.catchErrorJustReturn(default)  // Handle errors
.retry(3)                       // Retry on error
```

### Schedulers
```swift
.subscribeOn(scheduler)         // Where work happens
.observeOn(scheduler)           // Where results delivered
MainScheduler.instance          // Main thread
ConcurrentDispatchQueueScheduler(qos: .background) // Background
```

### Memory Management
```swift
let disposeBag = DisposeBag()
subscription.disposed(by: disposeBag)
```

---

## Best Practices

1. **Always use DisposeBag** to prevent memory leaks
2. **Use Driver for UI bindings** - it's error-free and runs on main thread
3. **Use flatMap carefully** - it can create memory leaks with long-running observables
4. **Use weak self in closures** when capturing self
5. **Use BehaviorRelay instead of BehaviorSubject** for state management
6. **Test your reactive code** with RxTest
7. **Use appropriate schedulers** - background for heavy work, main for UI updates


## Tips
1. **SwiftUI** → use property wrappers + Combine
2. **UIKit** → use RxSwift + RxCocoa (old UIKit used delegate/protocols instead)
3. Mixed/complex → can mix SwiftUI with RxSwift, but rare   

## Resources

- [RxSwift GitHub Repository](https://github.com/ReactiveX/RxSwift)
- [RxSwift Documentation](https://github.com/ReactiveX/RxSwift/tree/main/Documentation)
- [RxMarbles - Interactive Diagrams](https://rxmarbles.com/)

## Contributing

Feel free to contribute to this cheat sheet by submitting pull requests with improvements, additional examples, or corrections.
