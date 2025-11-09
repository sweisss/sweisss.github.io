# Smart Hub for Dumb Lights
This is a project that uses a Raspberry Pi to reproduce the signals of a 
radio frequency remote to control a set of patio lights.
Once successful transmission from the RPi to the lights was acheived, the project
then expanded to include a daily schedule in addition to on-demand signals,
essentially turning the RPi into a smart hub for "dumb" lights. 

This write-up is part tutorial and part story of a personal project.
If you wish to follow this as a tutorial, please note that it is not written
as a set of prescribed steps that must be strictly followed. Feel free to take
any or all sections as inspriation and adapt them to your own project.

### Development status
- [x] Outline
- [ ] ***Rough Draft***
- [ ] 2nd Draft
- [ ] Final Draft 

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
### Auto-start
### Security/access
### Port Forwarding

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
