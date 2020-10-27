---
published: true
title: Mocking Singleton to increase testability in Swift
layout: post
---

This article will take about 10 minutes to read.

> This is my first post that is part of a series about Singletons and Tests in Swift. In this one, I'll show how to make parts of your system that use a Singleton being testable. 

Singleton is a very popular design pattern on Apple's ecosystem. You probably have faced `Network.shared`, `UserDefaults.standard`, or other similar implementations around your project. To be honest, I think you probably have a Singleton made by your team in your project and you are here looking for some solution that helps you add testability to it. The good news is: you found it.

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

So, easy to test `CheckoutInteractor`, right? We can inject the product, call `calculate(with product: Product)` function and check if the result is expected:

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

    static var shared: CustomerManager = .init()

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

### Singletons

What is the main goal of a singleton? To keep a unique and shared instance of something throughout the app's lifecycle. So, here we have a big issue about testability, do you remember the theory about having control of the **states** to write acceptable unit tests, right? Let's check how we can update the tests of `CheckoutInteractor`:

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

Works fine? No my friend, for these tests we **don't have control of all states**, this kind of test is fragile like an eggshell, for example, what happens if another test tries to get the customer reward or set it? How could we guarantee the expected state? Remember, if you run your tests in randomic order won't be possible guarantee the _orchestrated state_, besides that, this kind of test does not follow F.I.R.S.T principles¹ because they are not isolated / independent.

--- 

Finally, what is the problem with Singletons and Tests? The answer is related to a requirement of unit testing and a characteristic of singletons in general:

<img src="https://raw.githubusercontent.com/serralvo/serralvo.github.io/master/_posts/singletons-and-testing.jpg" />

My point now is: how to get back the control of all states that test needs? The answer is a **mix of two techniques**: Dependency Injection and Dependency Inversion Principle:

#### Dependency Injection

This technique² helps us to take control of inputs of our system, thus ensuring testability. 

#### Dependency Inversion Principle

This one is part of SOLID Principles³, in a short way the goal is to make your system depends on abstractions (protocols) instead of concreates. Using interfaces allow us to mock parts of our system easily.

--- 

Now we should update the `CheckoutInteractor` to give more testability, but first, let's change the `CustomerManager` applying the Dependency Inversion Principle:

```swift
protocol CustomerManagerProtocol {
    func setCustomerReward(_ reward: Reward)
    func getReward() -> Reward
}

class CustomerManager {

    static var shared: CustomerManager = {
        let manager = CustomerManager()
        return manager
    }()

    private init() {}
    
    private var customerReward: Reward?
}

extension CustomerManager: CustomerManagerProtocol {

    func setCustomerReward(_ reward: Reward) {
        customerReward = reward
    }

    func getReward() -> Reward {
        return customerReward ?? Reward.unsubscribed
    }
}

``` 

Here we are implementing the ideas of **Dependency Inversion Principle**, we've created an interface and whole our code will use `CustomerManagerProtocol` (the abstraction) instead `CustomerManager` (the concrete), this will help us to take back the **control of states**. The second step is to follow the **Dependency Injection** concept and inject the manager to **control all inputs**, take a look: 

```swift
class CheckoutInteractor {

    private let customerManager: CustomerManagerProtocol

    init(withCustomerManager manager: CustomerManagerProtocol) {
        self.customerManager = manager
    }

    func calculate(with product: Product) -> Double {
        let price = product.price
        let discountByPrice: Double
        
        if price <= 100 {
            discountByPrice = price * 0.05
        } else {
            discountByPrice = price * 0.10
        }
        
        let customerReward = self.customerManager.getReward()        
        switch customerReward {
        case .subscribed:
            return discountByPrice + (price * 0.10)
        case .unsubscribed:
            return discountByPrice
        }
    }
}

```

Look, our singleton `CustomerManager` implements `CustomerManagerProtocol`, right? Now we can pass it on `CheckoutInteractor` initializer:

```swift 
let interactor = CheckoutInteractor(withCustomerManager: CustomerManager.shared)
``` 
Tip 💡 If you want more convenience you can set `CustomerManager.shared` as default implementation:

```swift
class CheckoutInteractor {

    private let customerManager: CustomerManagerProtocol

    init(withCustomerManager manager: CustomerManagerProtocol = CustomerManager.shared) {
        self.customerManager = manager
    }
}
``` 

--- 

We've updated our code to be able to **inject** and **mock** whatever we need, now we have control to check every scenario and test it:

```swift
class CustomerManagerMock: CustomerManagerProtocol {

    var rewardToBeReturned: Reward?

    func setCustomerReward(_ reward: Reward) {}

    func getReward() -> Reward {
        return rewardToBeReturned ?? Reward.unsubscribed
    }

}

private let customerManagerMock = CustomerManagerMock()
private lazy var sut = CheckoutInteractor(withCustomerManager: customerManagerMock)

func test_calculate_withProductThatPriceIsHigherThan100_andCustomerIsSubscribed_shouldReturn20PercentageDiscount() {
    let product = Product(price: 200)
    customerManagerMock.rewardToBeReturned = Reward.subscribed
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 40)
}

func test_calculate_withProductThatPriceIsHigherThan100_andCustomerIsUnsubscribed_shouldReturn10PercentageDiscount() {
    let product = Product(price: 200)
    customerManagerMock.rewardToBeReturned = Reward.unsubscribed
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 20)
}

```

### Conclusion 

The technique presented here is really useful and will help you to test parts of your system that **depends on singletons**. A pro tip is related to mocks in general, consider the idea of creating just one mock for each singleton and keep it accessible for the whole project, avoiding multiples implementations for the same necessity.

### References
1. <a href="http://agileinaflash.blogspot.com/2009/02/first.html" target="_blank">F.I.R.S.T Principles</a>
2. <a href="https://en.wikipedia.org/wiki/Dependency_injection" target="_blank">Dependency Injection</a>
3. <a href="https://en.wikipedia.org/wiki/SOLID" target="_blank">SOLID Principles</a>
