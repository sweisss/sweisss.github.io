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

## Background
- The lights
- The issue (only one remote, but the desire to turn the lights on/off from multiple locations)
- The Flipper Zero temporary solution

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
### Add Font Porch Light Control
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
