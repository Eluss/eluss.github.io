---
layout: post
title: My motives for MVVM + FlowControllers path
category: blog
header: /assets/images/mvvm-flowcontrollers/mvvm-flowcontrollers.jpeg
excerpt: There are many different approaches to building an iOS application. MVC, MVP, MVVM, VIPER, Redux... And surely, you can name many others. I've been writing iOS apps for some time now, and I have noticed that MVVM + FlowControllers approach works pretty well for me. In this post I'd like to focus on "WHY?" I use it over other approaches, "HOW?" I use it in my apps.
---

![image](/assets/images/mvvm-flowcontrollers/mvvm-flowcontrollers.jpeg)

There are many different approaches to building an iOS application. MVC (Model View Controller), MVP (Model View Presenter), MVVM (Model View ViewModel), VIPER (View Interactor Presenter Entity Routing), Redux... And surely, you can name many others. I've been writing iOS apps for some time now, and I have noticed that MVVM + FlowControllers approach works pretty well for me.
In this post I'd like to focus on "Why?" I use it over other approaches and "How?" I use it in my apps.

# Why?  

## The beginning
Starting iOS development is usually connected with learning basic patterns that you can use to solve problems that you will run into while struggling to create your first screens in the app. Most probably the first pattern that will take care of your views and logic behind them will be MVC, which is very popular among iOS developers (but not only them). This is mostly caused by the fact that Apple highly promotes this pattern across the `UIKit` framework and code examples that you can find in their tutorials.

## Is MVC bad?
Yes! It's the worst of all! It causes Massive ViewControllers to appear over your app... Jokes aside...  

No, it's not. I was really glad when I've recently read post ["Much ado about iOS app architecture" ](http://aplus.rs/2017/much-ado-about-ios-app-architecture/). I cannot say that I fully agree with everything that is said there, but there are parts that I can surely identify with. Why was I glad to read this? Because of this sentence:  

"No one is forcing you to implement multiple DataSources in one Controller. To initiate network calls in viewDidLoad. To parse JSONs in UIViewController. To hard-wire Views with Singleton instances."  

Many developers blame MVC for its horrible mess and chaos which in fact is created by developers themselves. If you lack discipline, then even VIPER will not help you. Certain patterns may make it easier to structure the code properly and keep it clean, but it's always up to you whether you keep the discipline or not.

Should I use MVC if it is not that bad in the end? If I was a skilled consultant I should probably say "it depends". MVC obviously has its own pros and cons and I bet you can find many great articles that will help you to make up your mind.

## Why MVVM?  
If MVC is not that bad, then what are my motives for following the MVVM path? Just to name a few of them:  
- It allows me to move a big portion of my code far away from `UIKit` ([Which gives an ability to test this code faster - as macOS frameworks ](https://eliaszsawicki.com/are-your-views-dumb-enough/)).  
- It allows me to test the logic that drives my views easier.  
- It works really well with reactive programming approaches (but you don't need to use them to benefit from MVVM)  

So now, after adopting the principles of MVVM pattern, I'm able to have my passive views (as dumb as possible) and my logic that drives my views which is also separated from the UIKit.  

Why not going any further with the division approach that patterns like VIPER target really well? Well... you can do that. I've never used VIPER in a big project before (I'd be glad to hear your opinion on this!), however, I'd say that this kind of patterns could be an easy overkill for a small/medium sized apps.  
I feel that MVVM works really well if you want to keep your solutions easy to understand while having an ability to easily test your code and have a nice separation from UIKit-dependant parts of your code.

## FlowControllers  
What are the motives for using them?  
It is not something unusual when inside ViewController "A" you find code that is responsible for transition to screen "B". A problem with this approach is that out of a sudden you create a tight coupling between these two entities and you might find yourself in trouble if you want to split or reuse them in other contexts.  
Well... It's always good to ask a question - Is it a problem? If you're working on something simple then using this kind of navigation may be completely fine, however, if you're up to something more complicated - adding a flowController will surely help. Keep in mind that adding a flowController is not a high cost task - I tend to use them even in simple apps as they help me to organize my code better and give me a good look on how navigation in my app works. Adding a flowController to your app will help you with:  
- Keeping your screens separated from each other, which will allow you to modularise and reuse them easily.  
- Controlling flow in parts of your app (You will probably have many different flowControllers)  
- Dependency injection place for your `ViewModels`

## The discipline

A pure fact that you start using MVVM and `FlowControllers` will not instantly make your code base clean. Guess what? You can still end up with `Massive ViewModels`! It's up to you whether you keep yourself tight and organize your code well.



# How?  


## MVVM
Ok, so how do I use MVVM? How do I keep my `ViewModels` clean? How do I use `FlowControllers`? Jump on board and let's see a quick example that will allow us to see the concept in use.  
I have to admit that while I was taking my first step into iOS development, I considered `ViewModel` to be an object that holds values which would be displayed by a view. At that point I did not see that much value in using `ViewModels`. What changed my point of view was the approach that you can read about on [Microsoft patterns and practices](https://msdn.microsoft.com/en-us/library/hh848246.aspx). The core information for me was the fact that the `View` layer in Microsoft's approach was represented as XAML (quote: "with a limited code-behind that does not contain business logic"). Ok... So does it mean that `ViewModel` is not only about holding values represented by our views? Can it also contain logic that drives these views? YES! Following this approach on iOS, it encourages you to keep your view layer simple (which in case of MVVM would be both `UIView` and `UIViewController`) and move the logic to `ViewModel`. This is the first step that allows us to reach better testability and move our code far away from `UIKit`.

Let's go step by step through a quick example of a MVVM pattern used together with `FlowController`. 

Let's start with `AppDelegate`. What do we have here?

```swift
   class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    var mainNavigationController: UINavigationController!
    var mainFlowController: MainFlowController!
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        
        mainNavigationController = UINavigationController()
        
        window = UIWindow(frame: UIScreen.main.bounds)
        window?.rootViewController = mainNavigationController
        window?.makeKeyAndVisible()
        
        mainFlowController = MainFlowController(rootNavigationController: mainNavigationController)
        mainFlowController.startFlow()
        
        return true
    }

}
```
The first thing that is important is that I create my `MainFlowController` which will be responsible for controlling my application flow. The only dependency that I pass here is my main navigation controller that is the root controller of my main window. I've also seen different approaches that pass `UIWindow` to a flowController directly, however, if you do not need it, then I would prefer to have this "lighter" object which has a more defined responsibility.

What about the `MainFlowController` itself? 

```swift
class MainFlowController {
    
    private let rootNavigationController: UINavigationController
    
    private lazy var entryViewModel: EntryViewModel = {
        let fetcher = FakeUserFetcher()
        let viewModel = EntryDefaultViewModel(userFetcher: fetcher)
        viewModel.onUserNameSelected = self.onUserNameSelected
        return viewModel
    }()
    private lazy var entryViewController = EntryViewController(viewModel: entryViewModel)
    
    init(rootNavigationController: UINavigationController) {
        self.rootNavigationController = rootNavigationController
    }
    
    func startFlow() {
        rootNavigationController.pushViewController(entryViewController, animated: true)
    }
    
    func onUserNameSelected(userName: String) {
        print("name: \(userName)");
        // Show next screen using selected user name
    }
    
}
```

The core element of this `FlowController` is surely the `startFlow` method, which in this case pushes a new `ViewController` to our navigation stack. This `ViewController` is created with a `ViewModel` as a parameter. The `viewModel` will be very important for us in a while, however, from the `FlowController's` point of view, we're especially interested in `onUserNameSelected` closure. This is the output of the `ViewModel` that our `FlowController` could be potentially interested in. For example, after selecting a user name we could open the next screen that allows us to select a birthday, surname, favourite pet or simply displays the name in a fancy way. Your `FlowController` allows you to control the flow of your application and makes the `ViewModel` unaware of the context that it's used in. `ViewModel` has its job to do and after it's done our `FlowController` will be notified about it. 

The ViewController

```swift
class EntryViewController: UIViewController {

    private let viewModel: EntryViewModel
    
    private lazy var fetchUserButton: UIButton = {        
        let button = UIButton()
        button.addTarget(self, action: #selector(onFetchUserButtonTapped), for: .touchUpInside)
        // ...setup button
        return button
    }()
    
    private lazy var selectUserButton: UIButton = {
        let button = UIButton()
        button.addTarget(self, action: #selector(onSelectUserButtonTapped), for: .touchUpInside)
        // ...setup button
        return button
    }()

    private let currentUserLabel = UILabel()
    
    init(viewModel: EntryViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil) // Layout created programamtically, sorry for that ;(
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func loadView() {
        super.loadView()
        setupView()
        setupObservers()
    }
    
    private func setupObservers() {
        currentUserLabel.reactive.text <~ self.viewModel.userName // bind viewModels output to your UI
    }
    
    @objc func onFetchUserButtonTapped() {
        viewModel.fetchUser()
    }
    
    @objc func onSelectUserButtonTapped() {
        viewModel.selectUser()
    }

    private func setupView() {
        // setup constraints etc
    }
    
}
```

In my case, `ViewController` is a part of `View` layer in MVVM. What happens here is: 
- Binding `ViewModel's` output to UI components
- Invoke methods from `ViewModel`
- If needed, passing lifecycle methods to `ViewModel`
- Layout setup

And last but not least - The ViewModel

```swift
protocol EntryViewModel {
    func fetchUser()    
    func selectUser()
    
    var onUserNameSelected: ((String)->Void)? {get set} // If you're using Reactive frameworks, you can also implement this as a stream
    // var onNextSelected: (()->Void)? // Other callbacks could exist here, they do not need to pass data

    var userName: Property<String?> {get} // ReadOnly property that allows others to observe its changes, but not change the property from outside of viewModel
}

class EntryDefaultViewModel: EntryViewModel {
    
    private let userFetcher: UserFetcher
    private var mutableUserName: MutableProperty<String?> = MutableProperty(nil)
    lazy var userName: Property<String?> = Property(self.mutableUserName)
    weak var onUserNameSelected: ((String)->Void)?
    
    // injecting dependencies to your viewModel 
    init(userFetcher: UserFetcher) {
        self.userFetcher = userFetcher
    }
    
    func selectUser() {
        if let userName = userName.value {
            onUserNameSelected?(userName)
        }
    }
    
    func fetchUser() {        
        // Bind the result of fetchUser() function to a mutableUserName property
        mutableUserName <~ userFetcher.fetchUser().map { $0.name }
    }
    
}
```

The `ViewModel's` part is the actual beating heart for our `View` layer. It will allow the `View` to observe changes in properties it exposes. It will also contain logic for various behaviours. Keep in mind that you do not need to keep the entire logic inside the `ViewModel`, you can easily extract it and add it as dependency - this is what happens with `userFetcher`. This allows us to keep our `ViewModels` cleaner and easily testable as we can mock our dependencies.


# Summary
Following the way of MVVM + FlowControllers has helped me a lot in separating/organising my code and making in more testable. If I had to pinpoint three benefits of using this approach that are most valuable for me, then it would be:  
- `ViewControllers` that can be easily reused in different scenarios  
- Comfortable testing of `ViewModels` that contain your logic and are not coupled with `UIKit`  
- Application modules that have their navigation defined in one place

If you have never used a similar approach to your apps, then I'd fully encourage you to try it!

Would you like to learn more about this topic? Check these out:  
[Improve your iOS Architecture with FlowControllers](http://merowing.info/2016/01/improve-your-ios-architecture-with-flowcontrollers/)  
[Swift By Sundell - "Boy, I have a lot of thoughts on this", with special guest Soroush Khanlou](https://www.swiftbysundell.com/podcast/10)  
[Coordinators – Soroush Khanlou](https://www.youtube.com/watch?v=a1g3k3NObkE)




*This article is cross-posted with [my company blog](http://blog.brightinventions.pl/)*