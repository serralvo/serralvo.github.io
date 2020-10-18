---
published: true
title: Swift Tips - allSatisfy 
layout: post
---

This article will take about 3 minutes to read.

A few days ago, I faced the following issue: We have a method that should return if all items of an array have some specific characteristic, for example:

```swift
struct Product {
   let price: Double
}

func allProductsHaveValueGreaterThan50(_ products: [Product]) -> Bool {
   // ...
}
```

The first solution was pretty simple:

```swift
func allProductsHaveValueGreaterThan50(_ products: [Product]) -> Bool {
   guard products.isEmpty == false else { return false }

   var allProductsHaveValueGreaterThan50 = true

   products.forEach {
       if $0.price <= 50 {
           allProductsHaveValueGreaterThan50 = false
           return
       }
   }

   return allProductsHaveValueGreaterThan50
}
```

Another way to do that is using a `filter` and check if filtered array and `products` has the same size: 

```swift
func allProductsHaveValueGreaterThan50(_ products: [Product]) -> Bool {
   guard products.isEmpty == false else { return false }
   
   let productsThatPriceIsGreaterThan50 = products.filter { $0.price > 50 }
   
   return products.count == productsThatPriceIsGreaterThan50.count
}
```

That one ☝️ will work like expected, but we can simplify using `allSatisfy`:

```swift
func allProductsHaveValueGreaterThan50(_ products: [Product]) -> Bool {
   guard products.isEmpty == false else { return false }

   return products.allSatisfy { $0.price > 50 }
}
```

## Conclusion and some thoughts

`allSatisfy` method is a capability of `Array` that is available since Swift 5.1, the syntax is pretty clear and will help us to remove all methods like I have presented here to solve this kind of issue. If you don't want to check array size every time, you can create an extension to encapsulate this rule:

```swift
extension Array {
   func allSatisfyAndIsNotEmpty(_ predicate: (Element) throws -> Bool) rethrows -> Bool {
      guard self.isEmpty == false else { return false }
      return try self.allSatisfy(predicate)
   }
}

func allProductsHaveValueGreaterThan50(_ products: [Product]) -> Bool {
   return products.allSatisfyAndIsNotEmpty { $0.price > 50 }
}

``` 

## References 

[Apple Documentation](https://developer.apple.com/documentation/swift/array/2994715-allsatisfy)
