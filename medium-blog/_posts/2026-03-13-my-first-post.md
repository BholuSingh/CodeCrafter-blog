---
layout: post
title: "Swift Concurrency: A Practical Guide to async/await, Tasks, and Actors"
date: 2026-03-13
categories: [swift, ios, concurrency]
---

Swift Concurrency provides a structured approach to writing asynchronous code that is both readable and safe. Introduced in Swift 5.5 and available from iOS 15, watchOS 8, and Xcode 14 onwards, it replaces completion handler patterns with language-level constructs that the compiler can verify at compile time.

## Defining Asynchronous Functions

An asynchronous function is declared by adding the `async` keyword after its parameter list. When you call an async function, you use the `await` keyword to mark the suspension point where execution may pause until the operation completes.

```swift
func fetchParticipants() async -> [Participant] {
    // Async work here
}
```

If an async function can throw errors, you combine `async` with `throws` and call it using `try await`:

```swift
do {
    try await store.save(caffeineSample)
    logger.debug("Drink saved to HealthKit")
} catch {
    logger.error("Unable to save: \(error.localizedDescription)")
}
```

This replaces the older pattern of handling errors through `Error?` parameters in completion handlers.

## Tasks: Creating Asynchronous Contexts

A `Task` is a unit of asynchronous work that starts running immediately after creation. You use tasks when you need to call async functions from synchronous contexts, such as button actions or lifecycle methods.

According to Apple's documentation: "When you create an instance of `Task`, you provide a closure that contains the work for that task to perform. Tasks can start running immediately after creation; you don't explicitly start or schedule them."

```swift
case let backgroundTask as WKApplicationRefreshBackgroundTask:
    Task {
        let model = CoffeeData.shared
        let success = await model.healthKitController.loadNewDataFromHealthKit()
        
        if success {
            scheduleBackgroundRefreshTasks()
            logger.debug("Background Task Completed Successfully!")
        }
        
        backgroundTask.setTaskCompletedWithSnapshot(success)
    }
```

Tasks support cancellation through `Task.checkCancellation()` and `Task.isCancelled`. It's your responsibility to check for cancellation at appropriate points and respond accordingly.

## Actors: Protecting Shared Mutable State

When you need thread-safe access to properties across asynchronous boundaries, convert your `class` to an `actor`:

```swift
actor HealthKitController {
    var anchor: HKQueryAnchor?
    
    func loadNewDataFromHealthKit() async -> Bool {
        // Actor-isolated code here
    }
}
```

The compiler enforces that all access to an actor's properties must go through the actor, preventing data races at compile time.

## MainActor: UI Thread Safety

For types that interact with SwiftUI views, apply `@MainActor` to ensure all property updates happen on the main thread:

```swift
@MainActor
class CoffeeData: ObservableObject {
    @Published private(set) var currentDrinks: [Drink] = []
    
    func drinksUpdated() async {
        await coffeeDataStore.save(currentDrinks)
    }
}
```

SwiftUI views using `@EnvironmentObject` with `@MainActor` types don't require explicit `await` calls since they're already on the main actor.

## Bridging Completion Handlers with Continuations

When an SDK still requires completion handlers, use `withCheckedThrowingContinuation` to bridge to async/await:

```swift
private func queryHealthKit() async throws -> ([HKSample]?, [HKDeletedObject]?, HKQueryAnchor?) {
    return try await withCheckedThrowingContinuation { continuation in
        let query = HKAnchoredObjectQuery(
            type: caffeineType,
            predicate: datePredicate,
            anchor: anchor,
            limit: HKObjectQueryNoLimit
        ) { _, samples, deletedSamples, newAnchor, error in
            if let error = error {
                continuation.resume(throwing: error)
            } else {
                continuation.resume(returning: (samples, deletedSamples, newAnchor))
            }
        }
        store.execute(query)
    }
}
```

The continuation must be resumed exactly once. The "checked" variant will trap at runtime if you resume more than once or fail to resume at all.

## Summary

Swift Concurrency fundamentally changes how we write asynchronous code:

- **async/await** makes asynchronous code read sequentially, eliminating callback pyramids
- **Task** creates asynchronous contexts from synchronous code and supports cancellation
- **actor** protects shared mutable state with compile-time verification
- **@MainActor** guarantees UI updates happen on the main thread
- **Continuations** bridge existing completion handler APIs to async/await

The compiler catches concurrency bugs at build time rather than runtime, making your code safer and more maintainable.

---

*References: [Apple Developer Documentation - Swift Concurrency](https://developer.apple.com/documentation/swift/concurrency/), [Updating an App to Use Swift Concurrency](https://developer.apple.com/documentation/swift/updating_an_app_to_use_swift_concurrency)*
