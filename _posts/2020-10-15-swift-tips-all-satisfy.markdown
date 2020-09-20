---
published: true
title: Swift Tips - allSatisfy 
layout: post
---

# Swift Tips: allSatisfy 

Few days ago I faced the following issue: We have a method that should return if all items of an array has some specific characteristic, for example:

```swift
struct Product {
   let price: Double
}

func allProductsAreGreaterThan50(_ products: [Product]) -> Bool {
   // ...
}
```

The first solution was pretty simple:

```swift
func allProductsAreGreaterThan50(_ products: [Product]) -> Bool {
   var allProductsAreGreaterThan50 = true

   products.forEach {
       if $0.price > 50 {
           allProductsAreGreaterThan50 = true 
       } else {
           allProductsAreGreaterThan50 = false
       }
   }

   return allProductsAreGreaterThan50
}
```

🤔 What is the problem of this implementation? Imagine if we call `allProductsAreGreaterThan50()` with these parameters:

```swift
let products = [
   Product(price: 70),
   Product(price: 80),
   Product(price: 40),
   Product(price: 60)
]

let result = allProductsAreGreaterThan50(products)
``` 

What is the value of `result`? 
