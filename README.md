# particle_device_blueprint
Connect a Particle IoT device to Blynk

# Introduction
This blueprint will configure bi-directional communication between any [Particle](https://www.particle.io/) device and [Blynk IoT](https://blynk.io/). &nbsp; Both a Blynk web dashboard and mobile app will be generated that will display data from the Particle device, and they will be used to control the Particle device remotely.  

## Functional Requirements
- The firmware on the Particle hardware will push data that includes an integer value and a floating point number (simulated sensor value or other source) at a regular interval of every 5 minutes (adjustable in the firmware).
- A date/time stamp in UTC must be included with data sent to Blynk so the last time data was sent will be visible.
- The last integer value, floating point value, and UTC datetime stamp must be displayed on the web dashboard and mobile app.
- The historical values for the integer and floating point value must be displayed in a line chart.  
- Multiple Particle devices must be able to use the same Particle webhook. 
- A button widget on the Blynk web dashboard and mobile app will be configured to send a state change command (1/0 value) back to the Particle hardware that will turn ON/OFF the built-in LED (if no LED on GPIO D7 then the user must configure custom GPIO and connect LED).
- Communication frequency must be configured to minimize data usage for the case when a cellular device is employed.  

# How It Works
All communication between the Particle hardware and Blynk will utilize the Blynk HTTP API rather than the [Blynk library](http://library.to/). &nbsp; This minimizes data usage when a Particle cellular device is employed.  

The BLYNK AUTH TOKEN will be sent by the device with each data packet and the Particle webhook will pass this on when it communicates with Blynk via the Blynk HTTP API.  This allows the same Particle integration webhook to be used with multiple Particle devices.  

The firmware sketch sends an integer value from the millis() function (number of milliseconds elapsed since the device started) and a constant floating point value every 5 minutes. &nbsp; The data is pushed from the Particle hardware to a Particle integration webhook. &nbsp; The webhook makes a call to the Blynk HTTP API and updates the Blynk datastreams on the Blynk cloud. &nbsp; The updated datastreams cause the Blynk web dashboard and mobile app widgets configured for those datastreams to be updated.  

Widgets on the Blynk web dashboard and mobile app trigger a Blynk webhook. &nbsp; The webhook makes a Particle HTTP API call that sends data to the Particle hardware. &nbsp; A firmware function on the Particle hardware reacts to the data received, and causes the hardware perform a custom action. 

The Blynk webhook called by the Blynk web or mobile app widget makes a call to the Particle API for a Particle cloud function with a device unique token.  The Particle cloud function running on the Particle hardware performs and action in response to the call, and responds the the API call with a result code.   

# Components Used in This Project
- [Particle Console](https://console.particle.io/) to activate the hardware and set up the integration/webhook. 
- Blynk Console and [Blynk App](https://docs.blynk.io/en/downloads/blynk-apps-for-ios-and-android?_gl=1*hxem43*_ga*NTQ1NjUzMTkwLjE2NjY1NTA3MTk.*_ga_E376ZQ635Y*MTY4NzE3MTI1Mi44OC4xLjE2ODcxNzEyNTQuMC4wLjA.) for web and mobile dashboards
- Particle Web IDE, Particle Workbench or Particle Comand Line to upload firmware.
- One Particle IoT device:
  - **Tracker One** is complete system including an enclosure. It also comes as a SoM (system on module) known as **Tracker SoM**.  See [Tracker button and LEDs](https://docs.particle.io/hardware/tracker/projects/tracker-buttons-leds/) for details on how to add an LED to the Tracker One. 
  - All **Boron** cellular devices, including those that are depreciated. &nbsp; All have a built-in LED is on D7.
  - **B Series SoM** system-on-a-module. &nbsp; No built-in LED, must custom install.  
  - **Photon 2** is a WiFi device with built in LED is on D7.
  - **Argon** is a WiFi device with built in LED is on D7.
  - **Photon** is a WiFi device with built-in LED on D7.
  - **P2** is an SMD module with WiFi and a microcontroller.  No built-in LED.  


The web and mobile app button widget toggles the state of a datastream of data type integer with the values of 1 and 0. &nbsp;   For a Boron in Feather format, GPIO D7 is utilized since it is connected to the built-in LED.  For other Particle hardware (without the built-in LED), the code will need to be adapted to specify a different GPIO, and the hardware connected to an LED.  

# 1. Prepare Your Hardware
The blueprint includes a sketch (.ino file) that is uploaded to the Particle hardware.  The device template, datastreams and a web dashboard and mobile app are all pre-configured.  Instructions are included later in this readme.md on how to configure the Particle integration webhook.  (TBD  will the Blynk webhook be automatically configured ???)


# 2. Create Webhooks on the Particle Cloud

# 3. Prepare Required Software

# 4. Prepare the Firmware and Upload It to Your Device

# 5. Dashboard Breakdown

# 6. Set up Automations 
[include this?]

# 7. Testing

# Conclusion

# Next Steps
The sketch sends an integer and floating point value every 5 minutes to Blynk.  The source of the integer and floating point value can be adapted to a particular application by the developer.  The interval default of 5 minutes can be reduced down to every 1 second.  It will be apparent to the developer from the content of the blueprint how to add additional data items to be pushed to Blynk.  &nbsp;

# Troubleshooting

# Related Links

