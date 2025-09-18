# RxSwift: `bind` vs `subscribe`

## 🔹 Using `subscribe`
```swift
userNameTf.rx.text.orEmpty
    .subscribe(onNext: { text in
        myLabel.text = text
    })
    .disposed(by: disposeBag)
```

✅ Gives you **full control**.  
❌ You must **manually assign** values to the UI.  
❌ You need to ensure updates happen on the **main thread**.

## 🔹 Using `bind`
```swift
userNameTf.rx.text.orEmpty
    .bind(to: myLabel.rx.text)
    .disposed(by: disposeBag)
```

✅ **Shorter, safer, cleaner**.  
✅ Automatically updates on the **main thread**.  
✅ **Auto-disposes** when the UI control is deallocated.  
❌ Less flexible (no `onError`, `onCompleted`).

## ⚡️ Summary
* Use `subscribe` when you need **custom logic** inside `onNext`, `onError`, or `onCompleted`.
* Use `bind` when you just want to **connect an Observable to a UI property**.
