# Node Red with the OmniPreSense RADAR sensor 
OmniPreSense has crafted some demonstrations of the OmniPreSense OPS 243A radar traffic sensor using NodeRed to present a live-traffic speed dashboard.  Node Red is a low-code rapid prototype web application which runs on the node.js platform.  More information on Node Red is available at [nodered.org](nodered.org)


## Table of Contents
Flows available in this repository are:

serial_flow.json - This flow is designed to work with the sensor attached to the serial port.  By default, COM3 is used.  If deployed on a linux system, /dev/ttyACM0 would be the proper value.

ipcamera_flow.json - This flow extends the serial_flow.json flow by adding some web service calls to an IP camera.  (In particular, this has worked with an Axis camera.)  Speeds are sent as video Overlays.  It also has some tricks to bring up a completely blank overlay when there is no traffic, which can ease the burden of license plate OCR when no traffic is present.


## Node Red download
On Raspberry Pi's, a copy of Node Red comes with the Linux distribution.  However,
[Node Red's website strongly discourages using the preinstalled version and installing fresh.](https://nodered.org/docs/getting-started/raspberrypi)   (They say that the preinstalled version does not have "npm".  Without npm it is very difficult to install node packages used by node red.)
Detailed installation information for installing Node Red on the Pi are found in the link above, however, here are two critical commands from that page:
```
sudo apt install build-essential git curl
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```

If installing on another linux/unix platform, the above two command lines should work well.  Microsoft Windows users should search for Node Red's installation documentation for Windows.    

## Node Red Installation and Configuration

Recent versions of Node Red ask a series of questions that are helpful and avoid some of the more unpleasant manual configuration tasks documented below.

For a convenient experience, Node Red can be configured to serve traffic on port 80 (in ```~/.node-red/settings.js```) and to have a password for admin pages.     

### General settings
Further refinements are listed below and should be an indicator of settings you might want to change:  
```
uiPort: process.env.PORT || 80,
credentialSecret: "**your secret**",  
httpAdminRoot: '/admin',  
ui: { path: "ui" },
```
### Securing the admin view
The modern node red installer asks you to create an admin account and a password.  You should follow these installer intructions.
This was formerly a chore that required editing the settings.json file.  If you still need to do it manually, you can by following instructions in the settings.json file.  Look for keywords such as 
```credentialSecret``` and the commented out adminAuth section.  
Read more about [securing Node Red](https://nodered.org/docs/user-guide/runtime/securing-node-red) at nodeRed.org

### Further OmniPreSense customization

The flows makes use of a few operating-system files for persistent storage. You'll need to create these files because Node Red does not.  (By default, node red will use your home directory for files, but if you change the home directory, cd to that location in the first step below.)
```
cd ~
echo {} > init.json
echo {} > inbound-store.json
echo {} > outbound-store.json
echo {} > all_received.json 
```

### Title and Location/map settings
These are now set in the admin user's initialization (first) tab, in the init node.

## Palette additions
Before importing the flow the first time to a just-installed node red, many modules will be unavailable and need to be installed.  

### To use Node Red's Palette Manager to install these dependencies
* node-red-node-serialport
* node-red-dashboard
* node-red-contrib-dashboard-bar-chart-data
* node-red-contrib-advance-logger
* node-red-contrib-web-worldmap
* node-red-contrib-persist

#### If you have shell access, you can use use npm directly to install node red libraries
As you run the following lines, fix any errors you see and use your judgement when ignoring warnings.  Note that serialport historically complains a lot during compilation:

sudo npm install node-red-node-serialport
sudo npm install node-red-contrib-web-worldmap
sudo npm install node-red-dashboard
sudo npm install node-red-contrib-dashboard-bar-chart-data
sudo npm install node-red-contrib-advance-logger
sudo npm install node-red-contrib-persist

Feeling lucky? Here's everything in one line
```
sudo npm install node-red-contrib-web-worldmap node-red-node-serialport node-red-dashboard node-red-contrib-dashboard-bar-chart-data node-red-contrib-advance-logger node-red-contrib-persist
```
Special note: ui_worldmap has, at times, been unreliable.
As of this writing, it's been working fine.  If troubles happen again, the following command can install a specific version, and in this example from years ago, we had good luck with version 2.2.1:
```sudo npm install node-red-contrib-web-worldmap@2.2.1```

## Linux considerations
A commmon use case is to deploy to a Raspberry Pi running raspbian or ubuntu and Node Red.  The Node Red executable should be granted the Capability to access port 80, specifically via
```setcap cap_net_bind_service=+ep ***your-node-red-binary***``` and to use the USB port (/dev/ttyACM0)  
To find the node binary (used by node-red), from a shell prompt, issue ``which node``  (verify that the value is not a symbolic link then..) run the command with sudo privilege. e.g.   (assuming that the node binary was found at /usr/bin/node)

```sudo setcap cap_net_bind_service=+ep $(eval readlink -f `which node`)```

## Node Red can be run securely.  
The installation script does not help with makin your installation secure.  
By installing Nginx and securing it with a Lets Encrypt certificate, and having nginx decrypt and forward traffic to the node red instance, you can have the most flexible, the most secure, and easiest path to installation, considering the hassles of editing ```settings.json```.   Furthermore, if you use nginx, you can leave Node Red's default port of 1880 and simply have nginx forward there.  
Alternatively, Node Red can be run with SSL on port 443.  If you are willing to get an SSL certificate (for free from Lets Encrypt) then you can run node red securely.  
Read more about [securing Node Red, at NodeRed.org.](https://nodered.org/docs/user-guide/runtime/securing-node-red)



## Flow import
Flows can be imported into Node Red running on a system with an OPS9243-A attached.

## MQTT considerations
If you wish to render data from a MQTT broker, disable/remove the nodes that refer to serial ports and install nodes to support MQTT subscription events from your MQTT broker.  ([Mosquitto](https://mosquitto.org/) is a great reference implementation of an MQTT broker, and is freely available, with installation support for many platforms.  Node Red is capable of also being the mqtt broker if desired.)  
The OmniPreSense Wifi module publishes events to an MQTT broker.  The OmniPreSense android app can be used to display these events.  Please contact our customer support if you wish to use MQTT outside of this narrow contraint.  We're happy to help advise other potential use cases.

## IP Camera
We have had success passing along data to various cameras to display in overlays

## Integration with email and SMS
We have had success integrating with email systems (gmail and MS Exchange) and with SMS vendors (like RingCentral) with these flows.  Please contact our customer support if you require assistance.
