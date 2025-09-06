# Welcome to Seth's Landing Page

This page was quickly cobbled together to showcase the 
[Integrated Test Stand](https://sweisss.github.io/osu-cascades/capstone-24/IntegratedTestStand.html)
Capstone Project for Oregon State University -- Cascades, Spring 2024.

This page is currently under construction and will eventually showcase other aspects of Seth's portfolio...

### Links
🐙 - [GitHub](https://github.com/sweisss) 
<br>
🔗 - [LinkedIn](https://www.linkedin.com/in/seth-weiss-62793858/)
<br>
▶️ - [YouTube](https://www.youtube.com/@sethweiss) 

----

### Contents
- [OSU Projects](#oregon-state-university-projects)
- [Personal Projects](#personal-projects)
- [Element 1 Projects](#element-1-projects)
- [Videos](#videos)


## Oregon State University Projects
Projects created for specific classes at various campuses of Oregon State University from Fall 2020 to Spring 2024.

### Capstone 2024 - Integrated Test Stand
An academic year-long project involving a team of three Computer Science students.
The team partnered with a local business to develop on consolidated data recording program that incorporates
various devices on a hydrogen generator.

[Read more](https://sweisss.github.io/osu-cascades/capstone-24/IntegratedTestStand.html)

### RxWatch
An Android app created as a final project by a team of four Computer Science students for CS 492 - Mobile Development at Oregon State University, Winter 2024.
The app allows users to enter a drug name and search for other drugs that may have adverse interactions with that drug.
It uses two different endpoints of the [openFDA](https://open.fda.gov/) API to search drug labels and return statics of reported adverse events. 

View the project on [GitHub](https://github.com/sweisss/RxWatch)

### Intro to Security
A GitHub repo containing a colleciton of projects for CS370 - Intro to Security at Oregon State University - Ecampus, Fall 2023.

View the project on [GitHub](https://github.com/sweisss/CS370-Intro_to_Security)

### Zillite
A web application created as a final project by a team of four Computer Science students for CS 494 at Oregon State University, Winter 2023.
The uses Zillow as an inspiration to practice and perfect proper web development techniques using the React library.  

View the project on [GitHub](https://github.com/sweisss/Zillite)

### Chemistry Calculations
An Excel workbook created during CH202 - Chemistry for Engineering Majors at Oregon State University - Cascades, Fall 2020.
Seth used the massive amount of chemistry homework as an opportunity to teach himself the wonders of Excel. Each sheet corresponds with a
particular week of the course. The workbook utilizes the name manager to reference constants and values throughout the workbook from the tables
located on the first sheet. 

[Direct Download](./projects/ChemistryCalculations.xlsx)


## Personal Projects
Projects that are unrelated to but utilize the knowledge and experience gained from a CS degreen at OSU-Cascades.

### Raspberry Pi Patio Lights
Turning dumb devices into smart devices using a Raspberry Pi, Node-RED, MQTT protocol, RF transmissions, and a Discord bot.
> **NOTE:** Make this description better!

Read more about the project [here](./projects/smarthubfordumblights.md).

### Percentage Clock
A simple clock that tells the current time as a percantage of two different time ranges. The first time range is midnight - midnight.
The second time range is a "Work Day" and can be selected by the user. 
This is a desktop GUI application written in python using tkinter.
> **Note:** This application currently only works on Windows.

View the project on [GitHub](https://github.com/sweisss/percent-clock-gui)

### Volcano Season 3
The third iteration of an Android app written in Kotlin that holds a list of quick-links to mountain forecasts and an equipment checklist.
This third iteration builds on the primitive first and second versions by applying the knowledge and experience gained from the RxWatch project above. 

Read more about the project [here](./projects/volcanoseason3.md).
<br>
View the project on [GitHub](https://github.com/sweisss/VolcanoSeason3)

### QR Coder
A simple command-line tool that takes text input and generates a QR code in any desired combination of:
- Printed to the console
- Saved as an SVG
- Saved as a PNG

The tool is based off of a portion of the OTP project in the Intro to Security project above.
It alows for quick and easy generation of QR codes without the need to sign up for online services.

View the project on [GitHub](https://github.com/sweisss/QR_Coder)

### Bowling Scoreboard
A bowling scoreboard built in Google Sheets to help keep track of the winners and losers during competitive events on family vacations involving
a [table bowling](https://us.amazon.com/bowling-table/s?k=bowling+table) set. The sheet includes a macro to clear out the scoreboard to be ready
for the next round as quickly as possible!

Access the Google Sheets file [here](https://docs.google.com/spreadsheets/d/1T2dHcvSfGK9w3xW4Mf51phTdxzOd0TZNsUDxLNJYssI/edit?usp=sharing).
It is set as "View Only", but anyone can make a copy of it and make it their own to start editing or simply entering scores. 

### More to come...


## Element 1 Projects
Thesse projects were all made for Element 1 (e1), a company in Bend, OR that makes hydrogen generators.
Because e1 has not yet given permission to release the source code of these projects, they are only described here. 

### Remote Access Alarm Server
Worked on as an intern during Summer 2022 and 2023.
The server is written in python and uses MQTT protocol to accept incoming messages
from registered hydrogen generators. The messages can either be CAN based or in JSON from a Siemens IoT device.
Once the server receives a message, it decodes it and logs it in a csv. It also searches the message for fault codes.
If a fault code is found, it utilizes the system ID (also in the message) to forward the fault via SMTP to the engineers
associated with that system in an SQLite database. 

### TigerTamer
A desktop GUI data processing program for the [TigerOptics Prismatic 3](https://www.process-insights.com/crds-gas-analyzers-2/prismatic-3/).
TigerTamer combines the csv data files from all four of the gas channels from the "Tiger" and more importantly,
it cuts out all data outside a set date/time range selected by the user. It then graphs these data in an Excel workbook to form a report. 

As the TigerTamer project evolved, it incorporated a server and an SQLite database that is connected to the Tiger which queries the tiger every night
to update the database. The server uses a simple API that the TigerTamer client accesses via the GUI application so that the user can
retrieve the Tiger data from accross the building while connected to the local Wi-Fi network. 

### Render FAT
A Desktop GUI program to process Factory Acceptance Test data for hydrogen generators. 
The program takes csv data from the internal contorl device (e.g. PLC) and optionally from an external measuring device
(e.g. PLC recording data from a [Coriolis flow meter](https://www.emerson.com/en-us/automation/measurement-instrumentation/flow-measurement/coriolis-flow-meters)).
The program lines up the data files based on flow movement and control states and then calculates efficiency data. 
After all data has been processed, it then writes and graphs the data in an Excel workbook (multiple Excel workbooks if it
detects multiple runs). 

### MM Calculator
A program to calculate the ideal metal mebrane area in the hydrogen generators.

### Mole Mapper
A program to find equilibrium composition by direct minimization of Gibbs free energy.

### Mini Reactor Reporter
A program to process and graph data from the mini reactor.

### S-Series Dockerized HMI
A new HMI for the S-Series hydrogen generators utilizing Docker containerization and PowerShell scripts for installation and startup.  


## Videos
### Python Dictionaries
A video explaining some cool and interesting things that you can do with a python dictionary.
The video was originally made for a leadership project at OSU-Cascades in Spring 2024, but was really intended for a greater audience.
It coveres a brief introduction to dictionaries, how to use a dictionary as a priority queue, and how to store and call functions from within a dictionary. 

Watch it on [YouTube](https://youtu.be/-SqWVBQvj-w)

### Packets
A short video for CS372 - Intro to Networking at Oregon State University - Ecampus, Summer 2023.
The vido coveres how to calculate transmission time of packets.
> **NOTE**: Watch the video and confirm that this is actually talks about!

Watch it on [YouTube](https://youtu.be/ZB5TZAU-Dq4)

### Informational Speach
A short video made for COM111 - Public Speaking at Central Oregon Community College, Fall 2020.
It coveres some basic safety meaures to take when backcountry skiing. However, the point of the video was to learn and practice
public speaking and communication skills, not to act as an authority on safe backcountry travel or recreation. 

Watch it on [YouTube](https://youtu.be/dyawWzhf51A)

-----

This is a pseudo-footer

(C) Seth Weiss, 2025
