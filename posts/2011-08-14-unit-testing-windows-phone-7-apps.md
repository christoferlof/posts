---
title: Unit testing Windows Phone 7 apps
layout: post
tags: testing, winphone
excerpt: Unit testing your apps can help you ensure that you didn't break anything between versions.
publish date: 2011-08-14
---
# Unit testing Windows Phone 7 apps

Unit testing is in itself a big topic with lots of opinions on how and reasons for why and why not. I will not dig into the why part further than stating – I do it on my WP7 apps to ensure that I didn’t break anything. Until now I used to use [my own testing framework]({% post_url 2010-11-06-windows-phone-7-testing-extravaganza %}) – which is now, thanks to the introduction of the [Portable Library Tools](https://visualstudiogallery.msdn.microsoft.com/b0e0b5e9-e138-410b-ad10-00cb3caf4981), kind of obsolete.
Portable Library Tools is a new Visual Studio add-in from Microsoft which enables you to create C# and Visual Basic libraries that run on a variety of .NET-based platforms without recompilation, including Windows Phone. This enables us to create a Visual Studio project containing the phone agnostic code and reference it from both a phone project and a test project. But since the portable library doesn’t know about phone specifics we may need to make them available to the library at runtime through patterns like Dependency Injection , Service Locator and Factory.

## Getting started

In order to get started – the first thing would be to make sure you’ve the Portable Library Tools installed. They’re easy to find and install through the Extensions Manager in Visual Studio. When installed you should have a new project type called Portable Class Libraryavailable in the new project wizard. Then you’ll need three projects in your solution; The phone app-, the portable library- and the test- project. The phone app project and the test project should both have a reference to the portable library project.
In my first example I’m writing a very simple test just to make sure that the project setup works as intended – a calculator with an Add method. The calculator class is created within the library project and an unit test that simply invokes the Add method is created within the test project. Also I invoke the Add method from the phone app just to create a baseline knowing that the project setup is OK.
Green lights!

## Model-View-ViewModel-what?

Two of the drivers behind the Model-View-ViewModel (MVVM) pattern are testability and code reuse. By extracting the ViewModels from the phone project into the portable library project you will be able to test them and reuse them with ease.
For this next example I create a phone app that’s implementing the MVVM pattern by choosing the *Databound Application* template in the new project wizard. The template will create two simple view-models and tie them to their views. The next thing would be for me to move the view-models to the library project.
So far this portable library journey has been friction free. When moving the just created view-models to the library project the first perceived problem arise – Visual Studio complains about the type [ObservableCollection](http://msdn.microsoft.com/en-us/library/ms668604(v=VS.95).aspx). That’s because `ObservableCollection` live in the `System.Windows` assembly which isn’t referenced by the portable library project. If I remove the .NET Fx 4 target from the portable library project – I would be able to reference System.Windows but I wouldn’t be able to reference the portable library project from the test project since the portable library project then would be a SilverLight only library.

![No ObservableCollection](/assets/2011-08-14-unit-testing-windows-phone-7-apps/no-observablecollection.png)

I’m not changing the target setting because in this case I want to unit test my app. I wrote perceived above just to state that this really isn’t a big problem. The principle to solve the ObservableCollection challenge is the same as it would’ve been for the phone’s NavigationService class for instance. There’re things in the phone part of the framework that you probably need to consume in the view-models. The best way to gain access to phone specifics, like ObservableCollection and NavigationService, would be to fulfill the *dependency inversion principle*.

In order to create an abstraction layer I need to change the type of the members that are returning ObservableCollection to a type that’s known by all my projects and exposes the functionality needed. That type would be `IList<T>` in this case. The next thing I need is something that can provide me with an instance of ObservableCollection at runtime – since I can’t instantiate it in the view-model. Furthermore I need an instance of something that implements the above defined contract during test execution. This is the part where more choices and opinions about how to provide instances at runtime arise. There’re a couple of libraries and guidance out there which could prescribe how and aid me in wiring up the dependency injection. But for the simplicity of this example I chose not to bring in an external framework like that. A simple Factory will solve this for now.

The base implementation of the ListFactory will return instances of `List<T>` , which satisfies the above contract and is available on all targeted platforms. All I need to do is to make sure that the factory is instantiated before the tests run. In the phone project I add a new factory class which derives from the ListFactory base class and overrides the CreateList method in order to return an instance of ObservableCollection instead. The ObservableListFactory is instantiated once in my phone project’s App class and it will hold a reference to itself thru theCurrent property. With this approach I will be provided a list implementation that’s compatible with the current execution context.

> I can finally throw away my home brewed unit test fx  
&mdash; Christofer Löf

## Wrapping it up

Splitting up your Windows Phone app in two projects – the phone app project and a portable library project – should be your pattern of choice if you want to unit test your implementation.
Is MVVM and unit testing suitable for all WP7 projects?
It certainly adds complexity to your application implementation but one great benefit you’ll get from it is confidence – knowing that you didn’t break anything from release 1 to release 2. The path I usually take is to first create some kind of Proof of Concept of the app in mind in order to validate the overall idea and to run a few iterations on the end user experience. When I believe in the overall I start to refactor my way into the patterns and practices described in this article. In the end it’s the end-user experience that matters.