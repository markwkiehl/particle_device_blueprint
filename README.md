# particle_device_blueprint
Connect any Particle IoT device to Blynk

# Introduction
This blueprint will configure bi-directional communication between any [Particle](https://www.particle.io/) device and [Blynk IoT](https://blynk.io/). &nbsp; Both a Blynk web dashboard and mobile app will be generated that will display data from the Particle device, and they will be used to control the Particle device remotely.  

# How It Works
Data is pushed from the Particle device to a Particle integration webhook. &nbsp; The webhook makes a call to the Blynk HTTP API and updates the Blynk datastreams on the Blynk cloud. &nbsp; The updated datastreams cause the Blynk web dashboard and mobile app widgets to be updated.  

Widgets on the Blynk web dashboard and mobile app trigger a Blynk webhook.  The webhook makes a Particle HTTP API call that sends data to the Particle device. &nbsp; A firmware function on the Particle device reacts to the data received and causes the hardware perform a custom action. 

All communication between the Particle device and Blynk will utilize the Blynk HTTP API rather than the [Blynk library](http://library.to/). &nbsp; This minimizes data usage when a Particle cellular device is employed.  

The BLYNK AUTH TOKEN will be sent by the device with each data packet and the Particle webhook will pass this on when it communicates with Blynk via the Blynk HTTP API.  This allows the same Particle integration webhooks to be used with multiple Particle devices.  

The Blynk webhook called by the Blynk web or mobile app widget makes a call to the Particle API for a Particle cloud function with a device unique token.  The Particle cloud function running on the Particle hardware performs and action in response to the call, and responds the the API call with a result code.   

## Functional Requirements
- The firmware on the Particle device will push data consisting of an integer value, floating point number (sensor value), and the BLYNK AUTH TOKEN. 
- A date/time stamp in UTC will be included with data sent to Blynk so the last time data was sent will be visible.
- A button widget on the Blynk web dashboard and mobile app will be configured to turn a built-in LED on the Particle device on or off.  

# Components Used in This Project
- Particle Console to set up the Webhook and activate the Hardware
- Blynk Console and Blynk App for web and mobile dashboards
- Particle Web IDE, Particle Workbench or Particle Comand Line to upload firmware
- Particle IoT device:
  - **Tracker One** is complete system including an enclosure. It also comes as a SoM (system on module) known as **Tracker SoM**.  See [Tracker button and LEDs](https://docs.particle.io/hardware/tracker/projects/tracker-buttons-leds/) for details on how to add an LED to the Tracker One. 
  - All **Boron** devices in the Feather hardware format, including those that are depreciated. &nbap; All are cellular devices. &nbsp; A built in LED is on D7.
  - **B Series SoM** system-on-a-module. &nbsp; No built-in LED, must custom install.  
  - **Photon 2** is a WiFi device with built in LED is on D7.
  - **Argon** is a WiFi device with built in LED is on D7.
  - **Photon** is a WiFi device with built-in LED on D7.
  - **P2** is an SMD module with WiFi and a microcontroller.  No built-in LED.  

The firmware sketch sends an integer value from the millis() function (number of milliseconds elapsed since the device started) and a constant floating point value. &nbsp; It is up to the developer to adapt these variables to the particular hardware and application that will be used. &nbsp;  It will be apparent from the content of the blueprint how to add additional data items to be pushed to Blynk.  &nbsp;

The web and mobile app button widget toggles the state of a datastream of data type integer with the values of 1 and 0. &nbsp;   For a Boron in Feather format, GPIO D7 is utilized since it is connected to the built-in LED.  For other Particle hardware (without the built-in LED), the code will need to be adapted to specify a different GPIO, and the hardware connected to an LED.  

# 1. Prepare Your Hardware

# 2. Create Webhooks on the Particle Cloud

# 3. Prepare Required Software

# 4. Prepare the Firmware and Upload It to Your Device

# 5. Dashboard Breakdown

# 6. Set up Automations 
[include this?]

# 7. Testing

# Conclusion

# Next Steps

# Troubleshooting

# Related Links

