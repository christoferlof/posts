---
title: Driver's log for Windows Phone
layout: post
tags: winphone, app
---
As a consultant I travel quite a lot by car. By the end of every month I need to fill in our corporate Excel spreadsheet with my mileage and send it to payroll in order to get reimbursed.

When I received by HTC Mozart a couple of weeks ago I immediately felt the urge to develop for it and get to know the platform. So I thought for myself – why not smash two birds with the same stone and learn something while I build something that I need?

Said and done – I named the application “Driver’s log”. It allows me (and you, since it’s available on the Marketplace) to keep track of my travels and car mileage and associated expenses like gas and parking. By the end of the month I export my log through e-mail and then easily copies the entries from the e-mail into the form my manager expects. (“Convince manager to accept Driver’s log export format” is on my to-do list). This saves me some time but more important – since it’s fast and easy – it ensures that I really report all trips and by the end of the day it means that I get accurately reimbursed (money saved).

![Driver's Log App](/assets/2011-03-11-driverslog-for-windows-phone/driverslog1.png)

So how did I implement Driver’s log?
On the framework side I took a dependency on Caliburn.Micro to do proper unit testing with VFx. Caliburn brings – among other things – Windows Phone API abstractions, Dependency Injection Container and Messaging to the table. This allowed me to clearly separate the Phone UI and Phone specific services from the core of the Driver’s log application – a necessity for unit testing the model outside of the emulator.
I also took a dependency on the Silverlight Toolkit. It provides Driver’s log with nice date pickers and touch effects.
At the time of writing this Driver’s log is at version 1.2. I have a few more things on my list that I want to implement. Besides from them I want to hear from you. Please post your questions, concerns, requests, criticism, praise and ideas in the comments section below.

[![Download in the Store](/assets/2011-03-11-driverslog-for-windows-phone/wp7_English_300x50_red.png)](http://www.windowsphone.com/en-us/store/app/driver-log/804f52f1-9334-e011-854c-00237de2db9e)