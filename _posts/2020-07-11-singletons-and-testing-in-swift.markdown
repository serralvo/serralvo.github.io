---
published: false
title: Singletons and Testing in Swift
layout: post
---

# Singletons and Testing in Swift

Singleton is a very popular design pattern on Apple's ecosystem. You probably have faced `Network.shared`, `UserDefaults.standards`, or other similar implementations around your project. To be honest, I think you probably have a Singleton made by your team in your project and you are here looking for some solution that helps you add testability to it. The good news is: you found it.

Before we start talking about the solution, let's go back to the unit testing theory: to write acceptable unit tests you should have control and visibility of **inputs**, **outputs**, and **states** of what you are testing. Let me give a basic example:

Imagine that we have a class that calculates how much discount customer will get, based on the price of the product:

```swift
struct Product {
    let price: Double
}

class DiscountCalculator {
    func calculate(with product: Product) -> Double {
        let price = product.price
        if price <= 100 {
            return product.price * 0.05
        } else {
            return product.price * 0.10
        }
    }
}

```

So, easy to test `DiscountCalculator`, right? We can inject the product, call `calculate(with product: Product)` function and check if result is expected:

```swift

private let sut = DiscountCalculator()

func test_calculate_withProductThatPriceIsHigherThan100_shouldReturn10PercentageDiscount() {
    let product = Product(price: 150)
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 15)
}

```

Perfect, for this example we have control of all **inputs** (`Product`) and visibility of **outputs** (the `Double` that is returned by `calculate` function). There's no **state** here for while. 

But now, imagine that we should give an extra discount if our customer is subscribed to a kind of rewards program and this information can change any time while the customer uses the app, so, and, for our luck, this information is stored into a Singleton.

And now I ask you: what's the main goal of a singleton? To keep a unique and shared instance of something throughout the app's lifecycle.

So, what's the problem with Singletons and Tests? The answer is related to a requirement of unit testing and a characteristic of singletons in general:

<img src="https://raw.githubusercontent.com/serralvo/serralvo.github.io/master/_posts/singletons-and-testing.jpg" />

