---
published: true
title: Tips for big teams and unit testing
reading-time: 3
layout: post
---

Automated tests are an important topic when talking about software quality, and when your team or project starts to grow, new challenges appear. In this article, I'll cover some aspects/issues of unit tests in big teams and how to fix them.

At scale, there are a few key root causes of issues with unit testing: no standardization or guidelines, duplicated code, flaky tests, no documentation. In my experience, these issues increase following the project growth.

When a project starts to grow fast (e.g. double the size in less than a year) and in a non-structured way (without documentation, guidelines, onboarding) some situations can happen in a sequence:

- New developers tend to write code following undocumented guidelines. For instance: "I looked in our features figured that the _pattern_ was X, so I followed that" or "I asked for help and someone told me to follow _this way_". As expected, this situation will happen with unit tests too.
- After some time, these _new practices_ were added through the project, and then, some questions will arise about what is expected (or even _"right"_). In some cases, these kinds of questions may appear during code review, tech meetings, or just idle talk.
- When the team realizes, the codebase has a bunch of patterns, techniques, approaches, different names for the same concept, duplicated code, etc. Needless to say this will also happen to tests.

<img src="https://raw.githubusercontent.com/serralvo/serralvo.github.io/master/_posts/approaches_and_engineers.png" />
<small>Created on <a href="https://www.autodraw.com" target="_blank">autodraw.com/</a></small>

</hr>

So, if your team is growing fast, you are recognizing the situations that I described above, and want to avoid the _red arrow_, consider following my tips about unit testing at scale:

- Define a test stack after understanding the tradeoffs, e.g. _"should we use libraries X or Y?"_, or _"what are the benefits of lib X or Y?"_, etc. Most importantly, after the decisions are made, documenting the _whys_ and sharing that document with the entire team, is key.

- Apply a linter to test files. If your production code has a linter, why not have a linter for the tests as well? _Code is code_, right?

- Create guidelines about naming. _"How should I name my Spy/Mock/Dummy file?"_, or _"What suffix should I use in the property that changes when some method is called?"_, or _"How should I name my test functions?"_. These questions will come up and it's important to have a document to answer them. If you use code generation within your project, also consider standardizing the name of the generated code whenever possible.

- If your project has mechanisms that are used in the whole app (a singleton, for example), consider creating reusable mocks for that. Imagine if each test creates a private spy class about a protocol, what will happen when this protocol got a change? You will need to change all _spies_ of this one. Reusable mocks, please.

</hr>

Special thanks to <a href="https://twitter.com/rogerluan_" target="_blank">Roger Oba.</a>
