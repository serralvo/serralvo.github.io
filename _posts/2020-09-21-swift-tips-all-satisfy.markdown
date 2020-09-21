---
published: true
title: Swift Tips - allSatisfy 
layout: post
---

# Swift Tips: allSatisfy 

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
       if $0.price > 50 {
           allProductsHasValueGreaterThan50 = true 
       } else {
           allProductsHasValueGreaterThan50 = false
       }
   }

   return allProductsHasValueGreaterThan50
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

What is the value of `result`? It'll be `true`, what is strange because the `value` of 3th Product is `40`. Here we have an error on the implementation and there many ways to fix, for example:

```swift
func allProductsHasValueGreaterThan50(_ products: [Product]) -> Bool {
   let productsWherePriceIsGreaterThan50 = products.filter { $0.price > 50 }
   return products.count == productsWherePriceIsGreaterThan50.count
}
```

That one ☝️ will work like expected, but we can simplify using `allSatisfy`:

```swift
func allProductsHasValueGreaterThan50(_ products: [Product]) -> Bool {
   return products.allSatisfy { $0.price > 50 }
}
```

## Conclusion 
