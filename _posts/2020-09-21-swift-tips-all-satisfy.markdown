---
published: true
title: Swift Tips - allSatisfy 
layout: post
---

# Swift Tips: allSatisfy 

> Reading time: 2 minutes

A few days ago, I faced the following issue: We have a method that should return if all items of an array have some specific characteristic, for example:

```swift
struct Product {
   let price: Double
}

func allProductsHasValueGreaterThan50(_ products: [Product]) -> Bool {
   // ...
}
```

The first solution was pretty simple:

```swift
func allProductsHasValueGreaterThan50(_ products: [Product]) -> Bool {
   var allProductsHasValueGreaterThan50 = true

   products.forEach {
       if $0.price <= 50 {
           allProductsHasValueGreaterThan50 = false
           return
       }
   }

   return allProductsHasValueGreaterThan50
}
```

Another way to do that is using a `filter` and check if filtered array and `products` has the same size: 

```swift
func allProductsHasValueGreaterThan50(_ products: [Product]) -> Bool {
   let productsThatPriceIsGreaterThan50 = products.filter { $0.price > 50 }
   return products.count == productsThatPriceIsGreaterThan50.count
}
```

That one ☝️ will work like expected, but we can simplify using `allSatisfy`:

```swift
func allProductsHasValueGreaterThan50(_ products: [Product]) -> Bool {
   return products.allSatisfy { $0.price > 50 }
}
```

## Conclusion 

`allSatisfy` method is a capability of `Sequence` that is available since Swift 5.1, the syntax is pretty clear and will help us to remove all methods like I have presented here above to solve this kind of issue.

## References 

[Apple Documentation](https://developer.apple.com/documentation/swift/array/2994715-allsatisfy)
