---
published: false
title: Singletons and Testing in Swift
layout: post
---

# Singletons and Testing in Swift

Singleton is a very popular design pattern on Apple's ecosystem, you probably had faced `Network.shared`, `UserDefaults.standards`, or another implementation of this pattern around your project, to be honest, I think you probably have a Singleton made by your team in your project and you are here looking for some solution that helps you to give testability. The good news: you gonna find it.

Before start talking about the solution, let's back to the unit testing theory: to write acceptable unit tests you should have control and visibility of inputs, outputs, and **states** of what you are testing, let me give a basic example:

// calc example

And now, what is the main goal of a singleton? To keep a unique and shared instance of something through the app's life cycle.

So, what is the problem with Singletons and Tests? The answer is related to a requirement of unit tests and a characteristic of singletons in general:

<img src="https://raw.githubusercontent.com/serralvo/serralvo.github.io/master/_posts/singletons-and-testing.jpg" />
