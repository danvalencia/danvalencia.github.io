+++
date = "2013-08-08T10:56:50-07:00"
tags = ["burning man", "hands", "arduino", "ios",  "ipad"]
title = "Hands (Part I)"

+++
I'm currently helping my friend Dave Gertler to build his Burning Man art installation which is called [**Hands**][1].  Specifically I'll be focusing on the electronics, which is a fairly big chunk of work.  In this post I'll introduce you to the project and show you some of the work we've been doing to make this a reality.

### Intro
The installation consists of 2 hands with the palms facing upwards.  The hands will be made out of CDX plywood and the piece is designed to be assembled as a huge 3D jigsaw puzzle. In addition, multiple series of LEDs will illuminate the hands. Check out the following scale model to have a better idea of what it looks like:

![Hands Scale Model][2]

The project is super ambitious: each hand will be 12 ft. tall and we've calculated more than 250 LEDs will be required.  Dave has given a _lot_ of thought to the structural design of the piece; given that people at Burning Man will undoubtly want to climb on the art pieces, the structure is designed to fit up to 18 people! _Per Hand!_ Given the irregular shape of the hand this probably won't be possible in practice, but the structure will be strong enough to account for a lot of weight.

The following images illustrate the dimensions of the installation with respect to a human being:

<table>
<tr>
<td><img src="http://s3.amazonaws.com/danvalencia_my_site/IMG_2175.JPG"></img></td>
<td><img src="http://s3.amazonaws.com/danvalencia_my_site/IMG_2176.JPG"></img></td>
</tr>
</table>

### The Electronics

The idea is that these Hands are illuminated with a series of LEDs that are individually addressable and can be programmed with different algorithms, similar to the [Bay Bridge Lights][7]. Ok, probably not as sophisticated as the Bay Bridge Lights, but you get the idea.

For _Hands_ I've chosen to use an [Arduino][3]  as the _brains_ of the project for the following reasons:

 - It's very easy to use.  
 - It's open source and there are tons of code samples in the web.
 - Extensible architecture in the form of plug-in boards or _shields_ make it possible to add functionality such as WiFi or Bluetooth fairly easily.

After researching for a little while we found a perfect step by step tutorial by [Adafruit][4] for controlling strands of 36mm LED Pixels with an Arduino.  The LEDs were perfect, as they come coated for outdoor use and offer several mounting options.  We ordered some LEDs strips and started playing with 'em immediately! Check out our first steps with the LEDs in the following video:

<iframe src="http://player.vimeo.com/video/61054270" width="500" height="282" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>

### Features and Human Interaction

Human interaction is key for any Burning Man art installation and _Hands_ is no exception.  We thought about different ways in which people could interact with the LEDs. One thought we explored is the ability to control the LEDs via an iPhone or iPod.  

In order to make this happen, we needed a way to establish wireless communication between the Arduino and an iOS device.  Bluetooth seemed like an obvious choice and after surfing the web for a while I found the amazing Arduino Bluetooth LE Shield by [Red Bear Lab][5].  

Until recently, Bluetooth support in iOS devices was restricted to Apple's [MFI][6] program, which made it practically impossible for hobbyists to use.  Luckily for us, this changed, and starting with iOS 6, Apple released an SDK for Bluetooth 4.0 devices (called CoreBluetooth).  

The Bluetooth shield comes with sample code for both the Arduino and iOS side of things. We realized that this was a very viable solution and decided to build an iOS app to interact with the LEDs.

I think I'll save the details of my progress with the app for my next post. For now, check out the following video which demonstrates some of the progress I've made:

<iframe src="http://player.vimeo.com/video/68868345" width="500" height="281" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>

Stayed tuned for more Arduino, iOS and Burning man fun!


  [1]: http://handsburningman.blogspot.com/
  [2]: http://s3.amazonaws.com/danvalencia_my_site/IMG_2179.JPG
  [3]: http://www.arduino.cc/
  [4]: http://learn.adafruit.com/36mm-led-pixels/overview
  [5]: http://redbearlab.com/bleshield/
  [6]: https://developer.apple.com/programs/mfi/
  [7]: https://vimeo.com/25870560
