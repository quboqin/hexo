---
title: 'From MVC to MVVM-C, Step by Step'
date: 2018-08-23 12:30:22
tags: [iOS, MVC, MVVM, RxSwift, Coordinator, Design Pattern, Architecture Pattern, ViewModel]
categories:
  - Software Architecture
---

# Table of Contents

## Preface
  In this article, I will share my experience how to refactor the codebase from MVC to MVVM-C. I will start with a simple example application, and do the refactoring step by step slowly to show you every key points of architecture pattern in iOS development.

## Introduction of the Demo application
  I will create an application which is cloned from some applications about cryptocurrency market on Appstore. The application doesn't touch any buying or selling actions, it just show the marketing data from Open APIs, so you don't worry about your wallet :)

  It is a very simple example, including only four main pages.

  The "Prices" show the current marketing prices, you can input the name of cryptocurrency which you interested in to filter the table list, and you can also sort the order by click the name, price and change label on the section header.

  <img src="/From-MVC-to-MVVM-C-Step-by-Step/Price_View_Controller.png" width="50%" margin-left="auto" margin-right="auto">

  Swipe the table items in the price list, you can save these items into your favorite list. In the "My Favorites", you can swipe the item to remove it from your favorite list.

  <div display="flex">
  <img src="/From-MVC-to-MVVM-C-Step-by-Step/Swipe_to_Add_Favorite.png" width="30%" padding="5px">

  <img src="/From-MVC-to-MVVM-C-Step-by-Step/My_Favorites_ViewController.png" width="30%" padding="5px">

  <img src="/From-MVC-to-MVVM-C-Step-by-Step/Swipe_to_Remove_from_Favorite_List.png" width="30%" padding="5px">
  </div>

  Click the item in the Price list, the item will be expanded to show a kline diagram.

  <img src="/From-MVC-to-MVVM-C-Step-by-Step/Expand_ViewController.png" width="50%" margin-left="auto" margin-right="auto">

  At the right cover of the price page, there is a setting button which will pop a setting page to let you choice the data source of the kline. You have two options, one is from Huobi, the other is from Crypto Compare. You can also click the switch to save your favorites then they will be reloaded when you open the application next time.

  <img src="/From-MVC-to-MVVM-C-Step-by-Step/Settings_ViewController.png" width="50%" margin-left="auto" margin-right="auto">

  That's all. Though it is an really simple example, it still cover lots of knowledges in iOS development. But this article only focuses on some topics listed below:
  1. the architecture patterns: <br/>
  From MVC, MVVM, MVVM+RxSwift to MVVM+RxSwift+Coordinator, so many acronyms, which architecture is the best? I think that none of them is the "Best" one. There is ** "No Silver Bullet"**. Each one can suit better a specific scenario. But we must follow some principles. In this article, I will share my experience and list the pros and the cons of each pattern, and talk about how to make choice when I design an application.
  2. how to implement the user interface: <br/>
  The first thing in writing an application is implementing the user interface. But sometime we need essential data to help us to design the UI. Where these data come from? Do we need to write the networking layer first and fetch data from the backend? Or just grab some data through branch of http requests and save these data as an asset file in JSON format? <br/><br/>
  But there is another good idea that we can use some data mock platforms to generate these data automatically.
    * [Mockaroo](https://mockaroo.com/) <br/>
    <img src="/From-MVC-to-MVVM-C-Step-by-Step/Mockaroo.png" width="50%" margin-left="auto" margin-right="auto">
    * [Easy Mock(大搜车)](https://github.com/easy-mock/easy-mock) <br/>
    <img src="/From-MVC-to-MVVM-C-Step-by-Step/Easy_Mock.png" width="50%" margin-left="auto" margin-right="auto"> <br/>
    This is also an open source solution, you can build your own data mock platform on your computer.
  3. how to build a networking layer <br/>
  After we finish our UI design, we must create service modules to connect the real world to get the data and present these data onto our UI components. If you are lazy, you can import a third party solution, the popular one is [Alamofire](https://github.com/Alamofire/Alamofire). It is very easy to use and very powerful. But in this article I will build my networking layer from scratch. You can reference this paper [Writing a Network Layer in Swift: Protocol-Oriented Approach](https://medium.com/flawless-app-stories/writing-network-layer-in-swift-protocol-oriented-approach-4fa40ef1f908). [Malcolm Kumwenda](https://medium.com/@malcolmcollin) is a good guy to help you create you own networking layer step by step. <br/> <br/>
  Why do I need to write networking layer by myself? One principle is we must implement the core module by ourselves and if these module are not very difficult to implement. Be careful to depend on a third party source, particularly when lots of other modules depend on it. You will take high risk to rely on these third party solutions.  
  4. how to test your modules. <br/>
  The deliver quality is based on how we dedicate on testing. And all outputs of refactoring the architecture are let us more easy to test our code. ViewModel separated from ViewController is more helpful to test the business logic and the presentational logic.
  5. immigrate from procedural programming to functional Reactive Programming <br/>
  Immigrating from procedural programming to declarative programming is just like people who immigrate to a new country where the language is totally different.
  the life is hard, but it is still going on. If you estimate that the business logic and the state of your application will become more complicated as time goes by.


  >The functional programming paradigm was explicitly created to support a pure functional approach to problem solving. Functional programming is a form of declarative programming. In contrast, most mainstream languages, including object-oriented programming (OOP) languages such as C#, C++, and Java, were designed to primarily support imperative (procedural) programming.

## Design Our UI
#### Storyboard or Programmatic is a problem
  Here is the whole picture of our pages in the storyboard.

  <img src="/From-MVC-to-MVVM-C-Step-by-Step/The_Whole_Picture_of_the_UI.png" width="90%" margin-left="auto" margin-right="auto">

  There are still lots of debates about whether to use storyboard or write UI programmatically. You can find a lot of articles to summarize the pros and cons on both sides([Why I Don’t Use Storyboard](https://blog.bobthedeveloper.io/why-i-dont-use-storyboard-fe14a1a99f58) and [Storyboard vs Programmatically in Swift](https://medium.com/@chan.henryk/storyboard-vs-programmatically-in-swift-9a65ff6aaeae)). These are guidelines when I need to make a choice at the crossroads.

  |Storyboard|Programmatic|
  |-|-|
  |Ease to use|Reusability|
  |Visualization|Merge Conflict|

  In a big project, the choice is depended on your team and the consensus made in your team. I prefer using storyboard to create static view objects and doing it programmatically when I create visual objects dynamically.

  >When I teach, I don’t wanna make my audience fall asleep. So, why not make it a little more interesting since seeing is believing.🤔

  Because this is a demo application, and the UI is very simple, so using storyboard is good choice.

#### How many view controllers do we need?
  UX/UI designer create the wireframes by using **Sketch**. Sometimes we just map the pages in the sketch file to view controllers, they are almost one-on-one mapping. But be careful! Before apply other architecture paradigm, we start from MVC, and view controllers in MVC mode are so heavy, so discuss the features with your UX/UI designers or product designers first if you don't want to mess up. Sometimes we must separate some of features in one page and move them into other view controllers. These view controllers are embedded in the "host" view controller as child view controllers. In our demo application, when we click on the item in the tableview, the cell will be expanded and show the kline of selected cryptocurrency, the expanded view in the cell view need a child view controller to coordinate the kline data service and the view who show the kline. So I  dynamically create a view controller named "ExpandViewcontroller" as a child view controller embedded in "CoinListViewController".

  Expand the cell view and add the "ExpandViewcontroller" as a child view controller,
  ```swift
    expandViewController = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "ExpandViewController") as! ExpandViewController

    self.addChildViewController(expandViewController)
    cell.expandView.addSubview(expandViewController.view)

    expandViewController.view.translatesAutoresizingMaskIntoConstraints = false

    NSLayoutConstraint.activate([
        expandViewController.view.leadingAnchor.constraint(equalTo: cell.expandView.leadingAnchor),
        expandViewController.view.trailingAnchor.constraint(equalTo: cell.expandView.trailingAnchor),
        expandViewController.view.topAnchor.constraint(equalTo: cell.expandView.topAnchor),
        expandViewController.view.bottomAnchor.constraint(equalTo: cell.expandView.bottomAnchor)
        ])

    expandViewController.didMove(toParentViewController: self)
  ```
  Fold the cell view when we click the cell again,
  ```swift
  expandViewController?.view.removeFromSuperview()
  expandViewController?.removeFromParentViewController()
  expandViewController = nil
  ```
