---
title: Portable Class Library and ICommand
layout: post
tags: testing, winphone
publish date: 2012-03-13
---
# Portable Class Library and ICommand

A while ago I [wrote an article](2011-08-14-unit-testing-windows-phone-7-apps.md) describing how the [Portable Class Library](https://visualstudiogallery.msdn.microsoft.com/b0e0b5e9-e138-410b-ad10-00cb3caf4981) can enable us to write Unit Tests for our Windows Phone apps using the tools and frameworks we’re already familiar with. By adding an abstraction layer and using types common to the Base Class Library, Portable Class Library and Windows Phone we can write unit tests using tools like MSTest and Moq.

## ViewModels and ICommand

A popular approach for writing Windows Phone apps is to use the Model-View-ViewModel pattern and by that expose operations on the View Model as commands which can be data bound to buttons on the view. The interface which enables commands, ICommand, isn’t available in the Portable Class Library thus exposing that kind of commands from our 

## BindingConverter to the rescue

Instead we can implement a BindingConverter which allow us to dispatch the invocation of a command to a method on the View Model. So far this approach seems to work really well and when giving this solution a second thought I really believe it improved the architecture by separating the concerns even further – the view specific ICommand is no longer part of the model library.

```xaml
<Button 
   Content="Button text" 
   Command="{ Binding 
      Converter={ StaticResource CommandConverter }, 
      ConverterParameter='ViewModelMethodName' }" 
   CommandParameter="MethodParameter" />
```

Thanks to [Johan Lindfors](https://twitter.com/JohanLindfors) who initiated this discussion.

## Visual Studio "11"

Today I learnt that Visual Studio “11” Beta has support for ICommand when targeting Windows Phone, Silverlight, Metro and .NET 4.5. Pure awesomeness.

<blockquote lang="en"><p><a href="https://twitter.com/chrislof">@chrislof</a> Just in case you missed it, we&#39;ve added support for ICommand in Visual Studio 11 Beta when targeting Phone, SL, Metro + .NET 4.5.</p>&mdash; David Kean (@davkean) <a href="https://twitter.com/davkean/status/179783353560600576">March 14, 2012</a></blockquote>
