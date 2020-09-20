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
func allProductsHasValueGreaterThan50(_ products: [Product]) -> Bool {
   var allProductsHasValueGreaterThan50 = true

   products.forEach {
       if $0.price > 50 {
           allProductsHasValueGreaterThan50 = true 
       } else {
           allProductsHasValueGreaterThan50 = false
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

let result = allProductsHasValueGreaterThan50(products)
``` 

What is the value of `result`? It'll be `true`, what is strange beacause the `value` of 3th Product is 40. Here we have an error on the implementation, we have many ways to fix that, look the example: 


