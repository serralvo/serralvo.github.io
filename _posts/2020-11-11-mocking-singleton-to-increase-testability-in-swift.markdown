---
published: true
title: Mocking Singleton to increase testability in Swift
reading-time: 12
layout: post
---

> This is my first post that is part of a series about Singletons and Tests in Swift. In this one, I'll show how to make parts of your system that use a Singleton being testable. 

Singleton is a very popular design pattern on Apple's ecosystem. You probably have faced `Network.shared`, `UserDefaults.standard`, or other similar implementations around your project. To be honest, I think you probably have a Singleton made by your team in your project and you are here looking for some solution that helps you add testability to it. The good news is: you found it.

Before we start talking about the solution, let's go back to the unit testing theory: to write acceptable unit tests you should have control and visibility of **inputs**, **outputs**, and **states** of what you are testing. Let me give a basic example:

Imagine that we have a scene that displays how much discount a customer will get on checkout, based on the price of the product:

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

But now, imagine that we should give an extra discount if our customer is subscribed to a kind of loyalty program and this information can change any time while the customer is using the app. For our luck (actually just for the purposes of this article), this information is stored in a Singleton.

```swift
enum LoyaltyStatus {
    case subscribed
    case unsubscribed
}

class CustomerManager {

    static var shared: CustomerManager = .init()

    private init() {}
    
    private var customerLoyaltyStatus: LoyaltyStatus?
    
    func setCustomerLoyaltyStatus(_ status: LoyaltyStatus) {
        customerLoyaltyStatus = status
    }

    func getLoyaltyStatus() -> LoyaltyStatus {
        return customerLoyaltyStatus ?? .unsubscribed
    }
}

``` 

So, what can we do? We can use the `CustomerManager` on `DiscountCalculator` to give the extra discount and everything is gonna be ok:

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
        
        let customerLoyaltyStatus = CustomerManager.shared.getLoyaltyStatus()
        switch customerLoyaltyStatus {
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

What is the main goal of a singleton? To keep a unique and shared instance of something throughout the app's lifecycle. However, that goes against the theory of writing acceptable units tests, where we must have control of the **states**. Let's check how we can update the tests of `CheckoutInteractor`:

```swift
private let sut = CheckoutInteractor()

func test_calculate_withProductThatPriceIsHigherThan100_andCustomerIsSubscribed_shouldReturn20PercentageDiscount() {
    let product = Product(price: 200)
    CustomerManager.shared.setCustomerLoyaltyStatus(.subscribed)
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 40)
}

```

Ok, and what happens if we want to test the unsubscribed scenario? 

```swift
private let sut = CheckoutInteractor()

func test_calculate_withProductThatPriceIsHigherThan100_andCustomerIsUnsubscribed_shouldReturn10PercentageDiscount() {
    let product = Product(price: 200)
    CustomerManager.shared.setCustomerLoyaltyStatus(.unsubscribed)
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 20)
}
```

Isn't this fine? No, my friend. In these tests we **don't have control of all states**. This kind of test is fragile like an eggshell. For example, what happens if another test tries to get or set the customer loyalty program status? How could we guarantee the expected state? Remember: if you run your tests in a random order, it won't be possible guarantee the _orchestrated state_. Besides that, this kind of test does not follow F.I.R.S.T. principles¹ because they are not isolated/independent.

--- 

Finally, what is the problem with Singletons and Tests? The answer is related to a requirement of unit testing and a characteristic of singletons in general:

<img src="https://raw.githubusercontent.com/serralvo/serralvo.github.io/master/_posts/singletons-and-testing.jpg" />

My point now is: how to get back the control of all states that the test needs? The answer is a **mix of two techniques**: Dependency Injection and Dependency Inversion Principle:

#### Dependency Injection

This technique² helps us to take control of inputs of our system, thus ensuring testability. 

#### Dependency Inversion Principle

This one is part of S.O.L.I.D. Principles³. Long story short: the goal is to make your system depend on abstractions (protocols) instead of concrete elements. The use of interfaces allows us to mock parts of our system easily.

--- 

Now we should update the `CheckoutInteractor` to allow more testability, but first, let's change the `CustomerManager` to apply the Dependency Inversion Principle:

```swift
protocol CustomerManagerProtocol {
    func setCustomerLoyaltyStatus(_ status: LoyaltyStatus)
    func getLoyaltyStatus() -> LoyaltyStatus
}

class CustomerManager {

    static var shared: CustomerManager = .init()

    private init() {}
    
    private var customerLoyaltyStatus: LoyaltyStatus?
}

extension CustomerManager: CustomerManagerProtocol {

    func setCustomerLoyaltyStatus(_ status: LoyaltyStatus) {
        customerLoyaltyStatus = status
    }

    func getLoyaltyStatus() -> LoyaltyStatus {
        return customerLoyaltyStatus ?? .unsubscribed
    }
}

``` 

Here we are implementing the ideas of **Dependency Inversion Principle**: we've created an interface and our code will now use `CustomerManagerProtocol` (the abstraction) instead of `CustomerManager` (a concrete implementation), this will help us to take back the **control of states**. The second step is to follow the **Dependency Injection** concept and inject the manager to **control all inputs**:

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
        
        let customerLoyaltyStatus = self.customerManager.getLoyaltyStatus()
        switch customerLoyaltyStatus {
        case .subscribed:
            return discountByPrice + (price * 0.10)
        case .unsubscribed:
            return discountByPrice
        }
    }
}

```

As you can see, our singleton `CustomerManager` implements `CustomerManagerProtocol`, so now we can pass it to `CheckoutInteractor` initializer:

```swift 
let interactor = CheckoutInteractor(withCustomerManager: CustomerManager.shared)
``` 
If you want more convenience you can set `CustomerManager.shared` as the default implementation, but **be careful** with this approach, if you forget to pass a mock, the tests will use the concrete instance (`CustomerManager.shared`) and this could generate a flaky test.

```swift
class CheckoutInteractor {

    private let customerManager: CustomerManagerProtocol

    init(withCustomerManager manager: CustomerManagerProtocol = CustomerManager.shared) {
        self.customerManager = manager
    }
}
``` 

--- 

We've updated our code to be able to **inject** and **mock** whatever we need. Now we have control to check every scenario and test it:

```swift
class CustomerManagerMock: CustomerManagerProtocol {

    var loyaltyStatusToBeReturned: LoyaltyStatus?

    func setCustomerLoyaltyStatus(_ status: LoyaltyStatus) {}

    func getLoyaltyStatus() -> LoyaltyStatus {
        return loyaltyStatusToBeReturned ?? .unsubscribed
    }
}

private let customerManagerMock = CustomerManagerMock()
private lazy var sut = CheckoutInteractor(withCustomerManager: customerManagerMock)

func test_calculate_withProductThatPriceIsHigherThan100_andCustomerIsSubscribed_shouldReturn20PercentageDiscount() {
    let product = Product(price: 200)
    customerManagerMock.loyaltyStatusToBeReturned = .subscribed
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 40)
}

func test_calculate_withProductThatPriceIsHigherThan100_andCustomerIsUnsubscribed_shouldReturn10PercentageDiscount() {
    let product = Product(price: 200)
    customerManagerMock.loyaltyStatusToBeReturned = .unsubscribed
    
    let discount = sut.calculate(with: product)
    
    XCTAssertEqual(discount, 20)
}

```

### Conclusion 

The technique presented here is really useful and will help you test parts of your system that **depend on singletons**. Going further, you can also consider the idea of creating just one mock for each singleton and keeping it accessible in the whole project, avoiding multiples implementations for the same necessity.

### References
1. <a href="http://agileinaflash.blogspot.com/2009/02/first.html" target="_blank">F.I.R.S.T. Principles</a>
2. <a href="https://en.wikipedia.org/wiki/Dependency_injection" target="_blank">Dependency Injection</a>
3. <a href="https://en.wikipedia.org/wiki/SOLID" target="_blank">S.O.L.I.D. Principles</a>
