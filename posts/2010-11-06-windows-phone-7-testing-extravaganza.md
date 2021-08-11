---
title: Windows Phone 7 Testing Extravaganza!
layout: post
tags: testing, winphone
---
# Windows Phone 7 Testing Extravaganza!

During my parental leave this summer I had some moments which I could spend on coding-for-fun. Like a whole lot of others these days I began to write a couple of Windows Phone 7 apps. Among the first things I looked into was the availability of testing frameworks especially suited for Windows Phone development. Back then I found one – the silverlight toolkit unit testing framework. It’s an impressive piece of work. But in my humble opinion it lacks, currently, two major features. The first feature is; execute unit test outside of the emulator or the device i.e. on your computer.

For a TDD style of development having to deploy to the emulator to execute one test just eats too much of my precious time. I’ve understood that the APIs of the Silverlight Unit Test Framework are compatible with the VSTest APIs so linking files between projects would probably be one way to execute the tests on the development machine. But that would force me to create shadow projects to link files and execute the tests outside from the Silverlight runtime – I don’t want that. The second feature I believe the Silverlight Testing Framework is lacking, which I really thought was there, is support for UI tests. So. When you’re a guy who likes to write frameworks and has some spare time. What do you do? You roll your own testing framework of course!

## Unit Tests
The amount of code that is required to execute tests within the Silverlight runtime from a console app isn’t too many. The number of moving parts within the solution are a few more though. The console app hosts a Browser control. The source URL of theWebBrowser control is set to a static HTML page (the RunnerPage) which hosts the generated xap file. The console app calls a JavaScript function in the HTML page using theInvokeScript method of the Browser control. The JavaScript in the HTML page executes the TestRunner within the xap package by calling a method exposed by the Silverlight JavaScript Bridge. The key parts here are the two different JavaScript bridges exposed by the WebBrowser control and by Silverlight.
The console app creates the xap package on the fly by examining the Package folder and include all assemblies found in there. The resulting xap is copied to the Content folder, where the RunnerPage expects it to be.
Writing the Unit Tests is as simple as you might expect. Create a Windows Phone 7 class library, add your tests and mark them with the Fact attribute.
To run the tests you have to copy the test assembly to the Package folder as previously described. To do that you could set up post build events in Visual Studio or just write a simple script to handle the copying and executing of tests for you. I’m currently using my new best friend [psake](https://github.com/JamesKovacs/psake) to handle all that for me. It allows me to just write `.\test.ps1` in a PowerShell in order to build and unit test my apps.

```csharp
[Fact]
public void should_set_id_on_save() {
    
    var item = new TodoItem();
    
    TodoItem.Add(item);
    TodoItem.SaveChanges();

    Assert.True(item.Id!=null);
}
```
## UI Tests
The UI test (or Integration tests as I call them in the Framework) functionality is at this point very immature. My goal here is to run the UI using the Silverlight Automation APIs. For the basic controls like Buttons and TextBoxes this isn’t a problem. But for the ApplicationBar, for instance, it’s harder because that control isn’t a Silverlight Control and hence it lacks automation support. I believe the UI test support in the framework will be a mix of Silverlight Automation and a pinch of reflection magic by relying on some naming conventions. (It’s true – it really will be magic. Promise.)
Lately I’ve used the [Lightweight Test Automation Framework](http://aspnet.codeplex.com/wikipage?title=ASP.NET%20QA&referringTitle=Home) by the ASP.NET QA team in a bunch of projects and I really like the APIs it provides for writing tests. So it’s not a coincidence that the VFx integration test APIs reminds you of LTAF.

The UI tests are run with the same runner as the unit tests. Currently the tests have to live in the app they are testing and the runner is executed by defining the INTEGRATIONTEST compilation symbol. This is clearly something I want to change in the future.

```csharp
[Fact]
public void should_create_todo() {
    Page("/CreatePage.xaml").Ready(p => {
        p   .Find<TextBox>("TitleTextBox")
            .SetText("New Todo");
        p   .Find<Button>("CreateButton")
            .Click();
        p   .Find<TextBox>("IdTextBox")
            .WaitForText(text => 

        Assert.True(text!=null));
    });
}
```
Download or fork: [https://github.com/christoferlof/VFx](https://github.com/christoferlof/VFx)