---
published: false
title: Singletons and Testing in Swift
layout: post
---

# Singletons and Testing in Swift

Singleton is a very popular design pattern on Apple's ecosystem. You probably have faced `Network.shared`, `UserDefaults.standards`, or other similar implementations around your project. To be honest, I think you probably have a Singleton made by your team in your project and you are here looking for some solution that helps you add testability to it. The good news is: you found it.

Before we start talking about the solution, let's go back to the unit testing theory: to write acceptable unit tests you should have control and visibility of inputs, outputs, and **states** of what you are testing. Let me give a basic example:

// basic example

And now I ask you: what's the main goal of a singleton? To keep a unique and shared instance of something throughout the app's lifecycle.

So, what's the problem with Singletons and Tests? The answer is related to a requirement of unit testing and a characteristic of singletons in general:

<img src="https://raw.githubusercontent.com/serralvo/serralvo.github.io/master/_posts/singletons-and-testing.jpg" />
