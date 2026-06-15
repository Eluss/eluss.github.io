---
layout: devlog
title: "TCA: Injecting Different Reducers Into the Same View"
category: devlog
tags: [swift, tca, ios, architecture]
---
Sometimes the same UI needs different logic depending on context — a form that fetches data in production but returns stubs in previews, or a screen that behaves differently in two flows. In TCA you can handle this by making the view generic over its reducer.

The trick is constraining the generic parameter so the `State` and `Action` types match, while letting the concrete reducer vary at the call site.

```swift
struct CounterView<R: Reducer>: View where R.State == CounterFeature.State,
                                            R.Action == CounterFeature.Action {
    @Bindable var store: Store<R.State, R.Action>

    var body: some View {
        VStack {
            Text("\(store.count)")
            Button("+") { store.send(.increment) }
        }
    }
}
```

Now define two reducers with the same `State`/`Action` but different logic:

```swift
@Reducer
struct CounterFeature {
    struct State: Equatable { var count = 0 }
    enum Action { case increment }

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case .increment:
                state.count += 1
                return .none
            }
        }
    }
}

@Reducer
struct PreviewCounterFeature {
    typealias State = CounterFeature.State
    typealias Action = CounterFeature.Action

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            state.count += 10
            return .none
        }
    }
}
```

Inject whichever reducer the context needs:

```swift
// Production
CounterView(store: Store(initialState: .init()) { CounterFeature() })

// Preview / test
CounterView(store: Store(initialState: .init()) { PreviewCounterFeature() })
```

The view has no knowledge of which reducer it received. The generic constraint guarantees both reducers speak the same `State` and `Action`, so the view compiles against either without change.

**Takeaway:** make a TCA view generic over `R: Reducer` with `where` constraints on `State` and `Action` to decouple the view from any specific reducer implementation.
