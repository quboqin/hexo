---
title: Architecture on iOS
date: 2018-04-28 13:29:45
tags:
---

### Why we need to choice an architecture before we write a program on iOS

### The critirias of evaluration of architecture

### The History of architectures

### Design Paradiam and Design Principle
##### SRP
Single Responsibility Principle
How to use SRP properly with clean Architecture. Every module or class should have responsibilty over a single part of the functionality provided by the software.

#### MVC
#### Apple's MVC
MVC introduced by Appll is stands for Massive View Controller. It is funny, but it is ture. But It is not the fault of the architecture, It depented on how we use it correctly. The architecture doesn't tell us how to use it correctly and where to put all other responsibilty we need. 

#### MVVM
#### MVVM with RxSwift
#### MVVM with RxSwift and Coordinattor
#### MVVM with ReSwift
#### Clean Architecture
Clean architecture explicitly divides some responsibilities among its classes: the presenters bridge the divide between UI and business logic, the interactors handle our use-cases, routers help us get to new scenes, etc. 

Clean architecture has become quite popular on Apples platforms and for a good reason. There are even several approaches we can choose from, like VIPER [4] and Clean Swift [3].

The interactor contains the applications business logic but there’s only one interactor per view controller. 

##### Clean Swift
In my opinion, in complex scenes, the interactor shouldn’t really do anything interesting. No algorithms, no database or parsing, nothing that requires a nested if statement or a loop. This belongs in Worker classes. The responsibility of the interactor will be delegating work to its workers and passing the results to the presenter. 

In clean architecture, an interactor should represent a single use case. 

##### VIPER
I heard many people say that VIPER is very complicated. 

#### Router

### References
[Cleaner Architecture on iOS](https://medium.com/inloopx/cleaner-architecture-on-ios-ac4027b85d1f)




