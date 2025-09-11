# Smart Hub for Dumb Lights
This is a project that uses a Raspberry Pi to control a set of patio lights
that can be controlled with a radio frequency remote (included with the lights).
Once successful transmission from the RPi to the lights was acheived, the project
then expanded to include a daily schedule in addition to the on-demand signals,
essentially turning the RPi into a smart hub for "dumb" lights. 

This write-up is part tutorial and part story of a personal project.
If you wish to follow this as a tutorial, please note that it is not written
as a set of prescribed steps that must be strictly followed. Feel free to take
any or all sections as inspriation and adapt them to your own project. 

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

## Future expansion
### Strengthening the RF Signal
### Adding in IR Remote for Night Light
### Adding Ceiling Fan Control
### Building an Android App

-----

This is a pseudo-footer

[Home](https://sweisss.github.io/)

(C) Seth Weiss, 2025
