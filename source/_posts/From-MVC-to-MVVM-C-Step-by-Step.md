---
title: 'From MVC to MVVM-C, Step by Step, Part I'
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
Understanding how and when a view updates requires a deeper understanding of the  UIKit framework. This article focuses on the architecture patterns, so we will not talk more about the UI staffs. But the topics I discuss below are very important to you to build an application as quickly as possible.

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

  #### AutoLayout and constraint
  Being familiar with UIKit is very important for developing your user interface. And you must be prepare to move beyond the basics, and dive into the UI framework. When you do start writing iOS apps, you’ll have a solid and rigorous
  understanding of what you are doing and where you are heading.

  There are three stages before views can be displayed on the screen. The first step is "updating constrains", which is form subviews to their superviews(bottom-up). It will prepare the information needed for the second step called "layout". In this step, frames(centers and bounds) of views will be set depending on the constrain system. Finally the "display" pass renders the views on the screen.

  Before you take control of layout, you need to know how to trigger the override functions in these three steps, and when to trigger these functions. The table below lists all relative functions in these steps.

  |Method purposes|Constraints|Layout|Display|
  |-|-|-|-|
  |Implement updates (override, don’t call explicitly)|updateConstraints|layoutSubviews|draw|
  |Explicitly mark view as needing update on next update cycle|setNeedsUpdateConstraints invalidateIntrinsicContentSize|setNeedsLayout|setNeedsDisplay|
  |Update immediately if view is marked as “dirty”|updateConstraintsIfNeeded||layoutIfNeeded|
  |Actions that implicitly cause views to be updated| Activate/deactivate constraints<br>Change constraint’s value or priority<br>Remove view from view hierarchy|addSubview<br>Resizing a view setFrame that changes a view’s bounds (not just a translation)<br>User scrolls a UIScrollView<br>User rotates device|Changes in a view’s bounds|

  ##### A good example of changing the local constraints

``` Swift
// Create a new property to hold the aspect ratio constraint of an imageView
fileprivate var aspectRatioConstraint:NSLayoutConstraint?

// Override the updateConstraints function to recreate the aspect ratio constraint depending on the image width and height
override func updateConstraints() {
  super.updateConstraints()

  var aspectRatio: CGFloat = 1
  if let image = image {
    aspectRatio = image.size.width / image.size.height
  }

  // setting the isActive of a constraint to false will make this constraint be nil, so you need to create a new one
  aspectRatioConstraint?.isActive = false
  aspectRatioConstraint =
      imageView.widthAnchor.constraint(
        equalTo: imageView.heightAnchor,
        multiplier: aspectRatio)
  aspectRatioConstraint?.isActive = true
}

// Change the image property declaration in order to invalidate the constraints and trigger the autolayout process again:
var image: UIImage? {
  didSet {
    imageView.image = image
    setNeedsUpdateConstraints()
  }
}
```

  ##### Change constraints with animations
If you want to replace the default behaviors of constraints animation, you must call the _layoutIfNeeded_ function in the animation block, otherwise the change of the constraint will be executed in the update cycle of the run loop, so the animation block you defined will be useless.

``` Swift
func collapseHeader() {
    UIView.animate(withDuration: 1.2, animations: {
        self.headerHeightConstraint.constant = self.minHeaderHeight
        self.view.layoutIfNeeded()
    })
}

func expandHeader() {
    UIView.animate(withDuration: 0.2, animations: {
        self.headerHeightConstraint.constant = self.maxHeaderHeight
        self.view.layoutIfNeeded()
    })
}
```

In the _updateConstraints()_ function, you can't invalidate any constraints, otherwise you will get a crash because your program has already been in the layout cycle.
<img src="/From-MVC-to-MVVM-C-Step-by-Step/Update_Cycle.png" width="80%" margin-left="auto" margin-right="auto"><br>
You can change the properties which are relative with the constraints, then invalidate constraints and trigger the layout process in the event process stage of the main run loop, then the system will call the callback function _updateConstraints_ you override in the ** _update cycle_ ** . In this function, you can change/remove/add constraints which depending on the properties changing before. This guideline can also apply to the process of layout and display.

There are some scenarios about how to fine-tune the frames of your views by overriding the _layoutSubviews_ function. The first one is not necessary, because you can add a constraint, but it is an example.

``` Objective-C
- layoutSubviews
{
    [super layoutSubviews];
    if (self.subviews[0].frame.size.width <= MINIMUM_WIDTH) {
        [self removeSubviewConstraints];
        self.layoutRows += 1;
        [super layoutSubviews];
    }
}

- updateConstraints
{
    // add constraints depended on self.layoutRows...
    [super updateConstraints];
}
```

> There is a trick. Because the layout process is top-down, you can get the frame of the current view before you call the super function _layoutSubviews_. After the super function _layoutSubviews_ being called, the subviews of this view also get their frames, so you can get the sizes and positions of its' subviews. In this case, we compare the width of the first subview, and change the _layoutRows_ property, then call its' super _layoutSubviews_ to fire the autolayout process from the updating constraints step again.<br>
You can also subclass your first subview(in this case), and override the _layoutSubviews_ function in its' subclass, and get its' frame like the next example which fine-tune the width of a multi-line label

``` Objective-C
@implementation MyLabel
- (void)layoutSubviews
{
    self.preferredMaxLayoutWidth = self.frame.size.width;
    [super layoutSubviews];
}
@end
```

> Or you can make this adjustment at the view controller level, put this logic into the _viewDidLayoutSubviews_ function. Because the autolayout process is finished,  so you must re-fire this process by calling _layoutIfNeeded_

``` Objective-C
- (void)viewDidLayoutSubviews
{
    [super viewDidLayoutSubviews];
    myLabel.preferredMaxLayoutWidth = myLabel.frame.size.width;
    [self.view layoutIfNeeded];
}
```

##### StackView with Fill Distribution

#### Core Animation’s models, classes and blocks
  Animation in iOS is huge topic, but the animation process can be as simple as changing properties during a given time. It includes two main animation systems. One is based on "View Animation", and the other is called "Core Animation". I only list some key issues about Animation.

##### View Animation
  There are three stages for "View Animation" because of the historical evolution of iOS. I just give some snippet codes below for each of these stages.

  * Begin and commit
  ``` Swift
  UIView.beginAnimations(nil, context: nil)
  UIView.setAnimationDuration(1)
  self.v.backgroundColor = .red
  UIView.commitAnimations()
  ```

  * Block-based animation
  ``` Swift
  UIView.animate(withDuration:1) {
    self.v.backgroundColor = .red
  }
  ```

  * Property animator
  ``` Swift
  let anim = UIViewPropertyAnimator(duration: 1, curve: .linear) {
      self.v.backgroundColor = .red
  }
  anim.startAnimation()
  ```

##### Implicit Layer Animation
  Before we jump to "Core Animation", we also can change properties on CALayer. It is called "Implicit Layer Animation". Just set layer properties, and your layers animate in the default way. You can not control the animation except you use a "CATransaction" with "Implicit Layer Animation". And remember, there is always an implicit transaction surrounding your code, and you can operate on this implicit transaction without any begin and commit.

  ``` Swift
  // can be omitted
  CATransaction.begin()

  CATransaction.setAnimationDuration(0.8)
  self.globalLabel.layer.transform = CATransform3DMakeRotation(0, 1, 0, 0)

  // can be omitted
  CATransaction.commit()
  ```

##### Core Animation
  "Core Animation" is also called "Explicit Layer Animation". To specify a property using a keyPath, we can create an CABasicAnimation object or its inheritance, then we add this object onto the layer of a view. There are two problems when we use "Core Animation". One is setting a property on a layer will fire the "Implicit Layer Animation", We can prevent this side effect by disabling actions:

  ``` swift
  CATransaction.begin()
  CATransaction.setDisableActions(true)

  let transformAnimation = CABasicAnimation(keyPath: #keyPath(CALayer.transform))
  transformAnimation.fromValue = CATransform3DMakeRotation(0, 1, 0, 0)
  transformAnimation.toValue = CATransform3DMakeRotation(CGFloat(Double.pi), 1, 0, 0)
  transformAnimation.duration = 2
  transformAnimation.autoreverses = true
  transformAnimation.repeatCount = .infinity

  CATransaction.setCompletionBlock({
    self.globalLabel.layer.transform = CATransform3DMakeRotation(0, 1, 0, 0)
  })

  self.globalLabel.layer.add(transformAnimation, forKey: #keyPath(CALayer.transform))

  CATransaction.commit()
  ```

  The other problem is if we use "Core Animation", the property will not change to the value when the animation is end, because "Core Animation" create a new layer to present the animation, called "presentation layer". So we need it set the new value into the property of its "model layer" in the completion block.

  If you use "Implicit Layer Animation" or "View Animation", you don't need to care about these problems, because the Animation framework will handle these for you. But when you use the "Core Animation" which is the fundamental underlying iOS animation technology, you get more powerful, so you need more responsibility.

##### Transitions
  Another concept called "Transitions" is very confused, if you are not a native English speaker. Usually we use "Transitions" in two situations. We can change the content of a view such as the image of a UIImageView using "transition(with:duration:options:animations:completion:)" function:
  ``` swift
  let opts : UIViewAnimationOptions = .transitionFlipFromLeft
  UIView.transition(with:self.iv, duration: 0.8, options: opts, animations: {
      self.iv.image = UIImage(named:"Smiley")
  })
  ```
  Or we can replace the first view with the second view by using "transition(from:to:duration:options:completion:)" function.
  ``` Swift
  let lab2 = UILabel(frame:self.lab.frame)
    lab2.text = self.lab.text == "Hello" ? "Howdy" : "Hello"
    lab2.sizeToFit()
    UIView.transition(from:self.lab, to: lab2,
        duration: 0.8, options: .transitionFlipFromLeft) { _ in
            self.lab = lab2
}
  ```

#### Custom UIViewController Transitions
The transitioning API includes several components which conform a collection of protocols. The diagram below shows these relative components:

<img src="/From-MVC-to-MVVM-C-Step-by-Step/Transitioning_API.jpg" width="80%" margin-left="auto" margin-right="auto"><br>

Here is the 7 steps involved in a presentation transition:

1. Before You trigger the transition either programmatically or via a segue, you need to set the transitioningDelegate property in your "to" view controller. In this case, the transitioningDelegate is the "from" view controller. The "from" view controller in here is also the ** presenting ** view controller. When you trigger the transition via a segue, the override function _prepare(for:_)_ in the presenting view controller will be called, in this function, you can get the "to" view controller(also called "** presented **") form the segue destination. Then you can set the transitioningDelegate property of the destination to itself.
``` swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    super.prepare(for: segue, sender: sender)
    segue.destination.transitioningDelegate = self

    if let navigationController = segue.destination as? SettingsNavigationController,
    let settingsViewController = navigationController.viewControllers.first as? SettingsViewController {
      settingsViewController.delegate = self
    }
}
```
2. UIKit asks the “presented” view controller (the view controller to be shown) for its transitioning delegate. If it doesn’t have one, UIKIt uses the standard, built-in transition. In here the delegate is the presenting view controller. The transition delegate conforms two methods of ** UIViewControllerTransitioningDelegate protocol **. One is for presenting, the other is for dismissing.
3. UIKit then asks the transitioning delegate for an animation controller via animationController(forPresented:presenting:source:). If this returns nil, the transition will use the default animation. The transition delegate create and return an animation controller who confirms the ** UIViewControllerAnimatedTransitioning protocol **
4. UIKit constructs the transitioning context. Be careful, if you want to see the presenting view controller behind the presented view controller, you need to set the presentation style to .overFullScreen or .overCurrentContext. If you set this property to .fullScreen, the view of the presenting view controller will be replaced
5. UIKit asks the animation controller for the duration of its animation by calling transitionDuration(using:).
6. UIKit invokes animateTransition(using:) on the the animation controller to perform the animation for the transition.
7. Finally, the animation controller calls completeTransition(_:) on the transitioning context to indicate that the animation is complete.

And the steps for a dismissing transition are nearly identical.
When you tigger the reserves process, UIKit asks the transitioning delegate for an animation controller animationController(forDismissed dismissed:), then repeat the steps from 4 to 7.

Here is the animation controller code who play the animation role:

``` Swift
class PresentTransitionController: NSObject, UIViewControllerAnimatedTransitioning {

    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.6
    }

    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {

        let fromViewController = transitionContext.viewController(forKey: .from)!
        let toViewController = transitionContext.viewController(forKey: .to)!
        let containerView = transitionContext.containerView

        let screenBounds = UIScreen.main.bounds
        let topOffset: CGFloat = 160.0

        var finalFrame = transitionContext.finalFrame(for: toViewController)
        finalFrame.origin.y += topOffset
        finalFrame.size.height -= topOffset

        toViewController.view.frame = CGRect(x: 0.0, y: screenBounds.size.height,
                                             width: finalFrame.size.width,
                                             height: finalFrame.size.height)
        containerView.addSubview(toViewController.view)

        UIView.animate(withDuration: transitionDuration(using: transitionContext), delay: 0.0, usingSpringWithDamping: 0.8, initialSpringVelocity: 1.0, options: .curveEaseOut, animations: {
            toViewController.view.frame = finalFrame
            fromViewController.view.alpha = 0.3
        }) { (finished) in
            transitionContext.completeTransition(finished)
        }
    }
}
```

#### Add Gesture On the Section header

## Connect with internet
We already have got the "Views" in MVC pattern, it is time to build the "Models" and create networking service to connect with our backend. There are lots of approaches to build the "Models" and networking service.
1. The most formal way is introducing some mocks and stubs before creating real data models and connecting the endpoints. The frontend developing will not depend on the progress of the backend developing. As I mentioned before you can use the data mock platform to mock the data we need, some of these platform are open source. You can setup your own data mock platform on your laptop, and you also can use their online service to create data, then download and save to a JSON file as a resource embedded into your application bundle. The other benefit is you can use these mock data to test your business logics and application logics. You will not stuck in debugging with backend engineers and complain each other, and the testers who get theirs test sets can help you test your application and business logics.

2. If the backend service is ready and stable. You can skip the step to build the mock data and stubs. Importing a third party module is also a good choice. [Alamofire](https://github.com/Alamofire/Alamofire) is one of the famous open sources. Using a heavy library sometimes you must be careful, if you don't want to relay on this one.

3. There are an assumption that building a networking framework is a big challenge. Implementing the basic feature I think is not difficult, you can reference this topic ["Writing a Network Layer in Swift: Protocol-Oriented Approach
"](https://medium.com/flawless-app-stories/writing-network-layer-in-swift-protocol-oriented-approach-4fa40ef1f908) from Malcolm Kumwenda. This guy write a good article to teach you how to build your own networking layer step by step.

First I wrote a http request using the native API **_URLSession_**, extend each model with a JSON initializer, handle the nested objects. When I get the data from a http request, I map the JSON format into a corresponding object.

``` Swift
extension Ticket {
    init?(from json: [String: Any]) {
        guard
            let id = json["id"] as? UInt,
            let name = json["name"] as? String,
            let symbol = json["symbol"] as? String,
            let website_slug = json["website_slug"] as? String,
            let rank = json["rank"] as? UInt,
            let circulating_supply = json["circulating_supply"] as? Double,
            let total_supply = json["total_supply"] as? Double,
            let max_supply = json["max_supply"] as? Double,
            let quotesDict = json["quotes"] as? [String: Any],
            let last_updated = json["last_updated"] as? UInt64
        else {
            return nil
        }

        self.init(id: id,
                  name: name,
                  symbol: symbol,
                  website_slug: website_slug,
                  rank: rank,
                  circulating_supply: circulating_supply,
                  total_supply: total_supply,
                  max_supply: max_supply,
                  quotes: Dictionary(uniqueKeysWithValues:
                    quotesDict.map { key, value in
                        let (key, value) = (key, QUOTE(from: (value as? [String: Any])!))
                        return (key, value!)
                    }
                  ),
                  last_updated: last_updated)
    }
}
```

``` Swift
let url = URL(string: "https://api.coinmarketcap.com/v2/ticker/")!

session.dataTask(with: url) { (data, response, error) in
    DispatchQueue.main.async {
        if let error = error {
            completionHandler(.error(error))
            return
        }

        guard
            let data = data,
            let jsonObject = try? JSONSerialization.jsonObject(with: data, options: []),
            let jsonDict = jsonObject as? [String: Any],
            let dataDict = jsonDict["data"] as? [String: Any]
        else {
            completionHandler(.error(ServiceError.cannotParse))
            return
        }

        let items = dataDict.values.map{ $0 } as? [[String: Any]]
        let tickets = items!.compactMap(Ticket.init)
        completionHandler(.success(tickets))
    }
}.resume()
```

It is not a bad idea, if you write a simple example, but it will turn to a disaster quickly when you write a real application. But I don't want to use the Alamofire right now. So I followed Malcolm Kumwenda to create my own networking library, base on his codebase, I make a little change to support multi data sources.

The diagram is showed below, there three main roles in this diagram
<img src="/From-MVC-to-MVVM-C-Step-by-Step/Networking.png" width="80%" margin-left="auto" margin-right="auto"><br>

1. Different endpoints in every data source are defined in different enums(**_Huobi API, CoinMarketAPI and CryptoCompareAPI_**), these enums all conform EndPointType protocol, you can get http parameters, paths and methods from these variables in this protocol, the return values depend on the values of the enums.

2. Router is a generic class. Giving different EndPoints, we can create different routers. The responsibility of this Router is prepare the parameters depending on the given endpoints, then trigger the real url sessions.

3. Our application involve three websites: Huobi, CoinMarket and CryptoCompare. So we create three NetworkManagers for each site. These NetworkManagers are all singleton, creating a router using an endpoint to replace the placeholder is their only jobs, then inject this router into themselves. These NetworkManagers all inherit form NetworkManager who handle the errors during the networking requests, and parse the JSON result form backend. It aslo managers all tasks created by the router. You can cancel these tasks by unique id.

4. NetworkEnvironment is a singleton class which distinguishs different running environment:
``` Swift
enum NetworkEnvironment {
    case qa
    case product
    case staging
}
```
In Swift 4 they provide Decodable protocol, I use this to convert my JSON objects to an equivalent Struct or Class. Sometimes you don't need to write a single line, but if the names of the properties in our Classes are different from the keys returning from backend, we need to build the mapping.

``` swift
//MARK: Listings
struct Item {
    let id: UInt
    let name: String
    let symbol: String
    let websiteSlug: String
}

extension Item: Decodable {
    private enum ItemCodingKeys: String, CodingKey {
        case id
        case name
        case symbol
        case websiteSlug = "website_slug"
    }

    init(from decoder: Decoder) throws {
        let itemContainer = try decoder.container(keyedBy: ItemCodingKeys.self)

        id = try itemContainer.decode(UInt.self, forKey: .id)
        name = try itemContainer.decode(String.self, forKey: .name)
        symbol = try itemContainer.decode(String.self, forKey: .symbol)
        websiteSlug = try itemContainer.decode(String.self, forKey: .websiteSlug)
    }
}
```

## The Bridge - Controllers

Now we have the two important elements "Views" and "Models" in MVC pattern. We will finish our business logics and application logics in our "Controllers" to connect our "Views" with our "Models".

The main feature of this demo application is fetch data from three websites who provide real time exchange infos of cryptocurrencies, and show these data in a tableview. Thee is no business logic at the front end, and this application only contains some simple application logics:
1. Filter the result got from the network by typing some keywords at the top of the tableview.
2. Choice whether or not to list tokens by clicking a switch of selecting cryptocurrency type.
3. Sort the list in a descending or ascending order by name or price or change rate.
4. You can swipe the tableview item in the price view controller to add this cryptocurrency into your favorite list. You can also remove it from your favorite list. This favorite list will be saved, and will be reload when this application is being launched.
5. When you click the item, the tableview cell will be expanded, and show the kline of the selected cryptocurrency in a chart.
6. As a demo application, I add a feature that you can select different data sources of this kline in the setting view controller.
7. It will pop a webview to present the detail information of the selected cryptocurrency, when you tap the "eye" on the right top of the expanded chart view.

Writing these application logics is not a big challenge. After I added a few lines of code, I can filter this list by some keywords and type. But when I was adding the sorting logic, I was aware that I had already messed around the view controllers.

``` Swift
let ticker = showCoinOnly ? (whichHeader == .coin ? sortTickers(_tickers.filter({ !$0.isToken }))[indexPath.row] : _tickers.filter({ !$0.isToken })[indexPath.row])
    : (indexPath.section == 0 ? (whichHeader == .coin ? sortTickers(_tickers.filter({ !$0.isToken }))[indexPath.row] : _tickers.filter({ !$0.isToken })[indexPath.row])
        : (whichHeader == .token ? sortTickers(_tickers.filter({ $0.isToken }))[indexPath.row] : _tickers.filter({ $0.isToken })[indexPath.row]))
```


But when I wanted to display the result on the tableview, Only three conditions I needed to apply on the result, this expression made me crazy, and I put this snippet in every place in the tableview controller. I realized that this code must be used and can be readable. All of the functions in the above code is filtering and sorting when some conditions is changed.

<img src="/From-MVC-to-MVVM-C-Step-by-Step/Data_Filter.png" width="80%" margin-left="auto" margin-right="auto"><br>

This diagram is more like some figures in reactive programming article, this is the main motivation why we need to change from functional programming paradigm to imperative mode. Let me finish the MVC part, because we still meet other problems. So I create an extension of Array where its element must be "Ticker". Then I moved this logic into the "milter" function in this extension.

``` Swift
extension Array where Element == Ticker {
    func filter(BySearch searchText: String?) -> [Ticker] {
        var filterTickers = [Ticker]()
        if let lowcasedSearchText = searchText?.lowercased() {
            filterTickers = self.filter { $0.fullName.lowercased().range(of: lowcasedSearchText) != nil }
        }
        if filterTickers.count == 0 {
            filterTickers = self
        }
        return filterTickers
    }

    func milter(filterBy searchText: String?, separatedBy section: Section, sortedBy condition: Sorting) -> [Ticker] {
        var filterTickers = [Ticker]()
        filterTickers = filter(BySearch: searchText)

        filterTickers = filterTickers.filter {
            switch section {
            case .coin:
                return !$0.isToken
            case .token:
                return $0.isToken
            default:
                return true
            }
        }

        if condition == .none {
            return filterTickers
        }
        filterTickers = filterTickers.sorted {
            switch condition {
            case .name:
                return $0.fullName < $1.fullName
            case .nameDesc:
                return $0.fullName > $1.fullName
            case .change:
                return $0.quotes["USD"]!.percentChange24h < $1.quotes["USD"]!.percentChange24h
            case .changeDesc:
                return $0.quotes["USD"]!.percentChange24h > $1.quotes["USD"]!.percentChange24h
            case .price:
                return $0.quotes["USD"]!.price < $1.quotes["USD"]!.price
            case .priceDesc:
                return $0.quotes["USD"]!.price > $1.quotes["USD"]!.price
            default:
                return $0.fullName < $1.fullName
            }
        }

        return filterTickers
    }
}
```

In the tableview delegator or datasource, I replaced the old ones with
``` swift
let _tickers = tickers.milter(filterBy: self.lowercasedSearchText,
                              separatedBy: Section(section: indexPath.section),
                              sortedBy: _sorted)
```

I cleaned my room, but the story wasn't end. The sorting logic start from the user clicking on the section header. There are three section headers, one is for coin type, one is for tokens and the other is for my favorite list. And each of these section has three types of sorting. To keep this sorting statuses, I also introduced lots of "Variables" to present the status of sorting logic. These code snippet are scattering into several classes. If I didn't check the code for some days and reviewed it again, I would almost forgot their relationship with the sorting logic.

The kline chart is embedded in the ExpandViewController, and the ExpandViewController is appeared in two view controllers, one is the price view controller, the other is the Favorite viewcontroller, so the ExpandViewController has two instances. And these instances have different life cycle. We change the kline datasource from the Setting view controller, this change can't be conveyed to every one. There is no good method to cache this change, except using a global singleton object to store this status. That is why I created a class called "KLineSource". Too many global object means disaster will come. But the MVC model, I have no choice, if I don't want to keep this status in a lot of objects, and transfers this status from one object to the other one. It will be more uglily than the method before.

``` swift
class KLineSource {
    static let shared = KLineSource()

    var dataSource = DataSource.cryptoCompare

    private init() {

    }
}
```

This is a simple demo application. So I only met these two main barriers. One is how to maintain relative statuses, and connect them effectively, some statuses are changed, some one will follow. Another is how to keep some statuses that we can access them convenience. I work around these issues, the codebase looks uglily now. And I have no idea how to test these code. So far I have finished the application under MVC pattern. You can check the code out here.

I think the MVVM pattern only brings us branch of new classes, if we don't introduce the reactive framework. If we separate some logics from the massive view controllers, it will help us to test our application logic, but I am not sure about it is useful. Our real problem is how to synchronize these statuses distributed in different objects. The RxSwift give us a better solution to connect the statuses together. So I will jump to the MVVM w/ RxSwift in the next section. 


## MVC-RxSwift
<img src="/From-MVC-to-MVVM-C-Step-by-Step/Methods_For_Information_Flow.png" width="80%" margin-left="auto" margin-right="auto"><br>
When I separate our application into different objects with MVC pattern, we need some methods to exchange informations betwen these objects. In Apple's framework, there are seveval ways we can choice, from delegates, callback blocks, notifications, KVO, to target/action event observers. It is convenient for us to glue these objects together. But it is also confused to us. Sometimes how to choice what kind of technology is a challenge. The big problem is we aslo need to connnect these different asynchronization methods in an appropriate way. In this process, we might create more additional varables to keep the status of the applcation. 

It's time to introduce the RxSwift. In RxSwift, an Obseravble can emit a sequences of event. And this event contains some kind of value. RxSwift also provide lots of operators to convert and combine these events whatever you want. The different kinds of Subjects are powerful tools we can use to bond these sequences to create reactive chains. When we subscribe these sequences, we will get the responses automatically. 

Though RxSwift is a powerful tool, but it seems like a totally different language. We need to change our minds to master the new language. And the learning curve of Swift is higher than learning Swift as an Objective-C programmer. Swift and Objective-C are different dialects in a same language. But when we use RxSwift, we just like talking with Alias. There is no shortcut to master the new language. Just use try and use it.

#### How to encapsulate networking API using RxSwift
The first thing to me is encapsulating callback methods in networking API with RxSwift Observable. I added a new function called _getDataFromEndPointRx_ to convert the callback into an Observable. And the Observable is a disposable object, I put the cancel logic in a callback function when the disposable object is created. So when the Obseravble object is released, It will cancel the http request automatically. 

```Swift
    func getDataFromEndPointRx<T: Decodable>(_ endPoint: EndPoint,
                             type: T.Type) -> Observable<T> {
        return Observable<T>.create({ observer in
            let taskKey = self.getDataFromEndPoint(endPoint, type: type) {
                (data, error) in
                if error != nil {
                    observer.onError(error!)
                    return
                }
                
                if let _data = data as? T {
                    observer.onNext(_data)
                }
            }
            
            return Disposables.create {
                self.cancelRequestWithUniqueKey(taskKey)
                self.tasks.removeValue(forKey: taskKey)
            }
        })
    }

    func getDataFromEndPoint<T: Decodable>(_ endPoint: EndPoint,
                                                 type: T.Type,
                             networkManagerCompletion: @escaping NetworkManagerCompletion) -> taskId {
        let task = router.request(endPoint) { (data, response, error) in
            DispatchQueue.main.async {
                if error != nil {
                    networkManagerCompletion(nil, NetworkResponse.failed)
                }
                if let response = response as? HTTPURLResponse {
                    let result = self.handleNetworkResponse(response)
                    switch result {
                    case .success:
                        guard let responseData = data else {
                            networkManagerCompletion(nil, NetworkResponse.noData)
                            return
                        }
                        do {
                            let apiResponse = try JSONDecoder().decode(type.self, from: responseData)
                            networkManagerCompletion(apiResponse, nil)
                        } catch {
                            networkManagerCompletion(nil, NetworkResponse.unableTODecode)
                        }
                    default:
                        networkManagerCompletion(nil, result)
                    }
                }
            }
        }
        let taskId = NSUUID().uuidString
        tasks = [taskId: task]
        return taskId
    }    
```
In our application, we use Filesystem APIs to save our favorite cryptocurrencies. The Filesystem APIs are more like our networking APIs. We can use the same methodology to convert the Filesystem APIs into RxSwift Observables. But in this application, I didn't do that, the main reason is our saved data is very small, and we can treat these APIs as synchronized function calls. So I think is not necessary for us to encapsulate them. I just subscrible the actions, and call these functions when the actions take place.

``` Swift
_selectRemoveMyFavorites.subscribe(onNext: {
    self.removeSavedJSONFileFromDisk()
}).disposed(by: disposeBag)
```

#### How to handle the error events emmited from networking API, Materialize and Dematerialize 
Our networking APIs have not the ability to handle the errors, they just conduct these errors from the bottom to the applcation layer. How to handle these errors is depended on our application logic. Without RxSwift, every thing is ok. But when we convert the callback function into Observable onject, the chain will be broken, because I send the errors through the onError event. After the onError event is sent, the Observable object will be terminated, this is not we want, but we still want to get these error messages in the application layer. How can we get them? In RxSwift, they provide a mechanism called _Materialize_. It add a shell on the onError event, then this event turns into a normal event, and the error event become a value inside the normal event. Then we can use a filter operator to catch the error event when it happans. If it is not a error event, then the filter will _dematerialize_ the event, we will get the original data again.

``` Swift
huobiReload.withLatestFrom(symbol.asObservable())
    { (dataSource, symbol) in
        return symbol
    }
    .flatMap { (symbol) ->  Observable<Event<KlineResponse>> in
        return HuobiNetworkManager.shared.getDataFromEndPointRx(.historyKline(symbol: symbol.lowercased() + "usdt", period: "5min", size: 150), type: KlineResponse.self).materialize()
    }
    .filter { [unowned self ] in
        guard $0.error == nil else {
            Log.e("Catch Error: +\($0.error!)")
            self.histoHourVolumes = Variable<[OHLCV]>([])
            return false
        }
        return true
    }
    .dematerialize()
    .map { (kLineResponse) -> [KlineItem] in
        return kLineResponse.data
    }
    .map { (kLineItems) -> [OHLCV] in
        var ohlcvs = [OHLCV]()
        kLineItems.forEach({ (kLineItem) in
            let ohlcv = OHLCV(time: 0, open: kLineItem.open!, close: kLineItem.close!, low: kLineItem.low!, high: kLineItem.high!, volumefrom: 0.0, volumeto: 0.0)
            ohlcvs.append(ohlcv)
        })
        return ohlcvs
    }
    .bind(to: histoHourVolumes)
    .disposed(by: disposeBag)
```

The Houbi API need me to provide correct symbol name, but I get the symbol name of cryptocurrency from CoinMarket. Sometimes the names of these symbols are not One-to-one correspondence. In this demo application, I don't want to handle this situation, So i will get an error return from the Huobi API, when I switch from CryptoCompare to the Huobi API. 

#### Use RxSwift Extensions: RxDataSource
Now we get the data, we can bind these data to views. RxCocoa has already encapsulated lots of controls in UIKit, but it is not enough when we encount some complex controls like TableView or CollectionView. At this situation, we need to use the extension framework from RxSwift community. RxDataSource makes us work more easy. First we create a datasource which is comfirmed SectionModelType protocol. This datasource also define a callback how to config tableview cell when we get the datasource item.

``` Swift 
dataSource = RxTableViewSectionedReloadDataSource<SectionModel<String, Ticker>>(
    configureCell: { dataSource, tableView, indexPath, item in
        let cell = tableView.dequeueReusableCell(withIdentifier: self.cellIdentifier, for: indexPath) as! CurrencyCell
        cell.selectionStyle = .none
        cell.setCoinImage(item.imageUrl, with: (self.baseImageUrl)!)
        cell.setName(item.fullName)
        cell.setPrice((item.quotes["USD"]?.price)!)
        cell.setChange((item.quotes["USD"]?.percentChange24h)!)
        cell.setVolume24h((item.quotes["USD"]?.volume24h)!)
        
        self.currentUrlString = (self.baseImageUrl)! + item.url
        
        cell.setWithExpand(self.expandedIndexPaths.contains(indexPath))
```

After we get the data from Observable of the networking API, we map the event from the [Ticker] to [SectionModel<String, Ticker>], and bind this sections into the tableview. The RxDataSource solved our multi sections problem. 

``` Swift
Observable.combineLatest(_coins, __tokens) {
    return ($0, $1)
}
.map {
    var sections = [SectionModel<String, Ticker>]()
    if $0.count != 0 {
        sections.append(SectionModel(model: "Coin", items: $0))
    }
    if $1.count != 0 {
        sections.append(SectionModel(model: "Token", items: $1))
    }

    return sections
}
.bind(to: tableView.rx.items(dataSource: dataSource!))
.disposed(by: disposeBag)
``` 

But I can not find how to customize the section header view in different sections. The RxSwift keeps the delegate solution for some situation it can't handle. We can set a delegate to a tableview.rx, then the tableview still will call our delegate function in the view controller. 

``` Swift
dataSource?.canEditRowAtIndexPath = { dataSource, indexPath in
    return !self.expandedIndexPaths.contains(indexPath as IndexPath)
}

tableView.rx
    .setDelegate(self)
    .disposed(by: disposeBag)
```

``` Swift
extension PricesViewController {    
    func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath) -> [UITableViewRowAction]? {
        let favoriteRowAction = UITableViewRowAction(style: UITableViewRowAction.Style.default, title: "Favorite", handler:{ [weak self] action, indexpath in
            
            guard let ticker = self?.dataSource?[indexPath] else {
                return
            }
            
            self?.favoritesViewController.baseImageUrl = self?.baseImageUrl
            self?.favoritesViewController.addTicker(ticker)
        })
        
        return [favoriteRowAction]
    }
    
    override func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
        if section == 0 && self.coinSectionHeaderView == nil || section == 1 && self.tokenSectionHeaderView == nil {
            if let headerView = super.tableView(tableView, viewForHeaderInSection: section) as? SectionHeaderView {
                if section == 0 {
                    headerView.section = SortSection.coin
                    self.coinSectionHeaderView = headerView
                } else {
                    headerView.section = SortSection.token
                    self.tokenSectionHeaderView = headerView
                }
                return headerView
            }
            return self.coinSectionHeaderView
        } else if section == 0 {
            return self.coinSectionHeaderView
        } else {
            return self.tokenSectionHeaderView
        }
    }
}    
```

#### Connect all Observables and Observers together
Distinguishing from the MVC pattern, we connect all of the Observables and Observers together and gather all application logics like filtering, sorting and separating, etc in one place. I drew a whole diagram of these logics to show how different it is between using RxSwift and the MVC model.
<img src="/From-MVC-to-MVVM-C-Step-by-Step/RxSwift_Chain.png" width="100%" margin-left="auto" margin-right="auto"><br>

In same cases, I use _filter_ operator to activate different path in the reactive chain. And using _withLatestFrom_ operator to tigger one event by the other event. Changing Kline datasource is an event emitted from a segment control in the Setting ViewController, and showing the Kline need another event which is changing the symbol name. The _changing datasource event_ will trigger the _changing symbol name_ event. 

<img src="/From-MVC-to-MVVM-C-Step-by-Step/RxSwift_Chain_2.png" width="100%" margin-left="auto" margin-right="auto"><br>

#### Interaction between view controllers from delegate to obserable and observer
After I established the reactive chains to replace the delegates in the view controllers, I deleted some of properties who refer to the data models in the Price View Controler and Favorite View Controller. And I don't need to keep the intermedia statuses for the appplication logics, but It doesn't mean we can remove all statuses, these statuses are useful in exchanging informations between view controllers. If the object owned by the view controller, I think you can use the Observable in the object through the property directly. But most of time, I need to create a stub in the target view controller like the Price View Controller, and bind this stub with the Observable in the source view controller like the Setting ViewController. When the SettingViewController is removed from the memory, the Variable showCoinOnly will be disconnected from its source and sleep. When the SettingViewController is created again, the connection will be rebuilt, and the showCoinOnly will resend the event from the SettingViewController to the filtering logic chain as a router.

<img src="/From-MVC-to-MVVM-C-Step-by-Step/RxSwift_Chain_3.png" width="100%" margin-left="auto" margin-right="auto"><br>

In this diagram, you can see I create a two way bind in the SettingViewCOntroller, the showCoinOnly in the PriceViewController will keep the status, when the SettingViewController is created again, the Switch will get the status again. But how to write a 2-way bind is confusing me, I just wrote the code below, but I don't know why it didn't cause the infinitive problem.

``` Swift 
showCoinOnlySwitch.rx.isOn.changed.debug()
    .bind(to: _selectShowCoinOnly)
    .disposed(by: disposeBag)

didSelectShowCoinOnly.debug()
    .bind(to: showCoinOnlySwitch.rx.isOn)
    .disposed(by: disposeBag)
```

And if you want to use 2-way bind in other place, you can create a generic funtion to handle this. 

``` Swift
infix operator <->

@discardableResult func <-><T>(property: ControlProperty<T>, variable: Variable<T>) -> Disposable {
    let variableToProperty = variable.asObservable()
        .bind(to: property)

    let propertyToVariable = property
        .subscribe(
            onNext: { variable.value = $0 },
            onCompleted: { variableToProperty.dispose() }
    )

    return Disposables.create(variableToProperty, propertyToVariable)
}
```


#### The Problems I met
##### The scope of a status
When I implement the application logic, I realize that there are three kinds of status I need to mantain. 
1. Local Status
To be honest there is no local status when we use the RxSwift, the RxSwift doesn't keep the status, it just trigger events from the Observables and consume these events by the Observers. I use this terminology in order to distingush the other two statuses. 
2. Status as a property
Just like I discuss in the last section, the showCoinOnly in the PriceViewCOntroller keep the status transfered from the SettingViewController. This Variable will share information between two view controllers, one will be released, the other is alway keeping in the memory. In this case I keep the status in the PriceViewcontroller, because, the PriceVIewController will not be released in the application whole life time.
3. Global status
But sometimes the two controllers who share inforamtion will all be released. So how can I keep the status like the KLineSource in this application. The KLineSource is produced from the SeetingViewController when a user select the datasource on the segment control, and the KLineSource si consumed by the ExpandViewController to determine which APIs will be called. But these two view controllers will be release at runtime. So where can I keep the status. In this demo example I create a singleton global object to keep this status called KLineSource. 

``` Swift

class KLineSource {
    static let shared = KLineSource()
    
    var dataSource = Variable<DataSource>(DataSource.cryptoCompare)
    
    private init() {
        
    }
}
```

##### How to initialize the RxSwift reactive chain when some of these view controllers are created dynamically
Because some of these view controllers will be created dynamically, it bring
another problem what is the appropriate time to create a view controller, and when to bind the reaction chain. When we create a view controller and send a event to its Observer, it won't get this event if it hasn't band the the Observer first.

``` Swift
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        super.prepare(for: segue, sender: sender)
        segue.destination.transitioningDelegate = self
        
        if let navigationController = segue.destination as? SettingsNavigationController,
            let settingsViewController = navigationController.viewControllers.first as? SettingsViewController {
    
            settingsViewController.didSelectShowCoinOnly
                .bind(to: showCoinOnly)
                .disposed(by: disposeBag)
            
            // FIXED: How to save the status
            settingsViewController._selectShowCoinOnly.onNext(showCoinOnly.value)
```

When the seague is triggered, the SettingViewControler will be created, and I will send the ShowCoinOnly status to the SettingViewController. But if I put the binding code in the ViewDidLoad() in the SettingViewController, this event will be discarded because when the event is being sent, the chain hasn't been setup yet. So I need to move the setup logic into awakeFromNib(). And I also need to load the view objects first compulsively like the code below.

``` Swift
override func awakeFromNib() {
    super.awakeFromNib()
    // FIXED: How to save the status..., force load view
    _ = self.view
    
    let closeButton = UIBarButtonItem(title: "Close", style: .plain, target: self, action: nil)
    self.navigationItem.leftBarButtonItem = closeButton
    
    dataSourceSegment.selectedSegmentIndex = KLineSource.shared.dataSource.value.rawValue
    
    setupBindings()
}

override func viewDidLoad() {
    super.viewDidLoad()
}
``` 

It is not very comfortable, I didn't find any other good solution before we use the coordinator. One resposibilty of the coordinator is maintaining the lifecycle of ViewModels and the Controllers. It will be a solution to fix this issue. But before we jump to the MVVM-RxSwit-Coordinator, we must create the ViewModel layer first.

## Create the ViewModels
View Models are extracted from view controllers. The process of building the view models is separating our application logics from view controllers. The view controller only keep the resposibilities for 
1. maintain lifecycle of it's views
2. create a view model
3. connect inputs and outputs of the view model to the view elements
   
<img src="/From-MVC-to-MVVM-C-Step-by-Step/MCVMVMV.gif" width="100%" margin-left="auto" margin-right="auto"><br>   
   
View models are not depending on view controllers. They only include application logics, so they are easy to be tested.

One typical view model in our demo is the SettingViewModel.

``` Swift
class SettingViewModel {
    // MARK: - Inputs
    let selectDataSource: AnyObserver<DataSource>
    let selectShowCoinOnly: AnyObserver<Bool>
    let removeMyFavorites: AnyObserver<Bool>
    let cancel: AnyObserver<Void>
    
    // MARK: - Status
    
    // MARK: - Outputs
    let didSelectDataSource : Observable<DataSource>
    let didSelectShowCoinOnly: Observable<Bool>
    let didRemoveMyFavorites: Observable<Bool>
    let didCancel: Observable<Void>
    
    init() {
        let _selectShowCoinOnly = BehaviorSubject<Bool>(value: false)
        self.selectShowCoinOnly = _selectShowCoinOnly.asObserver()
        self.didSelectShowCoinOnly = _selectShowCoinOnly.asObservable()
        
        let _removeMyFavorites = PublishSubject<Bool>()
        self.removeMyFavorites = _removeMyFavorites.asObserver()
        self.didRemoveMyFavorites = _removeMyFavorites.asObservable()
        
        let _cancel = PublishSubject<Void>()
        self.cancel = _cancel.asObserver()
        self.didCancel = _cancel.asObservable()
        
        let _selectDataSource = BehaviorSubject<DataSource>(value: DataSource.cryptoCompare)
        self.selectDataSource = _selectDataSource.asObserver()
        self.didSelectDataSource = _selectDataSource.asObservable()
    }
}
```

We don't have the coordinator yet, so the SettingViewController is still responsable for creating it's view model and connecting the inputs and outputs of the view model to some view controls and inputs and outputs of other view controllers.

If we take a picture of the SettingViewModel, we can get the diagram below:

<img src="/From-MVC-to-MVVM-C-Step-by-Step/ViewModel2.png" width="100%" margin-left="auto" margin-right="auto"><br> 

In this diagram, you can see the main logic on the client side is included in the PriceViewModel. This view model is just one class, and rely on any other views and view controllers. So we can write a test for it easily. 

## The Fininal Step, Build a coordinator tree.
I call it as a coordinator tree, because, all of our coordinators which corresponds their view controllers, and have parent-children relationship like view controllers. 

The top node of this tree is the AppCoordinator, it contains two coordinators, one is PriceCoordinator and the other FavoriteCoordinator.

All coordinators are inherited from BaseCoordinator. The main job of a coordinator is 
1. create a view model 
2. create a view controller
3. inject the view model into the view controller
4. maintain the navigation between view controllers.

The fourth one can be separated some sub-steps
1. creat a target coordinator as an observable object
2. flatmap some of source coordinator obserable into the target observable, this process is most like calling a networking service. 
3. bind the source coordinator input onto the target coordinator, so we can get the result from the target coordinator. 

The whole process looks like calling a asynchronization function, when we start a new coordinator. Every coordinator must comfirm the start() function. In this funtion the coordinator returns an Observable object for it's source coordinator.

``` Swift
 override func start() -> Observable<Void> {
        let navigationViewController0 = rootViewController.viewControllers![0] as! UINavigationController
        let viewController = navigationViewController0.viewControllers[0] as! PricesViewController
        
        let navigationViewController1 = rootViewController.viewControllers![1] as! UINavigationController
        let favoritesViewController = navigationViewController1.viewControllers[0] as? FavoritesViewController
        
        let viewModel = PriceViewModel()
        viewController.viewModel = viewModel
        
        let showSettings = viewModel.showSettings
            .flatMap { [weak self] _ -> Observable<SettingCoordinationResult> in
                guard let `self` = self else { return .empty() }
                return self.showSettings(on: viewController, with: viewModel)
            }.share()
        
        showSettings
            .filter({
                if case .kLineDataSource = $0 {
                    return true
                }
                return false
            })
            .map {
                if case .kLineDataSource(let value) = $0 {
                    return value
                }
                return DataSource.cryptoCompare
            }
            .bind(to: GlobalStatus.shared.klineDataSource)
            .disposed(by: disposeBag)
            
        showSettings
            .filter({
                if case .showCoinOnly = $0 {
                    return true
                }
                return false
            })
            .map {
                if case .showCoinOnly(let value) = $0 {
                    return value
                }
                return false
            }
            .bind(to: viewModel.showCoinOnly)
            .disposed(by: disposeBag)
        
        showSettings
            .filter({
                if case .removeMyFavorites = $0 {
                    return true
                }
                return false
            })
            .map {
                if case .removeMyFavorites(let value) = $0 {
                    return value
                }
                return false
            }
            .bind(to: (favoritesViewController?.viewModel.deleteFavoriteList)!)
            .disposed(by: disposeBag)
        
        
        window.makeKeyAndVisible()
        
        return Observable.never()
    }
    
    private func showSettings(on rootViewController: UIViewController, with rootViewModel: PriceViewModel) -> Observable<SettingCoordinationResult> {
        let settingsCoordinator = SettingCoordinator(rootViewController: rootViewController, rootViewModel: rootViewModel)
        return coordinate(to: settingsCoordinator)
    }
```

The relationship between these objects is showed in the figure below:

<img src="/From-MVC-to-MVVM-C-Step-by-Step/Coordinator_ Asynchronous_Call.png" width="100%" margin-left="auto" margin-right="auto"><br> 

So far we have finished our demo example. It is not a real project, but it is also not very simple. I show you how to refactor the code from MVC pattern to MVVM-Coordinator pattern. And this example also includes some key points to write a iOS application, such as networking service, autolayout, ...

I hope it is helpful for the newbs in iOS develepment.














