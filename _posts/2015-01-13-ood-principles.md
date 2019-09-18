---
layout: post
title: "OOD Principles"
description: ""
category: []
---

## Why we need OOD principles?

Object Oriented Programming ([OOP][1]) languages brings us `abstract data type`, `encapsulation`, `inheritance` and `polymorphyism`, which give us tremendous flexibility and also the ability to reuse and extent existed modules in development.

Unfortunately, __flexibility__ is not only a powerful weapon in handling real world problems, but also it may cause our design to go the wrong way, which means our projects may get harder and harder to maintain and add new features when the scale of them grow bigger and bigger. Hence we need some fundamental principles or best practice to help us during OOD (Object Oriented Design).

## What are the fundamental principles for OOD?

In 2000, [Robert Cecil Martin][2] wrote an article called [Design Principles and Design Patterns][3] and introduced  so called __SOLID__ principles to help us to do robust and easy-to-extend design. Uncle Bob(Robert' s nickname well known in developer community) published another book in 2002 named __Agile Software Development: Principles, Patterns, and Practices__ to systematically elaborate these principles. In his [official blog][9], he has gave the short but precious definition of them:

- SRP ([Single responsibility principle][4]) - A class should have one, and only one, reason to change.  
	_[xiaohui] This principle is to achieve the goal of high cohesion and low coupling. A class should have one and only one responsibility and focus on doing certain things highly related to itself, it should only be affected by only one potential specification change._
- OCP ([The Open Closed Principle][5]) - You should be able to extend a classes behavior, without modifying it.  
	_[xiaohui] This principle is to achieve scalability more easily by being open to extension and closed to modification. Since in this way we do not need to change the existed and released code, we do not need to be worried about compatibility issue and potential regression._
- LSP ([The Liskov Substitution Principle][6]) - Derived classes must be substitutable for their base classes.  
	_[xiaohui] This principle is also the basic requirment of polymorphism, we should be able to base class instance wherever derived class instance can be used, that is how we achieve dynamic binding of class behaviors._
- ISP ([The Interface Segregation Principle][7]) - Make fine grained interfaces that are client specific.  
	_[xiaohui] This principle is also the requirment of high cohesion and low coupling, it helps us to lower down the dependency of different modules and brings us more flexibility._
- DIP ([The Dependency Inversion Principle][8]) - Depend on abstractions, not on concretions.  
	_[xiaohui] This principle requires us to forget about the layer-ed design. We should not let high-lever modules depend on low-level modules and let abstraction depends on concretions. Because in this way, any change of the high-level modules which indicate the functionalities may require us to change the implementation of low-level modules in waterfall style, this can be the nightmare when we face future scalability and extensions. What we should do is to let both high-level and low-level modules depend on abstraction and let detailed implementation depend on abstraction._

Following the guidance of these principles could truly help us, like this:

![][fun]

## There are More 

The __Gang Of Four__(GOF) introduced 23 design patterns in the book [Design Patterns: Elements of Reusable Object-Oriented Software ][10], these patterns then have been widely discussed in OOP developer community and practiced ever since. Most of these patterns actually respect __SOLID__ principles, except some of them for certain scenarios like __Singleton__ violating __SRP__ and __Facade__ violating __OCP__. 

## Trade off and options

Both __SOLID__ principles and the 23 design patterns are not so-called `silver bullets`. The real world problems and software specification can be extremly complicated, not to mention that requirments changes almost always happend in most projects. They are guidance, understanding them and getting familiar with when and how to apply them with trading off based on current resources and deadlines is the key. For a project developed by more than one person, [The Mythical Man-Month: Essays on Software Engineering][11] has already conviced us besides having the correct design and development skills, there are other things like __progress tracking__, __team management__, __continuous integration__, etc that could critically impact the result of the project. I myself stand by this according to my 7 years development and team management experience. 

Uncle Bob introduced __Agile__ development style in the book mentioned above and gave a solution for project quality assurance, progress tracking process and the recommended ways to handle requirment changes. With all of these guidances along with __SOLID__ principles he concluded and [other principles about module package][9] also introduced by him, we have a whole roadmap to try our best to achieve our goal.

[1]: http://en.wikipedia.org/wiki/Object-oriented_programming
[2]: http://en.wikipedia.org/wiki/Robert_Cecil_Martin
[3]: http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf
[4]: http://en.wikipedia.org/wiki/Single_responsibility_principle
[5]: http://en.wikipedia.org/wiki/Open/closed_principle
[6]: http://en.wikipedia.org/wiki/Liskov_substitution_principle
[7]: http://en.wikipedia.org/wiki/Interface_segregation_principle
[8]: http://en.wikipedia.org/wiki/Dependency_inversion_principle
[9]: http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod
[10]: http://en.wikipedia.org/wiki/Design_Patterns
[11]: http://en.wikipedia.org/wiki/The_Mythical_Man-Month
[fun]: /images/ood.jpg


