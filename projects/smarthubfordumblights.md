[Home](https://sweisss.github.io/) &emsp; &emsp;
[Personal Projects](https://sweisss.github.io/#personal-projects) &emsp; &emsp;
[OSU Projects](https://sweisss.github.io/#oregon-state-university-projects) &emsp; &emsp;
[Element 1 Projects](https://sweisss.github.io/#element-1-projects) &emsp; &emsp;
[Videos](https://sweisss.github.io/#videos)

-----------

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

The GitHub repository for the project is located [here](https://github.com/sweisss/rpi-smart-hub).

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
Node-RED is a "flow-based, low-code development tool for visual programming, originally developed by IBM for wiring together hardware devices, APIs and online services as part of the Internet of things." ([Wikipedia](https://en.wikipedia.org/wiki/Node-RED))

Node-RED came preinstalled on my Raspberry Pi, and it likely is on yours too. In the event that it isn't, visit the [official Node-RED documentation](https://nodered.org/docs/getting-started/raspberrypi) to get it installed. That article also says that you can set up Node-RED to autostart on system boot of the Pi with the following command:
```
sudo systemctl enable nodered.service
```
I could not get it to work using this command. Instead, I wrote a very simple bash script that contained nothing more than the line `node-red` and saved it to `/home/raspberry/Documents/scripts/node_red_startup.sh`. 

I then opened up my crontab in the nano editor with the command `crontab -e` and added the following line to the end of the file:
```
@reboot sh /home/raspberry/Documents/scripts/node_red_startup.sh
```
I saved the file and rebooted the Pi and it worked. 

To view the Node-RED interface, open the web browser to http://localhost:1880. From there you can make a very simple flow to subscribe to your broker and output the message contents to the debug window. With this step complete, I then shifted my focus to transmitting the RF signals using the Pi. 

## Writing the RF Transmit Script
### Determining the Correct Signials
As mentioned in the [Background](#background) section above, I had already cloned the main RF signals of the remote (on, off, white, red, and blue) using the Flipper Zero. I was able to easily get the files for these cloned signals onto my computer using the [qFlipper app](https://docs.flipper.net/zero/qflipper). The files contain several pieces of information about the signals, but the most important parts are the protocol (e.g. Princeton 24bit), the frequency (**NOTE TO AUTHOR: Or is it bandwidth??**) (e.g. 433.92 AM), and the "key", which is a hex representation of the signal pattern (**NOTE TO AUTHOR: Confirm this is accurate and get more official terminology**). 

The Flipper actually tells you everything you need to reproduce the signal. However, I couldn't find explicit documentation on understanding and reproducing a Flipper Sub-GHz file on the Raspberry Pi, so I used some other techniques and followed some other guides to confirm I had the correct information.

> NOTE TO AUTHOR: Maybe put some screenshots in here of the files.

Many of the guides and tutorials I found online also set up a way to receive the RF signals on the Raspberry Pi. Luckily, most of the hardware devices come in packs with multiple sets of both transmitters and receivers. Just search amazon (or better yet, the Internet outside of amazon) for "433 MHz transmitter" and you should find a number of packs. I believe I settled on [this one](https://www.amazon.com/D-FLIFE-Wireless-Transmitter-Receiver-Antenna/dp/B0BZRRBBNK/ref=sr_1_5).

> NOTE TO AUTHOR: Get the actual device pack, maybe even model number.

The first way I confirmed the information on the Flipper file was to follow an article on Instructables titled [Super Simple Raspberry Pi 433MHz Home Automation](https://www.instructables.com/id/Super-Simple-Raspberry-Pi-433MHz-Home-Automation/). Unfortunately, it appears the article is no longer up as of this writing (Dec, 2025), so I'll give a brief summary here.

The article is broken into two main sections: receive and transmit. The first section, receiving the signal, wires up the RX hardware to the Pi and uses a script to capture the RF signal and graph the results using Matplotlib. It then explains that an RF signal works by interpreting a pattern of high/low signals. The pattern is made of a combination of short signals and long signals, often preceded by a preamble to let whatever is receiving the signal know that what's about to follow is the data. Different protocols will use different combinations of high/low and long/short signals to signify a 1 or a 0. To decode the signal, you just need to view the pattern and write down either a 1 or a 0 each time you see a long-high to short-low or a short-high to long-low, for example. It may sound like this method is tedious and not guaranteed, which is correct. However, after going through it multiple times I ended up with consistent results. I used an online binary to hex converter to translate the binary code I kept getting from the graph, and it matched up with the hex value in the Flipper file!

> NOTE TO AUTHOR: Review how the RF signals work and write up a better explanation. Don't just link the reader to external articles and videos.

If you're interested in diving deeper into the analysis of the RF signal, by all means, go ahead. It's actually really interesting. I found a number of YouTube videos by [Derek Jamison](https://www.youtube.com/@MrDerekJamison) that dive deep into the Flipper Zero and how it analyzes RF signals. [This one](https://www.youtube.com/watch?v=ojpc7Q2fjS8) in particular is a great one to start with and explains how the signals work (probably in a better way than I just did). Some of his other videos also explain the "Read RAW" feature, which was actually another way that I confirmed I was using the correct signal.
However, for the purpose of this project, these were just a ways to confirm that the hex value from the "key" of the Flipper file was actually the signal that I would need to transmit to the lights. 

Once I was confident about the signal I needed to send, I then swapped out the RX hardware for the TX hardware and wired it up to the Pi. I actually got hung up on this part of the project for quite a while. Looking back, this was definitely the crux of the whole project for me. 

> NOTE TO AUTHOR: Describe how the hardware is wired to the Pi. Find a good picture of the pinout.
> Also use pictures of the final setup.

### Confirming the Script Works
Getting the timing right for the signal lengths (short vs. long in ms) proved to be pretty tricky. However, after some trial and error and a little help from some AI to analyze my flipper files and my working python script, I landed on a python library to do the heavy lifting of the RF transmission. The library is `rpi_rf`. If I recall correctly, this library was pre-installed on my RPi. If it's not on yours, you can install it with `pip install rpi-rf`. Notice the dash (`-`) in the pip install command as opposed to the underscore (`_`) that should be used in the import statement (`from rpi_rf import RFDevice`). Documentation for this library can be found on [pypi](https://pypi.org/project/rpi-rf/) and [GitHub](https://github.com/milaq/rpi-rf).

To test that I was sending the correct signal, I set up my Flipper Zero to listen for RF signals next to my RPi transmitter. After working out the bugs in my script the Flipper sounded the alert that it had captured a signal. I checked the hex value of it and it matched the one I sent from the python script! Since the lights did not turn on when I successfully captured this signal, I assumed the transmitter was not powerful enough to reach the RF receiver on the patio lights. Maybe it was my poor soldering skills, or maybe it was my lack of knowledge, but every time I tried to solder an antenna to a transmitter to extend the range I seemed to fry it. I then found a workaround by finding a spot in my bedroom that was close enough to the patio lights receiver for the RF signal to reach it, yet close enough to the wall of my home office so that I could reach the Pi from the keyboard and monitor in my office via a bluetooth connection (this makes it much easier to work on, since my bedroom isn't really set up with a desirable space for computer work).
> Pro Tip: I found a bluetooth [keyboard with built-in touchpad](https://www.amazon.com/Bnnwa-Multi-Device-Bluetooth-Touchpad-Wireless-Multi-Touch/dp/B0D5CR6Y47) to control the Pi. I also got a wireless [HDMI transmitter/receiver](https://www.amazon.com/Wireless-Transmitter-Receiver-Portable-Streaming/dp/B0DBPTCQDC/?th=1) and an [HDMI switch](https://www.amazon.com/Anker-Bi-Directional-Switcher-Compatible-Projector/dp/B0CJT6JBM8/?th=1) to alternate my 2nd monitor between the Pi and my desktop in my home office. Those products aren't the only ones out there, and you may be able to find better deals, but those are the ones I ended up getting.

The script is very simple and reusable. It uses [`argparse`](https://docs.python.org/3/library/argparse.html) to accept an argument representing a command to forward to the lights. The argument given to the script is a string (on, off, blue, white, etc.). The script stores a dictionary mapping expected string commands to the hex code values of the different signals. After confirming that the string represents a valid key, the hex value is then transmitted via the `RFDevice` class of the `rpi_rf` module. 

The most up-to-date version of the script can be found [here](https://github.com/sweisss/rpi-smart-hub/blob/main/rf_transmit.py).

With the python script successfully transmitting the RF signals to the lights, and my work station connected to the Pi from the other room, it was now time to return to Node-RED and start putting it all together. 

## Building the Node-RED Flow and Incorporating the RF TX Script
### Confirming MQTT Connections with MQTT Explorer and Android App
The first step I took to incorporate my python script into the Node-RED flow was to call it from a simple `exec` node and activate that node from a simple `inject` node. After confirming that the lights would respond correctly to the signals triggered through Node-RED, I then hooked it up to the `mqtt in` node described earlier. I added some string parsing and error checking to make sure it would only call the `exec` node with valid arguments, and then tested it by sending the commands through the MQTT Explorer app on my desktop. 

From here, I started testing it with various MQTT apps for my Android phone. As mentioned earlier, the end goal of this project is to control the lights from anywhere with an Internet connection (i.e. mobile data connectivity). Also as mentioned earlier, I did not like any of the MQTT apps available for Android devices that I found. I thought about making my own custom Android app, and I still might eventually do that. However, I realized it would be much quicker and easier to set up a private Discord server and make a Discord bot that could relay commands and the status of the lights. This turned out to be a very fun part of the project. 

## Creating the Discord Bot
### Create a Dedicated Server
Creating a personal Discord server is very straightforward. If you haven't done so yet, all you need to do is simply click on the circle icon with the plus (+) in it at the bottom of the list of your server icons.

![Add a Discord Server](images/rpilights/add_discord_server.png)

After you click on it, a menu will pop up with the option to "Create My Own" and after a few more clicks you'll have your own server. Add an icon image for your server so it stands out in the list, and a channel for each device that you want to control to keep things organized (if you want to organize it like I did, it's completely up to you). 

### Create the Bot
With my dedicated Discord server up an running, I then began to set up the bot. This can be done from the developers section of the Discord website. The helpful docs page can be found [here](https://discord.com/developers/docs/intro). The [Applications](https://discord.com/developers/applications) page can be reached at the top of the left-hand navigation menu. You will need to log in with your Discord account credentials to reach this portion of the website. 

Once logged in on the Applications page, click on the blue "New Application" button on the upper-right. You will be prompted to give the app a name. I chose "RPi Messenger" since this app would essentially serve as a messenger between my phone (or computer) and the Raspberry Pi. After agreeing to the Terms of Service and confirming that you're a human, the "General Information" screen for the app will be displayed. Use the navigation menu on the left to move to the "Bot" page under the Settings section.

![Discord Bot Settings Menu](images/rpilights/discord_bot_settings_menu.png)

Give the app an icon image so it stands out, and a banner for a nice touch. I gave it the same one that I gave my server: a fun image of a finger with a happy face drawn on it and wearing a raspberry as a hat. 

![RPi Messenger Icon and Banner](images/rpilights/rpi_messenger_icon_banner.png)

Beneath this area will be an entry box to update the bot's Username. I kept it the same as the Application name (the default setting). The next item on the settings menu is the "Token". If you see the token value and a blue "Copy" button, click copy and immediately save it somewhere safe. It will only be shown once and you will need to reset it if you lose it. If it's already hidden and you only see the blue "Reset Token" button, go ahead and reset it. You will need to enter your password in order to reset it. I had to reset it a couple times. It's a bit of friction, but overall inconsequential if the bot is still new and nothing else depends on the token yet. Below is an example of the token from a throwaway test bot I made.

![Discord Bot Example Token](images/rpilights/discord_bot_example_token.png)

To make things easy, I kept "Public Bot" checked and "Requires OAuth2 Code Grant" unchecked considering I don't intent to invite anyone else to the dedicated server this bot is on and don't intend to invite the bot to any other servers. 

![Discord Bot Public and OAuth Check Boxes](images/rpilights/discord_bot_public_oauth_settings.png)

In the "Privileged Gateway Intents" section, make sure that "Server Members Intent" and "Message Content Intent" are checked. These are necessary for the bot to receive the content of messages sent in the server channels. 

![Discord Bot Intents](images/rpilights/discord_bot_intents.png)

Make sure to save any changes. 

### Attach the Bot to the Server
The final step in setting up the bot is to invite it to the dedicated Discord server. To do this, move from the "Bot" page up to the "OAuth2" page of the left-hand navigation menu and scroll down to the "OAuth2 URL Generator" section. Under "Scopes" check to box for "bot". 

![Discord Bot Scopes Section](images/rpilights/discord_bot_scopes.png)

After selecting the "bot" checkbox, a new section will appear named "Bot Permissions". 
To allow the bot to send and receive messages back and forth between Node-RED and Discord, I selected "View Channels", "Send Messages", and "Read Message History". If you want your bot to have additional abilities, check the appropriate permissions boxes in this section.

![Discord Bot Permissions](images/rpilights/discord_bot_permissions.png)

At the bottom of the page will be a Generated URL. Copy this and enter it into a new tab in your web browser. It will display a few more confirmation screens to authorize the bot. Once authorized, the bot will be added to the server!

## Integrating the Bot to Node-RED
There are several node libraries that integrate Discord with Node-RED. My RPi seemed to be a bit dated, and while several were searchable from the Node-RED "Manage palette", all but one required me to update my underlying NodeJS version for them to work. I decided to go with the one library that did not require fully updating NodeJS and Node-RED: [node-red-contrib-discord 5.0.0](https://flows.nodered.org/node/node-red-contrib-discord). That library hadn't been updated in nearly 5 years. It is definitely a little buggy, but it serves its purpose for my project and hasn't yet given me enough trouble to force me to try a more recently maintained and updated one. 

The Discord library is fairly simple to use. You just need to create a `discord-token` node with the token you saved when creating your bot and give it a helpful name. This `discord-token` node can be created within (while setting up) a `discordMessage` node or a `discordSendMessage` node, which simply listen for and return messages from the bot or send out payload messages to the bot, respectively.

From here I set up two main sections of the flow. The first section starts by using the `discordMessage` node to listen to commands sent via the Discord bot. It then passes the message payload through a series of string parsing and verification nodes. If the command sent is valid, it is forwarded to the MQTT broker. If the command is invalid (for example, I often send it "test" to simply see if the bot is responsive), it returns a message to the bot listing the valid command options. 

The second main section of the Node-RED flow starts by subscribing to the MQTT broker. Any messages published by the broker will be received by this node and go through some string parsing steps before being forwarded to the `exec` node to call the python script that directly sends the RF signals. More string parsing steps follow to combine the return code from the `exec` node with the command from the `mqtt in` node and build a nice, readable message about the status of the action (whether the script completed successfully or not). The `discordSendMessage` node then sends this message out to the Discord bot. 

> NOTE TO AUTHOR: Use detail screenshots of these sections of the flow

### Replacing the MQTT Android App
At this point, the Discord server and bot became my primary means of communication with the RPi, essentially replacing the user-unfriendly MQTT Android apps. Rather than communicating with the broker directly, the Discord nodes communicate with the Discord server via websockets and HTTPS (**NOTE TO AUTHOR: Look into this deeper**). It's best practice to only expose ports that are regularly being used or are essential for the services to run correctly. Therefore it's a good idea to go back to the router interface and close the port exposing the broker to the public-facing Internet (1883). 

Because I am now utilizing the Discord bot as the main communication with the RPi, the MQTT broker could probably be bypassed altogether. This would certainly simplify the Node-RED flow and the overall project. However, I decided to keep the MQTT broker for a couple reasons. First, I still have the idea of possibly making a custom Android app that would utilize MQTT to communicate with the RPi. Second, the bugginess of the Discord node library has given me the attitude that it is nice to be ready with backup methods of communication. If you're using this writeup to influence your own project, weigh these options and make the decision that makes the most sense for your situation. 

## Polishing and Final Touches
### Incorporating the Discord Bot
- Utilizing environment variables
### Building the Schedule
- Turning the lights on a sunset
- Set a backup fixed time in case of loss of Internet
### Setting a static IP in the router interface
### Adding the Nightlight
  - Attempt at IR transmission
      - Issues (failure to transmit, line of sight)
  - RF USB switch and USB LED workaround
### Fine tuning string parsing and error messages

## Future Expansion
### Add Self-Hosted DNS
So the MQTT broker doesn't get reasigned a new IP address
late at night when you are trying to go to sleep...
### Strengthen the RF Signal
So the RPi isn't as limited on possible locations 
### Add Ceiling Fan Control
### Add Front Porch Light Control
Use Wireshark to analyze and decode the signals that the smart switch uses to control
the front porch light and try to reproduce those signals in the Node-RED flow
### Build an Android App

-----

[Home](https://sweisss.github.io/) &emsp; &emsp;
[Personal Projects](https://sweisss.github.io/#personal-projects) &emsp; &emsp;
[OSU Projects](https://sweisss.github.io/#oregon-state-university-projects) &emsp; &emsp;
[Element 1 Projects](https://sweisss.github.io/#element-1-projects) &emsp; &emsp;
[Videos](https://sweisss.github.io/#videos)

(C) Seth Weiss, 2025
