# Smart Hub for Dumb Lights
This is a project that uses a Raspberry Pi to reproduce the signals of a 
radio frequency remote to control a set of patio lights.
Once successful transmission from the RPi to the lights was achieved, the project
then expanded to include a daily schedule in addition to on-demand signals,
essentially turning the RPi into a smart hub for "dumb" lights. 

This write-up is part tutorial and part story of a personal project.
If you wish to follow this as a tutorial, please note that it is not written
as a set of prescribed steps that must be strictly followed. Feel free to take
any or all sections as inspiration and adapt them to your own project.

### Contents
- [Background](#background)
- [Setting Up the MQTT Broker](#setting-up-the-mqtt-broker)
- [Setting Up Node-RED](#setting-up-node-red)
- [Writing the RF Transmit Script](#writing-the-rf-transmit-script)
- [Incorporating It All in Node-RED](#building-the-node-red-flow-and-incorporating-the-rf-tx-script)
- [Adding a Discord Bot](#creating-the-discord-bot)

> **NOTE:** This is a work in progress. Check back occasionally for updates. 
> #### Document Development status
> - [x] Outline
> - [ ] ***Rough Draft***
> - [ ] 2nd Draft
> - [ ] Final Draft 

## Background

> ### TL;DR
>
> I have a set of patio lights controlled by an RF remote. I wanted to control them outside the
> limitations of that RF remote. I developed a temporary fix using a Flipper Zero. I realized I
> could develop the full solution using a Raspberry Pi. 

Years ago I purchased a set of LED string lights for my back patio, very similar to
[model 56521 from Luminar](https://www.citylightsusa.com/luminar-outdoor-color-changing-led-string-lights-24-ft-of-string-12-bulbs-remote-control-outdoor-rated-56521/). They look like typical string lights with an aesthetically pleasing bulb like you would find in a restaurant or bar patio, with the added feature that they could change colors from commands sent by a radio frequency (RF) remote. I liked the idea of having them on for me when I came home late at night (I typically enter/exit my property through a gate on my back fence that connects to a bike path rather than through the front door), but if I left my house before the sun went down I would often forget to turn them on ahead of time. I tried for a bit to bring the remote with me, but it only works so far away since it uses RF signal and not Wi-Fi.
I started to keep the remote in my home office since I liked to have them on as I worked, but would often forget to turn them off until I was in bed about to go to sleep and would notice the light beaming in through my bedroom window, forcing me to then get out of bed and go into the other room to turn them off. I thought that If I could somehow connect the lights or the remote to my Wi-Fi network and then make a custom app to control it, I could turn my lights on/off from anywhere that has Internet connectivity. 

I looked into various smart outlet and smart switch products, but none of them fit my specific needs. They all would only control the on/off of the lights, and I wanted the ability to also control the colors. Furthermore, the RF receiver was at the end of the lights cable, in the same casing as the power adapter and plug. This was too far from my router to get a stable Wi-Fi connection. 

In late 2023, when the [Flipper Zero](https://flipperzero.one/) was going viral on YouTube and social media, getting banned in some places like Brazil and Canada, I thought this device might be a neat way to learn certain protocols and technologies. I was able to use the "Sub GHz" feature to capture the RF signals from my patio lights remote, save the signals in separate files, and replay them at will. This provided a great temporary solution to my situation. I essentially cloned the remote and could now leave the actual one in my office and the Flipper next to my bed. I could then control the lights from both locations. 

However, because the RF signal only traveled so far, I still could not turn on the lights from downtown if I realized I left long before the sun went down and it was now late and dark at my house. A friend then gifted me a Raspberry Pi and I realized that this could be the ticket to my solution. Based on some projects I had done at work, I realized that if I could host an MQTT broker on the Pi, and utilize Node-RED to process the messages, I could likely wire up the Pi to an RF transmitter to transmit the commands to the lights. I could then send the lights the commands from anywhere that I had an Internet connection. 

### The Plan
- Set up an MQTT broker on the Raspberry Pi and successfully communicate with it via my phone.
- Set up a Node-RED flow on the Pi to process the MQTT messages.
- Write a python script to send the RF commands to the lights from the Pi.
- Call the python script from the Node-RED flow based on the MQTT messages.

## Setting Up the MQTT Broker
[Eclipse Mosquitto](https://mosquitto.org/) is an open source message broker using the MQTT protocol. I decided to make this the core of communication for the lights. To get Mosquitto installed on the Raspberry Pi, I followed [this guide](https://randomnerdtutorials.com/how-to-install-mosquitto-broker-on-raspberry-pi/). It is a very straightforward and helpful walkthrough on not only how to get Mosquitto installed on the Pi, but also how to set it up so that the broker starts running automatically when the Pi boots up, as well as setting up authentication. 

I first set it up unauthenticated, with my `mosquitto.conf` file containing the following:
```
listener 1883 0.0.0.0
allow_anonymous true
```
I did this to easily test the connections and the auto-start and remote connections. 
There are various ways to test the connections. Personally, I followed [Steve's Guide on setting up a python client using Paho](http://www.steves-internet-guide.com/into-mqtt-python-client/). I set up two clients on separate devices (my laptop): a publisher and a subscriber. If the subscriber was able to read the message sent from the publisher, then I knew that the broker was working. 

Another way to test is to use [MQTT Explorer](https://mqtt-explorer.com/). This is a GUI based tool that can subscribe and publish to a broker and is very helpful in debugging. 

After confirming that the broker handled the messages properly and that I could attach both publish clients and subscribe clients, I then added the authentication as described in the [aforementioned article](https://randomnerdtutorials.com/how-to-install-mosquitto-broker-on-raspberry-pi/) and repeated the python and MQTT testing.

My final `mosquitto.conf` file looks like this:
```
# Place your local configuration in /etc/mosquitto/conf.d/
#
# A full description of the configuration file is at
# /usr/share/doc/mosquitto/examples/mosquitto.conf.example

per_listener_settings true

pid_file /run/mosquitto/mosquitto.pid

persistence true
persistence_location /var/lib/mosquitto/

log_dest file /var/log/mosquitto/mosquitto.log

include_dir /etc/mosquitto/conf.d

listener 1883
allow_anonymous false
password_file /etc/mosquitto/passwd
```

I could now access the broker and send/receive messages from anywhere on my home network. However, the end goal would be to control the lights from anywhere with an Internet connection, so I needed to set up port forwarding. 

### Port Forwarding
This turned out to be very simple. 
Access your router's interface by navigating a web browser to `192.168.0.1`. You will then be prompted to log in using your router's password. Once logged in, there should be an option in the settings interface to add port forwarding. My router is Arris. The Arris interface has the port forwarding option under the "Advanced" section of the left-hand navigation menu. From there, you can enable the port forwarding and add a service. This service should include a helpful name (so you know what it is a few months down the road when you open it again), the IP of the Pi (Server IPv4), and the port mapping (both the internal and external ports). The default port for Mosquitto is 1883. To make it simple, I just used the same port for both the internal and external. Note that the external port is the one that you will be accessing from the outside Internet, and the internal port is the one that the service is running on inside your network (on the Pi). 

![Port forwarding interface](images/rpilights/ARRIS_port_forwarding.png)

*An example of the port forwarding screen of my router's interface.*

Once the port forwarding is set up, you can then use MQTT Explorer to connect to the broker via your external IP (this can be found at https://www.whatsmyip.org/). To further prove that you can access the broker from anywhere with an Internet connection, download an MQTT app for your phone and disconnect your phone from your Wi-Fi so it's on your mobile service provider's data connection. Then test the broker connection through your phone app.

> Note: There are several MQTT apps out there for Android and iOS. Some of them are OK, but I really don't like any of them too much. I'll elaborate more on this later, but at this step it is worth mentioning that [MyMQTT](https://mymqtt.app/en) is the Android app that I found to be the most usable. 

A quick note on security. It's best practice to close all unused ports. Therefore, if you get all this up and running and set the project aside for a couple months like I did, it's a good idea to deactivate the service or simply disable port forwarding on your router altogether until you're ready to continue. 

## Setting Up Node-RED
Auto-start

## Writing the RF Transmit Script
### Determining the Correct Signials
- RF recieve Instructables Matplotlib technique
- Flipper file
- Flipper RAW file
### Confirming the Script Works
- Using Flipper to capture signal

## Building the Node-RED Flow and Incorporating the RF TX Script
### Confirming MQTT Connections with MQTT Explorer and Android App

## Creating the Discord Bot
### Replacing MQTT Android App with Discord
Remove port forwarding

## Expanding the Node-RED Flow
### Incorporating the Discord Bot
### Building the Schedule

## Future Expansion
### Add Self-Hosted DNS
So the MQTT broker doesn't get reasigned a new IP address
late at night when you are trying to go to sleep...
### Strengthen the RF Signal
So the RPi isn't as limited on possible locations 
### Add in IR Remote for Night Light
### Add Ceiling Fan Control
### Add Front Porch Light Control
Use Wireshark to analyze and decode the signals that the smart switch uses to control
the front porch light and try to reproduce those signals in the Node-RED flow
### Build an Android App

-----

[Home](https://sweisss.github.io/) &emsp; &emsp;
[OSU Projects](https://sweisss.github.io/#oregon-state-university-projects) &emsp; &emsp;
[Personal Projects](https://sweisss.github.io/#personal-projects) &emsp; &emsp;
[Element 1 Projects](https://sweisss.github.io/#element-1-projects) &emsp; &emsp;
[Videos](https://sweisss.github.io/#videos)

(C) Seth Weiss, 2025
