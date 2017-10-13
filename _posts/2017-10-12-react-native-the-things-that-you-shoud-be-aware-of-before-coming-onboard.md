---
layout: post
title: React Native - The things that you should be aware of before coming onboard
category: blog
header: /assets/images/react-native-the-things-that-you-shoud-be-aware-of-before-coming-onboard/header.jpeg
excerpt: React Native is a neat piece of technology that I get along pretty well with. Creating multiplatform apps using shared codebase and having a great feedback loop...
---

![Header](/assets/images/react-native-the-things-that-you-shoud-be-aware-of-before-coming-onboard/header.jpeg)
 
React Native is a neat piece of technology that I get along pretty well with. Creating multiplatform apps using shared codebase and having a great feedback loop sounds really promising and after hearing such things you may start wondering "Why the hell am I not using RN?!". Well... As you probably know, many things aren't all that shiny after you go past the happy-path tutorials. In this post I'd like to give you my perspective on things that I found problematic, frustrating or things that I just wasn't prepared for while entering the React Native world. However, this will be a perspective of iOS developer who still had a lot of fun with while developing with RN.

Please do not mistake this post with a "React Native - The things that should discourage you from using it". In fact I love React Native for many reasons! My point is to make you aware of what difficulties you may find on the RN path, which will hopefully make you better prepared for them and will result with less frustration and great development experience when you decide to hop on the React Native train! 

* This post has been written on 13.10.2017, keep in mind that currently things may look different!

# Let's start!

## 1. The "Do-It-Yourself" approach  

![Tools](/assets/images/react-native-the-things-that-you-shoud-be-aware-of-before-coming-onboard/tools.jpeg)

If you are a native developer since forever, then coming into RN will be something completely new to you.
You have to keep in mind, that there will be many times where there won't be an existing solution for your usecase and the only way to push the project will be to write the whole solution yourself, adapt existing open-source project to your needs or simply fix a bug in a library. Well... This wouldn't be a surprising thing for a native developer, however keep in mind that react-native itself is also a dependency that may require fixes at times. You probably didn't have to fix things in UIKit before ;) Coming to RN from native will probably make you much more of an open-source person than you were before ;)

## 2. Leaky abstractions
Android and iOS are different platforms. It's not surprising that things may work differently. It's pretty hard to cover all cases under one universal abstraction. Sometimes you probably wouldn't event want to have an universal API for both Android and iOS, because this would mean giving up on some platform specific features that you might still want to cover with your RN code. Be prepared, that there will still be places where you will yell "Hey! I've had it in Swift!". Well... Maybe it's not covered by RN APIs yet?

## 3. Dependencies, dependencies, dependencies  
If you come from the web background, then you will not be surprised at all, but if you are a native dev, then I guess that you did not have that many dependencies in your project. If you think that you actually have many of them, then wait until you see the dependency tree in your RN project. Even at the start of the project, they will reach a high number. It may look scray, but you'll probably get used to it. You have to switch your way of thinking when you start working on JS based projects. Normally you turn your IDE on and compile the code / run tests. Here you will have to bring your own test runner / transpiler and work with that.  

## 4. Things may not work always in the same way  
Even if you use the same API and it feels great, keep in mind this simple rule. If you are writing a multi-platform app, then you should test it on those platforms separately. You will run into places, where certain implementation works on iOS, but behaves slighly different on Android. Some basic examples I can think of are:  
- A scrollView, that will result in slightly different experience while scrolling (programmatically or with your finger) on both platforms
- By default an android view will render with transparent backgrounds, while iOS will render them white.

## 5. Learning React Native  
React Native is fairly new technology. If you're looking for a RN developer it's not the same as if you would look for a Java developer. There are simply not as many of them. So in order to find one, you will usually need a person who will learn it. There are two common paths while entering the world of RN - you are either a web dev or a native dev. Which is better? It's usually hard to say, but from my perspective going from native dev to RN will be an easier task. You will surely have to learn a lot regarding the programming language, tools and environment, however you will have an advantage of knowing how to solve the native problems regarding project configuration, using native APIs, common practices in mobile world and if the need comes - you will also be able to efficiently bridge your native code to RN (And the time will surely come). Web devs! Don't be discouraged! Just keep in mind that it's not the same as writing a website for a smaller device ;)  

## 6. Project configuration  
You change a name of a build configuration in Xcode for your React Native project, and suddenly you realise that you are not able to compile it anymore. What's wrong? Well... you start reading the React Native library code (You can do it, because it's open source!) that is responsible for building the project and you find out, that other configurations than `Debug` or `Release` are not fully supported. Things like this happen and it's good to be aware of issues like this beforehand. If you find issues like this remember, that you are not alone! There is a great community that will surely help you!  
But remember - don't come with "Give me!" approach ;)  
Example project configuration issue - [Github](https://github.com/facebook/react-native/issues/11813#issuecomment-331519708)

## 7. Navigation  

![Navigation](/assets/images/react-native-the-things-that-you-shoud-be-aware-of-before-coming-onboard/navigation.jpeg)

As a native dev, whole navigation stack is focused around pushing/popping/dismissing controllers. You have a specific API for that and everything is working fine... If you come to RN world... You will find many navigation libraries that will wait for you. It will be very hard to come up with a goot fit at first. You may end up with something that will turn out to be not as pleasant as you expected. I'm currently a happy user of [React Native Navigation by Wix](https://github.com/wix/react-native-navigation). It should be fine for you, but if you're missing something... contribute!
Be aware that there you won't find a standarized way to solve navigation as you used to do in you iOS/Android apps.  

## 8. Bridging existing android/iOS libraries  
Many platform-specific libraries that you will use in React Native will in fact be native libraries bridged to javascript API. This is important to know, because in order to use a fresh version of your favourite swift library, you will also need an updated version of the library that bridges the API to JS. This may cause delay in having such updates, unless you go with do-it-yourself approach.  

## 9. Debugging  
I have to admit that debugging the code that I've written in RN projects is not as pleasant as in IDEs that I've previously used. Some of the text editors will support debugging, but in my case I've been using Typescript for my project, which means that it was a bit harder to setup the debugger in a text editor. I've used the debugger in Visual Studio Code for some time, however  many times I ended up debugging my code in chrome dev tools. By any means, it is not a horrible experience, but you will surely feel the difference compared to debugging in your IDE. 

## 10. Are you flexible with the app?  
In any project you may come to a place, where implementing a certain layout will cost you a big portion of time without actually contributing too much to the value of a project. Such things are often a nice addition to the app, however if you want to release your MVP quickly to hit the market, it might be just not worth it to implement the whole thing and go on with a simpler version of the layout instead. This is especially a thing if you are pursuing your path as RN developer, because at first the time-investment point in this case will also be caused by additional code that you may need to brige to RN or by additional features that you would have to add to existing libraries in order to support a detail in layout. It would be a good thing to have a bit of flexibility regarding the details of the UI.

## 11. You don't have to use JS  
Many people are discouraged by the fact that they will have to write JS, however I do not consider that a big problem. Let's ignore the fact that nowadays JS has a lot of nice features that you may love. Let's look at the "issue" from other perspective... You do not have to actually use pure JS for your app. In my case Typescript was a pleasant solution that made developing the app much more pleasant. It will still transpile down to JS, however you will not need to have too much contact with it. If you come from Swift/Kotlin background, then by using Typescript you will not feel as dramatic change as you would while goind on with pure JS.

Take a look at [Typescript](https://www.typescriptlang.org/) and [Flow](https://flow.org/)!

## 12. You don't have to setup your project from the ground  

![Building](/assets/images/react-native-the-things-that-you-shoud-be-aware-of-before-coming-onboard/building.jpeg)

Setting up a React Native project can be a daunting thing... It would be really time consuming to handle all those dependencies by yourself. But don't worry! There is a great tool that will help you with a setup of a project - [Create React Native App](https://github.com/react-community/create-react-native-app). It will allow you to directly jump into writing the code instead of dancing around with configs and dependencies. It's surely important to know how your project is built, but if it's your first time in "JS environment" then you will already have a lot of things to wrap your head around.


# Summary
These points should not discourage you from entering the world of React Native. They should make you aware of some problems, so that you can better prepare for them. So far I've had a great experience with it and I'm really looking forward to seeing how this technology evolves in the future.  
Well... you should surely avoid RN if you don't want to experience the feeling of longing after coming back to native development. When you come back, there will be many things that you won't feel comfortable with after trying RN ;)

This article is cross-posted with my [my company blog](http://blog.brightinventions.pl/)
