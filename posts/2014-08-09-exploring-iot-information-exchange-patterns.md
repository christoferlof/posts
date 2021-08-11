---
title: Exploring IoT information exchange patterns with Azure Service Bus and Gadgeteer
layout: post
tags: iot, patterns
---
# Exploring IoT information exchange patterns with Azure Service Bus and Gadgeteer

In the article [Service Assisted Communication for Connected Devices](http://blogs.msdn.com/b/clemensv/archive/2014/02/10/service-assisted-communication-for-connected-devices.aspx) Clemens Vasters discuss challenges with different approaches for exchanging information with special-purpose devices - The type of devices which we today like to group into the term 'Things' in 'Internet of Things'. Addressing and security being two of the main concerns in many of today's implementations.

Service Assisted Communication addresses these concerns by introducing an intermediary which will ensure secure and reliable delivery of information to and from the connected devices. The devices will not allow inbound connections but only outbound to the intermediary services, which holds a queue of messages intended for the particular device.
Message based information exchange is widely used in today's architectures of distributed solutions and with today's technology and protocols like AMQP, Internet based messaging is available to even the smallest devices.

The Azure Service Bus is a platform service on which you can implement the communication intermediary in an IoT scenario. In this post I'm being a bit more concrete to see how the different messaging patterns could be implemented and communicating with a device that's more constrained than my phone and laptop.

Before going to the different scenarios I believe it would be a good idea to have a little look on the device I used for this lab.

## .NET Gadgeteer

Microsoft .NET Gadgeteer is an open-source toolkit for building small electronic devices using the .NET Micro Framework. There are several hardware kits and components available that implements Gadgeteer and they are excellent for rapid prototyping like the work I'm doing here.

I ordered the GHI Electronics FEZ Spider Starter Kit from Watterott. The Starter Kit has more components than I needed for this lab but I thought it would be great to have some modules at hand to choose from for upcoming labs as well.

If you're buing a FEZ Spider I recommend you to read through the getting started guide from GHI and do the 'Hello World' app. It took me some hours to get firmwares to the right version,  enable SSL, installing .NET MF kits and so on.

The device I created for this lab is pictured below and consists of the following modules

* Ethernet
* Button
* Joystick
* Multicolor LED
* Display

![FEZ Spider main board with components attached, ready for action](/assets/iot-patterns-gadgeteer/device.jpg)

## The Scenarios

Just to keep things simple I chose the domain of 'Connected Cars' and then made up one scenario for each messaging pattern. There are four major messaging patterns in this context; Telemetry, Inquiry, Command and Notification.

### Telemetry - G-Force

*Information flowing from a device to other systems for conveying status of device and environment.*

The joystick on the device simulates a multi-axis accelerometer that measures g-forces. The device reads the joystick position twice every second and transmits that information to the event ingestion topic on the Service Bus. The telemetry events are listed and shown in a spline graph in a web app.

The telemetry data could then be used to identify the driving style and give feedback to the driver on how to improve the fuel consumption rate and how to drive more environmental friendly.

If a force is greater than 90% (of joystick travel) the LED on the device will blink red to indicate a harsh move (turn, braking etc).

![List showing raw telemetry evenst and graph plotting G-Forces over time](/assets/iot-patterns-gadgeteer/telemetry.jpg)

### Inquiry - Alert on harsh move

*Requests from devices looking to gather required information or asking to initiate activities.*

When pressing the button on the device, an inquiry message is sent asking whether to alert the driver on a harsh move (blink LED). Once the inquiry is received a pane is displayed in the web app allowing the user to respond with a 'Yes' or 'No'. The device will then update its configuration based on the response.

![The device asks for input](/assets/iot-patterns-gadgeteer/inquiry.jpg)

### Command - Unlock car

*Commands from other systems to a device or a group of devices to perform specific activities.*

A command to unlock the car can be sent from both the web app and the companion app for Windows Phone. Just as the inquiry message a command expects a response to determine whether the command was executed or not. In addition, it has a time-to-live time set on it. So if the command hasn't been delivered to the device before the time runs out, it will not be delivered at all. In this scenario where the car should be unlocked we want the car to be unlocked immediately or never. It wouldn't be a very pleasant experience if the car was unlocked one hour later when I've left the parking lot in frustration and anger.

![Send a command to fade the LED. Any response from the device is shown to the right](/assets/iot-patterns-gadgeteer/command.jpg)

### Notification - Traffic news

*Information flowing from other systems to a device or a group of devices for conveying status changes in the rest of the world.*

A notification can be sent from the web app and is simply shown on the display once it's received by the device. This could be weather or traffic information or something similar.

### Pair companion app with car

Unlike the above scenarios, this one isn't associated with a particular messaging pattern.

The device displays its device identifier as a QR Code on the display. Using the companion app on my smartphone I scan the QR Code to connect with the device. (Obviously layers of authentication and security needs to be in place here but were omitted for the simplicity of the lab). Once paired I can use my smartphone to unlock the car and to digest the feedback on my driving style.

### The solution

![The solution](/assets/iot-patterns-gadgeteer/solution.jpg)

The intermediary service is implemented using Azure Service Bus Topics and Subscriptions with messaging over AMQP. A SignalR hub exposes an API which simplifies interaction with the connected devices. The API will always be available and addressable regardless of the connectivity state of the actual target device. The reason for chosing SignalR over Web API in this scenario was simply to enable notifications for the Web UI and Companion App in a simple fashion.

```csharp
// SignalR API to send the Fade Command to the device
public void SendCommand(int fadeDuration, string respondTo)
{
  var message = new BrokeredMessage();
  //only valid for 30sec
  message.TimeToLive = new TimeSpan(0,0,0,30);
  message.Properties.Add("message-type", "command");
  message.Properties.Add("fade", fadeDuration);
  message.Properties.Add("respondTo", respondTo);
  commandClient.Send(message);
}
```

The device communicates with Service Bus using the AMQP.Net Lite library. When using AMPQ with Service Bus you must ensure that you don't create partitioned queues or topics since AMQP doesn't support partitioned entities.

```csharp
private void InitializeReceiveLink() {
	// Initializing receive link using AMQP.NET Lite and .NET MF
	receiverConnection = new Connection(address);
	receiverSession = new Session(receiverConnection);
	receiver = new ReceiverLink(receiverSession, "receive-link" + InEntity, InEntity);
	receiver.Start(50, OnInboundMessage);
	receiver.OnClosed += (o, error) => TraceWrite("Receive Link Closed", error);
}
```

[The full source code is available on GitHub.](https://github.com/christoferlof/IoTMessageExchangePatterns)

## Wrapping up

In this lab I didn't pay any attention to security aspects like authentication or authorization. In addition, I didn't touch on scalability nor service availability. These are crucial aspects that needs to be factored into a real-world connected devices solution.

Microsoft recently announced the preview availability of [Event Hubs](http://azure.microsoft.com/en-us/services/event-hubs/). Event Hubs is an telemetry ingestion service on top of Service Bus which will handle millions of telemetry events per second from your devices and services. So instead of using a simple topic (or group of topic) for event ingestion you should  have a look at Event Hub.

## Resources

* [Service Assisted Communication for Connected Devices](http://blogs.msdn.com/b/clemensv/archive/2014/02/10/service-assisted-communication-for-connected-devices.aspx)
* [Using Microsoft Azure Service Bus for ... Things!](http://msdn.microsoft.com/en-us/magazine/jj133819.aspx)
* [AMQP.Net Lite](http://amqpnetlite.codeplex.com/)
* [Service Bus AMQP: Developers Guide](http://msdn.microsoft.com/en-us/library/jj841071.aspx)
* [GHI .NET Micro Framework packages](https://www.ghielectronics.com/support/netmf)
* [Getting started with the FEZ Spider kit for .NET Gadgeteer](http://www.ghielectronics.com/downloads/Gadgeteer/Mainboard/Spider/FEZSpider%20Starter%20Kit%20Guide.pdf)