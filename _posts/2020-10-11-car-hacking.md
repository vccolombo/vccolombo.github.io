---
title: "Car Hacking"
excerpt: "An overview of the car hacking scenario in the last decade."
date: 2020-10-11T17:51:30-03:00
categories:
  - Cybersecurity
tags:
  - Car Hacking
---

This post is an overview of the car hacking scenario in the past decade. I wrote it as part of a presentation I did in the [InfoSec group](https://ganesh.icmc.usp.br/) I'm a member of. Car hacking is becoming more important year after year, and the growth of autonomous vehicles calls for better security in our cars.

## Vectors of attack

### CAN Bus and ECUs

Cars' electronic components communicate over a serial bus communication protocol called Controller Area Network (CAN). These components are called Electronic Control Units, or ECUs. Modern cars have a lot of ECUs, for controlling things like the engine, audio system, breaks, airbags, etc.

The presence of a centralized bus is a potential vector of attack for cars. If a single ECU is compromised, it could be used to send malicious commands to other critical parts. It means that it's not even necessary to have physical access to the vehicle, as an ECU with wireless capabilities, like Bluetooth or cellular, could be hacked remotely and used as the attack vector.

Why is the CAN bus so insecure? Because [this architecture is 30 years old](https://www.sans.org/reading-room/whitepapers/ICS/developments-car-hacking-36607), when no one planned for remote car hacking decades in the future. As stated in the paper, there is no network segmentation in CAN, no built-in authentication, and no encryption, risk factors that might create a vulnerability scenario in modern car systems.

### OBD-II Devices (dongles)

OBD stands for Onboard Diagnostics and is a port present in cars that allow mechanics, insurance companies, truck fleets, etc, to plug a monitor device (called "dongle") on it, with the purpose of diagnosing problems, track user behavior, fleet localization, etc. These devices communicate directly with the car's internal network, meaning that vulnerabilities can lead to command injection in the vehicle. There are also privacy concerns, as these devices have tracking capabilities and could expose private information to unauthorized people.

These devices are especially dangerous because they often have wireless capabilities that could be exploited remotely.

### Key fobs and Remote Keyless System

Key fobs and cars that can be turned on without a key (Remote Keyless System) are another important attack vector. Cloned or exploited fobs can lead to a criminal entering the vehicle and stealing personal belongings. Also, keyless systems being hacked means that an attacker can just leave driving your car.

The technology of these devices is getting old, meaning that [it's cheaper than ever](https://www.securitymagazine.com/articles/91192-how-hackers-exploit-automotive-software-to-overtake-cars) to buy tools that can exploit them.

### Paired devices and mobile apps

In the ever-growing mobile world, it's starting to be very common to control everything with your phone, including cars. Mobile devices could be hacked (via malicious applications or phishing campaigns) and used to execute commands on cars if the phone is connected to it. This includes opening the doors, turning on the engine, and much more.

### Autonomous vehicles

Self-driving vehicles are a fresh technology, not fully implemented yet. However, the enormous amount of software and electronic components that goes into these cars make them a honeypot for hackers.

For example, sensors can be tricked to make the vehicle behave unexpectedly, like [changing road signs](https://ia.acs.org.au/article/2019/how-to-hack-a-self-driving-car.html) to trick the camera systems and confuse the decision making unity of the vehicle, potentially causing accidents.

Researchers used a simulation to predict that [hacking and disabling only 20% of the cars in Manhattan could cause a total deadlock in the city's traffic in rush-hour](https://www.futurity.org/hacking-cars-physics-2118772/).

## Real-life examples

### 2010 research from the University of Washington

In 2010, researchers at the University of California and the University of Washington published a [paper on how they were able to hack an automobile](http://www.autosec.org/pubs/cars-oakland2010.pdf) remotely. This is one of the first works in the area.

The attack consisted of a way to [manipulate the display and volume of the radio, temper with the car displays, unlock doors, and more](https://physicsworld.com/a/how-to-hack-a-self-driving-car/). A year later, [they showed it again](http://www.autosec.org/pubs/cars-usenixsec2011.pdf), now without physical access to the vehicle. By exploiting a mobile app used by drivers to execute some commands on the car remotely, an attacker would be able to ["locate, unlock and remote start"](https://www.reuters.com/article/us-gm-hacking-idUSKCN0Q42FI20150730) vehicles by intercepting and tempering with the app's communication with the car.

Read more here:

- [https://www.wired.com/2015/09/gm-took-5-years-fix-full-takeover-hack-millions-onstar-cars/](https://www.wired.com/2015/09/gm-took-5-years-fix-full-takeover-hack-millions-onstar-cars/)

### CAN packet injection

In 2013 Miller and Valasek wrote [this (great) paper](http://illmatics.com/car_hacking.pdf) showing a lot about car hacking. The first example that is shown is a packet injection in the CAN bus, where sending a simple packet to it alters the speedometer and odometer displays. This paper also shows a denial of service attack on the CAN bus that led to the steering wheel being partially locked, which is a way worse attack than just change the speed display. It also succeeded in apply breaks in a car equipped with anti-collision assistance. Take a look at page 35 onwards for more information about these and many other attacks, including tempering with the vehicle lighting, cutting the engine off, and more.

The attacks were shown using a physical attack on the bus, but it could be reproduced in a wireless scenario with a Bluetooth vulnerability for example. Read more about it:

- [https://www.forbes.com/sites/andygreenberg/2013/07/24/hackers-reveal-nasty-new-car-attacks-with-me-behind-the-wheel-video/#b4ced2e228c7](https://www.forbes.com/sites/andygreenberg/2013/07/24/hackers-reveal-nasty-new-car-attacks-with-me-behind-the-wheel-video/#b4ced2e228c7)

### Hacking a car with text messages

In 2015, [researchers showed](https://www.wired.com/2015/08/hackers-cut-corvettes-brakes-via-common-car-gadget/) that they could hack a car with simple text messages, by exploiting a security flaw in a car dongle and gaining access to the car's CAN bus. With this, they were able to perform actions like braking, turning on the wipers, and more, all of this remotely. This dongle was used by companies like Uber, and even U.S. Federal Agencies.

And it was only a proof-of-concept! They claim "they could have hijacked the steering or brakes of just about any modern vehicle with the Mobile Devices dongle plugged into its dash". The vulnerability was later fixed.

### Insurance company dongle

Another vulnerable dongle example. This one was used by an insurance company to track its clients' behavior. [It was found that this specific device had several vulnerabilities](https://www.darkreading.com/vulnerabilities---threats/security-mia-in-car-insurance-dongle/d/d-id/1318644), including [usage of insecure protocols, unauthenticated communication with cellular devices, not validated/signed firmware, and more](https://www.forbes.com/sites/thomasbrewster/2015/01/15/researcher-says-progressive-insurance-dongle-totally-insecure/#673eb7c71772).

Remotely attacks could be done, like man-in-the-middle (as the traffic was not encrypted), and sending remote commands to the car.

### "A remote attack on an aftermarket telematics service"

As you can see, there is a lot of attacks on dongle devices. [This one](https://argus-sec.com/remote-attack-aftermarket-telematics-service/) communicated using the non-secure HTTP protocol and didn't sign its updates. This allowed hackers to remotely update the device's firmware, effectively taking control of it and being able to send malicious commands to the CAN bus, leading to car hijacking. Also, it could be used in more subtle ways, like spoofing the vehicle's GPS data, driver behaviors, and more.

### 2015 Jeep Cherokee remote hack

Miller and Valasek strike again, now in a remote hacking. This attack is very famous, as it got a lot of attention from the press and led to [more than a million cars getting recalled](https://www.bbc.com/news/technology-33650491) by Jeep.

First, the researchers found a vulnerability in the multimedia system of the car was flawed. Its wi-fi password generation was not random, allowing malicious agents to easily brute-force it. With this, they could control the entire multimedia system. However, the multimedia system was not directly connected to the CAN bus, so a more troublesome attack was not possible.

However, they found out that the multimedia system was connected with another ECU. This component had access to the CAN, so it could send malicious commands to other components in the car. All they had to do was to update this component's firmware, using the exploit previously found in the multimedia. And voila, everything was compromised.

Read more here:

- [https://www.kaspersky.com/blog/blackhat-jeep-cherokee-hack-explained/9493/](https://www.kaspersky.com/blog/blackhat-jeep-cherokee-hack-explained/9493/)
- [https://www.wired.com/2015/07/hackers-remotely-kill-jeep-highway/](https://www.wired.com/2015/07/hackers-remotely-kill-jeep-highway/)
- [https://www.sans.org/reading-room/whitepapers/ICS/developments-car-hacking-36607](https://www.sans.org/reading-room/whitepapers/ICS/developments-car-hacking-36607) page 20

### 2016 Tasla Model S

In 2016, Chinese researchers were able to hack a Tesla Model S from up to 20 kilometers (12 miles). [The demonstrated attacks included adjusting the mirrors, tampering with the locks, and even slamming on the breaks](https://www.theguardian.com/technology/2016/sep/20/tesla-model-s-chinese-hack-remote-control-brakes).

Again, this was a CAN bus attack. This time, a vulnerability in the car's web browser, when connected to a Wi-Fi hotspot controlled by the attackers, could be exploited to send commands to the car's internal network.

Tesla quickly fixed the vulnerabilities.

### Tesla Model S Key Fob

It was found out that the Tesla Model S key fob [could be easily cloned, and it could lead to an attacker to just drive away with the car](https://www.cnet.com/roadshow/news/tesla-model-s-key-fob-hack/).

## References

- https://www.csselectronics.com/screen/page/simple-intro-to-can-bus/language/en
- https://people.kth.se/~kallej/papers/can_necs_handbook05.pdf
- http://illmatics.com/car_hacking.pdf
- https://argus-sec.com/remote-attack-aftermarket-telematics-service/
- https://www.theguardian.com/technology/2015/aug/12/hack-car-brakes-sms-text
- https://www.wired.com/2015/08/hackers-cut-corvettes-brakes-via-common-car-gadget/
- https://www.washingtonpost.com/cars/corvette-brakes-hacked-by-researchers-using-text-messages/2015/08/12/45b8feb6-4110-11e5-9f53-d1e3ddfd0cda_story.html
- https://www.theguardian.com/technology/2015/aug/12/hack-car-brakes-sms-text
- https://www.darkreading.com/vulnerabilities---threats/security-mia-in-car-insurance-dongle/d/d-id/1318644
- https://argus-sec.com/remote-attack-aftermarket-telematics-service/
- https://www.sans.org/reading-room/whitepapers/ICS/developments-car-hacking-36607
- https://www.securitymagazine.com/articles/91192-how-hackers-exploit-automotive-software-to-overtake-cars
- https://arstechnica.com/information-technology/2010/05/car-hacks-could-turn-commutes-into-a-scene-from-speed/
- https://www.nytimes.com/2011/03/10/business/10hack.html?_r=0
- https://www.forbes.com/sites/thomasbrewster/2015/01/15/researcher-says-progressive-insurance-dongle-totally-insecure/#673eb7c71772
- https://www.welivesecurity.com/2016/09/21/tesla-model-s-hack/
- https://www.theguardian.com/technology/2016/sep/20/tesla-model-s-chinese-hack-remote-control-brakes
- https://www.bbc.com/news/technology-33650491
- https://www.kaspersky.com/blog/blackhat-jeep-cherokee-hack-explained/9493/
- https://www.wired.com/2015/07/hackers-remotely-kill-jeep-highway/
- http://www.autosec.org/pubs/cars-oakland2010.pdf
- https://www.wired.com/2015/09/gm-took-5-years-fix-full-takeover-hack-millions-onstar-cars/
- https://www.reuters.com/article/us-gm-hacking-idUSKCN0Q42FI20150730
- https://www.cnet.com/roadshow/news/tesla-model-s-key-fob-hack/
- https://ia.acs.org.au/article/2019/how-to-hack-a-self-driving-car.html
- http://www.autosec.org/pubs/cars-usenixsec2011.pdf
- https://www.futurity.org/hacking-cars-physics-2118772/
- https://www.forbes.com/sites/jamiecartereurope/2019/03/05/hacked-driverless-cars-could-cause-collisions-and-gridlock-in-cities-say-researchers/#7f7a6ae02a09
