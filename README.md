# WLED-Weather-Lamp

**Warning, project not yet complete. Please see the list of items not yet finished**

Items that still need completion:
1. Code files are not yet uploaded. They will be soon. 
2. Not all weather animations are created yet. The common ones are, but there are still enough incomplete that you will see the unknown animation occasionall
3. Forecast handling is not yet finished. Anything that comes back as "some weather THEN some other weather" throws the unknown animation
4. There is still a lot of optimization to be done with the nodered flow, though most funstionality works for now


Fuctionality Overview
======================
- The main behavior of the weather lamp is to poll your local observation station, and display the latest reported conditions and temperature. The bottom row of LEDs will show a color mapped representation of the current temperature. The rest of the lamp displays the set animation for that weather condition. 
![image](https://github.com/cegan09/WLED-Weather-Lamp/blob/master/pictures/conver1.jpg?raw=true)

- A short press of the button on top will poll weather.gov for the next day's forecast and then display the corrisponding animation and high temperature. Currently there is a limitation where any forecast that lists some condition "then" some other condition will only show the first listed condition. In the future this may change. 

- Long pressing the button will prompt a temperature visulization for the next 24(ish) hours. The lamp will poll weather.gov for a forecast and then display the received high or low temperatures for the next 3 time segments as three bands of color. Normally these bands will be the current day's high, that night's low, and the following day's hight. If pressed late enough at night it should display that night's low, the next day's hight, and the following night's low. The bottom band is the current with the top band being the furthest forecast segment. 
![image](https://github.com/cegan09/WLED-Weather-Lamp/blob/master/pictures/cover2.jpg?raw=true)

- A double press of the button should turn the lamp off/on. The controller will remain on and communicating with the node red flow, but the LEDs will be powered off or on. 

Future To-do List
1. Write the how to on how to add animations of your own
2. Write the advanced guide to tweaking behavior for forecast modes


This project allows you to build a WLED controlled lamp that provides weather visualization for your area. 
----------------------------------------------------------------------------------------------------------

Up Front warnings. 
This project relies on data from your closest observation station as reported to weather.gov. This means some areas will recieve more frequent and more accurate updates to your specific location. If you live further away from the observation station, the data reported has the chance of being less accurate. In addition some stations do not update with the same frequency as others. As a result you may not see updates more frequently than once an hour. The point of this project was not to create the most accurate current weather display possible, more to create something fun that conveyed some useful information. 

Required parts:
1. Physical Lamp - (Thingiverse)[https://www.thingiverse.com/thing:4566731]
2. NodeRed server (can be installed on a computer or raspberry Pi)
3. MQTT Server (can be installed on a computer or Raspberry Pi)

NodeRed and MQTT Server
There are many ways to implement these portions. The easiest is to simply install both on a raspberry pi. 
I have used Mosquitto Broker for MQTT, the installation documentation can be found here - https://mosquitto.org/ or here for a raspberry Pi specific tutorial https://randomnerdtutorials.com/how-to-install-mosquitto-broker-on-raspberry-pi/. Be sure to set a static IP for your MQTT server. 
Nodered installation instructions for the raspberry pi are here - https://nodered.org/docs/getting-started/raspberrypi . Be sure to set a static IP for your NodeRed Server (this will be the same as the MQTT server if both are on the same device, but you do not have to have them on the same device). 

Once Node Red is installed you will need to install some initial nodes. From the upper right menu, select "manage palette" and then go to the install tab. Search for "node-red-contrib-color-convert" and install. Additionally you may install the "node-red-dashboard" nodes which will allow you to build a web dashboard to send the weather data to (not covered in this project). 

Copy the NodeRed Flow code from the flow.txt file and paste it into the nodered import window to setup the flows. 

Building and configuring the lamp. 
1. Follow the instructions in the thingiverse page to construct the actual lamp. 
2. Flash the ESP controller (D1 Mini) with WLED and connect it to your network following their instructions (https://github.com/Aircoookie/WLED/wiki/Install-WLED-binary)
3. Setup a static IP for the WLED Controller (varies based on your home networking hardware)
4. Make the following changes in the WLED settings:

  A) LED Settings
     
     LED Count - 80
     
     Max Current - set to your own power supply rating
     
     Apply Preset 1 at boot
     
   B) Time Setup
   
    Set to your timezone
    
    Short press macro - 1
    
    Long Press macro - 2
    
    Double Press macro - 3
    
    macro 1 - &PL=2
    
    macro 2 - &PL=3
    
    macro 3 - &T=2
    
  C) Sync Interfaces
    
    enable MQTT and set the IP address of your server
    
    Set the device ID to "WLED-Weather"
    
    Set the device topic to "wled/weather"

Finding the location data from weather.gov. 
You will need to know your local reporting station and the local area for forecast data in order to enter them in the nodered flow. 
1. Go to weather.gov and enter your zip code to get the current conditions and forecast. One of the first sections will be current conditions at your local airport or other reporting station. You will need to write down the identifier code for that station. In this example it is KBOS for the Boston Airport. 

![image](https://github.com/cegan09/WLED-Weather-Lamp/blob/master/pictures/local%20station.PNG)

2. Now you need the grid points for your local forecast from weather.gov. This takes two steps. First find the latitude and longitude for the area you want the forecast of. Go to maps.google.com and navigate to your location. Click once on the location you're interested in and a small popup at the bottom of the window will show the latitude and longitude of that point. 

![image](https://github.com/cegan09/WLED-Weather-Lamp/blob/master/pictures/lat%20long.PNG)

Next you need to use those in an API call to weather.gov to find your grid points. In a new web browser tab, enter the URL https://api.weather.gov/points/{latitude},{longitude} and enter the latitude and longitude values you located on google maps. Down the page, in the properties section you will see the grid X and Y values that you need to record.

![image](https://github.com/cegan09/WLED-Weather-Lamp/blob/master/pictures/grid%20points.PNG)


Initial Node Red Setup. 
Once the flows are imported, there are a couple variables that need to be set before anything will run. In the Weather Lamp Setup Flow set the following:
1. MQTT config Node
  
  Set the address to your MQTT broker ip and port (default is 1883). If your IP is 192.168.0.100 the entry should be 192.168.0.100:1883
  
  Set the topic to "wled/weather/c"
  
2. Configure the Weather API node
  
  edit the node, in the URL field replace the KBOS station with your local station found above. 
  
3. Configure the Forecast API

  Edit the node, in the URL field replace grip point values with the once you located in the above steps. 
  
4. Configure the WLED JSON Send node

  Edit the node, in the URL field change the IP to be the IP of your WLED controller. Be sure not to change the /json/state at the end. 
  
5. Deploy the current flow


Setting the WLED Save states

In the Weather Lamp Setup Flow, do the following:
1. At the bottom of the flow there are three input triggers, one for each WLED save state. You want to trigger each one and then save the state in WLED. 

    A) Trigger the first line for save state 1. You should see an MQTT message come through the debug window with #010000 as the message for color set. 
  
    B) Go to the WLED control page (at the IP of your WLED controller) and in favorites, make sure "saving mode" is checked and then select slot 1. You should see a confirmation message that save state 1 is saved
  
    C) Back in nodered, trigger the line for Save state 2. You should see the MQTT message come through the debug window with #000100 as the color
  
    D) In WLED save this as state 2. 
  
    E) Repeate with state 3, which should show #000001 as the MQTT message. Save this as state 3 in WLED. 
  
2. To test that this was done correctly, first unplug the lamp, then plug it in again. In the debug window of node red you should see an MQTT message of "#010000" come through. Next, short press the button on the lamp, which should have a message of "#000100" come through the debug window. Lastly long press the button, which should show "#000001" in the debug window. If all of these work, you're ready to continue

Enabling and configuring the rest of the nodered flow. 

1. In the Weather Lamp Setup Flow, scroll down to the "Current Condition polling interfal" section, edit the Update Trigger node, and enable it (bottom left of the window). Here you can also change the frequency that nodered polls for updates. Note that it's not worth polling much faster than your local station actually updates. For some locations this may be often as as the weather changes. Other stations may be as slow as once per hour. The default is set to every 5 minutes. 

2. In the Weather Lamp Setup Flow, scroll down to the "Current Conditions Reactivation Delay" section, edit the delay node to your prefereance. This node determines how long the lamp will show the forecast animations before returning to show the current conditions. The default here is 1 minute, feel free to change this to as long as you wish. 

3. Next, at the top of the Weather Lamp Setup Flow, edit the link out node attached to the MQTT Config node, at the bottom left of the window enable the node. 

3. Deploy the Flows again to enable the full functionality. These items were dissabled to stop the flow trying to run during the save state setup steps. 

4. Enjoy the colorful distraction!

 
