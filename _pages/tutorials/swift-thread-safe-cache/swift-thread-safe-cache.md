---
title: Build a Generic Thread-Safe Cache in Swift
permalink: /swift-stack/
---

## Introduction
In this tutorial, we'll build a generic, thread-safe cache in Swift and use it to create an asynchronous cache for downloading and storing images from remote URLs. Before we start building our cache, let's take a moment to understand some of the key components we'll be using.

## Understanding `DispatchQueue`
`DispatchQueue` is a class built on top of the [Grand Central Dispatch (GCD)](https://developer.apple.com/documentation/DISPATCH) system, and used for managing the concurrent and serial execution of programming tasks. This enables developers to improve the speed and efficiency of their apps by distributing work across multiple processing threads, and synchronizing the end result.

### Defining a DispatchQueue
A `DispatchQueue` can be initialized with a **label** and **attributes**. 

```
let queue = DispatchQueue(label: "com.example.myqueue")
```
- **Label:** a unique string which can be useful for debugging and identification.

### DispatchQueue Attributes
When creating a `DispatchQueue`, you can specify its attributes:
- **Serial Queue (Default):** Tasks are executed in a *first-in -> first-out* order, one at a time.
    ```
    let queue = DispatchQueue(label: "com.example.serialQueue")
    ```
- **Concurrent Queue:** Tasks are executed concurrently, or in parallel.
    ```
    let queue = DispatchQueue(label: "com.example.concurrentQueue", attributes: .concurrent)
    ```  

### Synchronization Methods
- **Synchronous Execution (`sync`):** The current thread waits until the task completes before continuing.
- **Asynchronous Execution (`async`):** The task runs in the background, and the current thread continues execution.

Example usage:
```
queue.sync {
    print("This runs synchronously")
}

queue.async {
    print("This runs asynchronously")
}
```

### Barrier Flags for Exclusive Access
As part of our read and write safety, we need to ensure that we don't try to access an element in our cache while it is being modified. To do that we can use the `.barrier` flag on asynchronous operations to guarantee exclusive access to our data during write operations.
```
queue.async(flags: .barrier) {
    print("This task runs exclusively on the queue")
}
```

## Understanding NSCache
`NSCache` is a special type of mutable collection that can be used for storing key/value pairs, similar to a [Dictionary](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes#Dictionaries). However, unlike a regular dictionary, `NSCache` automatically evicts its contents when system memory is low.

### Other Key Features of `NSCache`
- **Thread safety:** `NSCache` is inherently thread-safe for read and write operations. 
- **Cost-based eviction:** You can optionally assign the `cost` of an item when storing it in cache, and a [`totalCostLimit`](https://developer.apple.com/documentation/foundation/nscache/1407672-totalcostlimit). Eviction will start either when system memory resources are low, or when the cache reaches the `totalCostLimit`.
- **Delegate Support:** `NSCache` provides a [delegate](https://developer.apple.com/documentation/foundation/nscachedelegate) to handle object eviction. The delegate provides a method for taking some action when an object will be evicted from the cache.

## Creating a Generic Thread-Safe Cache
Building on the previous sections, we'll incorporate `DispatchQueue` and `NSCache` to create our `Cache` class.

```
class Cache<Key: Hashable & Sendable, Value: Sendable> {
    // 1
    private var cache = NSCache<WrappedKey, WrappedValue>()
    private let queue = DispatchQueue(label: "com.cache.queue", attributes: .concurrent)
    
    // 2
    func getValue(for key: Key) -> Value? {
        queue.sync {
            cache.object(forKey: WrappedKey(key: key))?.value
        }
    }
    
    // 3
    func set(_ value: Value, for key: Key) {
        queue.async(flags: .barrier) { [self] in
            cache.setObject(WrappedValue(value: value), forKey: WrappedKey(key: key))
        }
    }
    
    // 4
    private class WrappedKey: NSObject {
        let key: Key
        
        init(key: Key) {
            self.key = key
        }
        
        override var hash: Int {
            key.hashValue
        }
        
        override func isEqual(_ object: Any?) -> Bool {
            guard let other = object as? WrappedKey else { return false }
            return other.key == key
        }
    }
    
    private class WrappedValue {
        let value: Value
        init(value: Value) {
            self.value = value
        }
    }
}
```

**Step 1:** Create private instances of our cache using and dispatch queue as described in the previous sections.

**Step 2:** Provide a method for reading a value from cache, based on a given key. We use the queue's `.sync` method here because, in general, reading from the cache is a quick operation, and does not modify shared state.

**Step 3**: Provide a method for writing a value to the cache, and assigning it a key. We use the queue's `.async` method with `.barrier` flag because we want to ensure that writes occur sequentially and do not block read operations from proceeding.

**Step 4**: We use a wrapper class for our key and values to ensure proper key comparison and compatibility with `NSCache`.

## Creating an Async Image Cache
We can build on top of the generic cache we created in the previous section to create a new cache for loading remote images either from cache or a URL.

```
public class AsyncImageCache {
    // 1
    private let cache = Cache<URL, UIImage>()
    
    public init () {}
    
    // 2
    public func loadImage(from url: URL) async throws -> UIImage {
        // 3
        if let cachedImage = cache.getValue(for: url) {
            return cachedImage
        }
        // 4
        let (data, _) = try await URLSession.shared.data(from: url)
        guard let image = UIImage(data: data) else {
            throw AsyncImageCacheError.imageConversionError
        }
        // 5
        cache.set(image, for: url)
        // 6
        return image
    }
}

enum AsyncImageCacheError: Error {
    case imageConversionError
}
```

**Step 1:** Initialize the cache storage using the generic `Cache` class, setting the `Key` type to  `URL`, and `Value` type to `UIImage`.

**Step 2:** Provide an asynchronous method for loading an `UIImage` from a `URL`. This method can also throw an error in case there is a problem retrieving the image. 

**Step 3:** Search the cache for the requested image. If it is found, return it.

**Step 4:** If the image is not found in cache, use the shared networking session to fetch the image data from its source url. Attempt to convert the returned image data into an `UIImage` type, and throw an error if the conversion fails.

**Step 5:** Write the converted `UIImage` value to cache, using its `URL` as the key.

**Step 6:** Return the image.

## Loading an Image with Image Cache
Now that we've built an asynchronous image cache on top of our generic, thread-safe cache, let's load an image.

```
// 1
let imageCache = AsyncImageCache()

// 2
let url = URL(string: "https://bryantm1123.github.io/_pages/tutorials/swift-stack/pancake-stack.jpeg")!

// 3
Task {
    do {
        // 4
        let image = try await imageCache.loadImage(from: url)
        print("Image loaded successfully!")
    } catch {
        print("Failed to load image: \(error)")
    }
}
```

**Step 1:** Initialize an instance of the `AsyncImageCache` we created in the previous section.

**Step 2:** Provide a `URL` object to represent the source location of our desired image.

**Step 3:** Provide a task for calling the asynchronous `loadImage` method.

**Step 4:** Try loading the image from the `AsyncImageCache`. If successful, print that the image loaded successfully. If an error is thrown, catch and print it.