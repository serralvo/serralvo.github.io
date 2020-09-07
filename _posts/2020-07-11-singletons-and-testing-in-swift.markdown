---
published: false
title: Singletons and Testing in Swift
layout: post
---

# Singletons and Testing in Swift

Singleton is a very popular design pattern on Apple's ecosystem. You probably have faced `Network.shared`, `UserDefaults.standards`, or other similar implementations around your project. To be honest, I think you probably have a Singleton made by your team in your project and you are here looking for some solution that helps you add testability to it. The good news is: you found it.

Before we start talking about the solution, let's go back to the unit testing theory: to write acceptable unit tests you should have control and visibility of **inputs**, **outputs**, and **states** of what you are testing. Let me give a basic example:

Imagine that we have a scene that displays how much discount customer will get on checkout, based on the price of the product:

```swift
struct Product {
    let price: Double
}

class CheckoutInteractor {
    func calculate(with product: Product) -> Double {
        let price = product.price
        if price <= 100 {
            return price * 0.05
        } else {
            return price * 0.10
        }
    }
}

```

So, easy to test `CheckoutInteractor`, right? We can inject the product, call `calculate(with product: Product)` function and check if result is expected:

```swift

private let sut = CheckoutInteractor()

func test_calculate_withProductThatPriceIsHigherThan100_shouldReturn10PercentageDiscount() {
    let product = Product(price: 150)
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 15)
}

```

Perfect, for this example we have control of all **inputs** (`Product`) and visibility of **outputs** (the `Double` that is returned by `calculate` function). There's no **state** here for while. 

But now, imagine that we should give an extra discount if our customer is subscribed to a kind of rewards program and this information can change any time while the customer uses the app, so, and, for our luck, this information is stored into a Singleton.

```swift

enum Reward {
    case subscribed
    case unsubscribed
}

class CustomerManager {

    static var shared: CustomerManager = {
        let manager = CustomerManager()
        return manager
    }()

    private init() {}
    
    private var customerReward: Reward?
    
    func setCustomerReward(_ reward: Reward) {
        customerReward = reward
    }

    func getReward() -> Reward {
        return customerReward ?? Reward.unsubscribed
    }
}

``` 

So, what we can do? We can use the `CustomerManager` on `DiscountCalculator` to give the extra discount and everything is gonna be ok:

```swift

class CheckoutInteractor {
    func calculate(with product: Product) -> Double {
        let price = product.price
        let discountByPrice: Double
        
        if price <= 100 {
            discountByPrice = price * 0.05
        } else {
            discountByPrice = price * 0.10
        }
        
        let customerReward = CustomerManager.shared.getReward()        
        switch customerReward {
        case .subscribed:
            return discountByPrice + (price * 0.10)
        case .unsubscribed:
            return discountByPrice
        }
    }
}

```
Perfect 🎉, this works fine and our customers will be happy with these discounts! 🤑

---

Now, time to talk about Singletons! What's the main goal of a singleton? To keep a unique and shared instance of something throughout the app's lifecycle. So, here we have a big issue about testability, do you remember the point about have control of **state** to write aceptable unit tests, right? Let's check how we can update the tests of `CheckoutInteractor`:

```swift

private let sut = CheckoutInteractor()

func test_calculate_withProductThatPriceIsHigherThan100_andCustomerIsSubscribed_shouldReturn20PercentageDiscount() {
    let product = Product(price: 200)
    CustomerManager.shared.setCustomerReward(Reward.subscribed)
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 40)
}

```

Ok, and what happens if we want to test the unsubscribed scenario? 

```swift

private let sut = CheckoutInteractor()

func test_calculate_withProductThatPriceIsHigherThan100_andCustomerIsUnsubscribed_shouldReturn10PercentageDiscount() {
    let product = Product(price: 200)
    CustomerManager.shared.setCustomerReward(Reward.unsubscribed)
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 20)
}
```

Works fine? No my friend, for these tests we **don't have control of all states** and this kind of test is fragile like an eggshell.

--- 

So, what's the problem with Singletons and Tests? The answer is related to a requirement of unit testing and a characteristic of singletons in general:

<img src="https://raw.githubusercontent.com/serralvo/serralvo.github.io/master/_posts/singletons-and-testing.jpg" />

My point now is: how to get back the control of all states that test needs? The answer is a **mix of two techniques**: Dependency Injection and Dependency Inversion Principle: 


