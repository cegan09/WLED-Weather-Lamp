# WLED-Weather-Lamp
This project allows you to build a WLED controlled lamp that provides weather visualization for your area. 

Up Front warnings. 
This project relies on data from your closest observation station as reported to weather.gov. This means some areas will recieve more frequent and more accurate updates to your specific location. If you live further away from the observation station, the data reported has the chance of being less accurate. In addition some stations do not update with the same frequency as others. As a result you may not see updates more frequently than once an hour. The point of this project was not to create the most accurate current weather display possible, more to create something fun that conveyed some useful information. 

Required parts:
1. Physical Lamp - thingiverse link
2. NodeRed server (can be installed on a computer or raspberry Pi)
3. MQTT Server (can be installed on a computer or Raspberry Pi)

NodeRed and MQTT Server
There are many ways to implement these portions. The easiest is to simply install both on a raspberry pi. 
I have used Mosquitto Broker for MQTT, the installation documentation can be found here - https://mosquitto.org/ . Be sure to set a static IP for your MQTT server. 
Nodered installation instructions for the raspberry pi are here - https://nodered.org/docs/getting-started/raspberrypi . Be sure to set a static IP for your NodeRed Server (this will be the same as the MQTT server if both are on the same device, but you do not have to have them on the same device). 

Copy the NodeRed Flow code from the flow.txt file and paste it into the nodered import window to setup the flows. 

Building and configuring the lamp. 
1. Follow the instructions in the thingiverse page to construct the actual lamp. 
2. Flash the ESP controller (D1 Mini) with WLED and connect it to your network following their instructions (https://github.com/Aircoookie/WLED/wiki/Install-WLED-binary)
3. Setup a static IP for the WLED Controller (varies based on your home networking hardware)
4. Make the following changes in the WLED settings:
  A) LED Settings
     a) LED Count - 80
     b) Max Current - set to your own power supply rating
     c) Apply Preset 1 at boot
   B) Time Setup
    a) Set to your timezone
    b) Short press macro - 1
    c) Long Press macro - 2
    d) Double Press macro - 3
    e) macro 1 - &PL=2
    f) macro 2 - &PL=3
    g) macro 3 - &T=2
  C) Sync Interfaces
    a) enable MQTT and set the IP address of your server
    b) Set the device ID to "WLED-Weather"
    c) Set the device topic to "wled/weather"

Finding the location data from weather.gov. 
You will need to know your local reporting station and the local area for forecast data in order to enter them in the nodered flow. 
1. Go to weather.gov and enter your zip code to get the current conditions and forecast. One of the first sections will be current conditions at your local airport or other reporting station. You will need to write down the identifier code for that station. In this example it is KBOS for the Boston Airport. 

![image](https://github.com/cegan09/WLED-Weather-Lamp/blob/master/pictures/local%20station.PNG)

2. Now you need the grid points for your local forecast from weather.gov. This takes two steps. First find the latitude and longitude for the area you want the forecast of. Go to maps.google.com and navigate to your location. Click once on the location you're interested in and a small popup at the bottom of the window will show the latitude and longitude of that point. 
**picture**
Next you need to use those in an API call to weather.gov to find your grid points. In a new web browser tab, enter the URL https://api.weather.gov/points/{latitude},{longitude} and enter the latitude and longitude values you located on google maps. Down the page, in the properties section you will see the grid X and Y values that you need to record.
**picture**


Initial Node Red Setup. 
Once the flows are imported, there are a couple variables that need to be set before anything will run. 
1. Configure the MQTT in node
  A) Set the address to your MQTT broker ip and port (default is 1883). If your IP is 192.168.0.100 the entry should be 192.168.0.100:1883
  B) Set the topic to "wled/weather/c"
2. Configure the Weather API node
  A) edit the node, in the URL field replace the KBOS station with your local station found above. 
3. Configure the Forecast API
  A) Edit the node, in the URL field replace grip point values with the once you located in the above steps. 
4. Configure the WLED JSON Send node
  A) Edit the node, in the URL field change the IP to be the IP of your WLED controller. Be sure not to change the /json/state at the end. 
5. Deploy the current flow


Setting the WLED Save states
1. In the Setup Functions Flow there are three input triggers, one for each WLED save state. You want to trigger each one and then save the state in WLED. 
  A) Trigger the first line for save state 1. You should see an MQTT message come through the debug window with #010000 as the message for color set. 
  B) Go to the WLED control page (at the IP of your WLED controller) and in favorites, make sure "saving mode" is checked and then select slot 1. 
  C) Back in nodered, trigger the line for Save state 2. You should see the MQTT message come through with #000100 as the color
  D) In WLED save this as state 2. 
  E) Repeate with state 3, which should show #000001 as the MQTT message. Save this as state 3 in WLED. 
2. This will setup the WLED controller with the specific colors needed to trigger different functions in nodered. 

Enabling the rest of the nodered flow. 
1. In the main WLED Control Flow, scroll down to the "Current Conditions" section, edit the 15minute trigger node, and enable it. Here you can also change the frequency that nodered polls for updates, but it's not worth polling any more frequently than your local station pushes updates. 
2. Next in the "MQTT Handling" section, edit and enable the link in node. 
3. Deploy the Flows again to enable the full functionality. These items were dissabled to stop the flow trying to run during the save state setup steps. 
4. Enjoy the colorful distraction!

 
