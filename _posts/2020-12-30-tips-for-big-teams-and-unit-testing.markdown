---
published: false
title: Tips for big teams and unit testing
reading-time: 1
layout: post
---

Automated tests are an important topic when we are talking about software quality, and when your team or project starts to grow new challenges appear, in this article, I'll cover some aspects / issues of unit tests in big teams and how to fix these.

There are some reasons related to issues with unit testing at scale: no standards or guides, duplicated code, flaky tests, no documentation. In my experience, I saw these issues raising while the project grows.

When a project starts to grow fast (in this case fast means double the size in less than a year) and in a non-structured way (without documentation, guidelines, onboarding) some situations can happen in a sequence:

- New developers tend to write code following non-documented guidelines, which means "I looked in our features the _patterns_ and I followed that" or "I asked for help and someone said me to follow _this way_". As expected, this situation will happen with unit tests too.
- After some time, these _new practices_ were added through the project, and then, some questions will arise about what is expected (or even _"right"_) in some cases, this kind of question can appear while code review, tech meetings, or just idle talk.
- When the team realizes, the code base has a bunch of patterns, techniques, approaches, different names for the same concept, duplicated code, etc. These situations will happen in tests too.

_insert here a connection with these issues and my typs_

___

Finally, my tips about unit testing at scale are:

Define a test stack after understanding the tradeoffs, _should we use Libraries X or Y?_. Witches are the benefits of lib X or Y? Important, after the definition of the stack, documenting the _whys_ and share that doc with all team. 

Apply a lint in test files. If your production code has a lint, why does not have a lint for the tests? _Code is code_, right?

Create guidelines about naming. _How I should name my Spy / Mock / Dummy file?_ / _What suffix I should use in the property that is changed when some method is called?_. These questions will appear and are pretty important to have a guide about naming. Another important point is about the name of test functions, consider creating a standard. A plus about naming is related to code generation, do that when possible.

If your project has mechanisms that are used in the whole app (a singleton, for example), consider creating reusable mocks for that. Imagine if each test creates a private spy class about a protocol, what will happen when this protocol got a change? You will need to change all _spies_ of this one. Reusable mocks, please.

