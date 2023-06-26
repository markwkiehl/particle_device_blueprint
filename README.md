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

Widgets on the Blynk web dashboard and mobile app trigger a Blynk webhook. &nbsp; The webhook makes a Particle HTTP API call that sends data to the Particle hardware. &nbsp; A firmware function on the Particle hardware reacts to the data received, and causes the hardware perform the custom action of toggling the state of the built-in LED. 

The Blynk webhook called by the Blynk web or mobile app widget makes a call to the Particle API for a Particle cloud function with a device unique token. &nbsp; The Particle cloud function running on the Particle hardware performs and action in response to the call, and responds to the API call with a result code.   

The blueprint includes a sketch (.ino file) that is uploaded to the Particle hardware. &nbsp; The Blynk device template, datastreams, web dashboard ,and mobile app (iOS/Android) are all pre-configured. &nbsp; Instructions are included later in this readme.md on how to activate a new Blynk device, configure the Particle integration webhook, and how to create a Blynk webhook. &nbsp;

# Components Used in This Project
- [Particle Console](https://console.particle.io/) to activate the hardware and set up the integration/webhook. 
- Blynk Console and [Blynk App](https://docs.blynk.io/en/downloads/blynk-apps-for-ios-and-android?_gl=1*hxem43*_ga*NTQ1NjUzMTkwLjE2NjY1NTA3MTk.*_ga_E376ZQ635Y*MTY4NzE3MTI1Mi44OC4xLjE2ODcxNzEyNTQuMC4wLjA.) for web and mobile dashboards
- Particle Web IDE, Particle Workbench or Particle Comand Line to upload firmware.
- Any of the following Particle IoT devices:
  - **Tracker One** is complete system including an enclosure. It also comes as a SoM (system on module) known as **Tracker SoM**.  See [Tracker button and LEDs](https://docs.particle.io/hardware/tracker/projects/tracker-buttons-leds/) for details on how to add an LED to the Tracker One. 
  - Any **Boron** cellular devices, including those that are depreciated. &nbsp; All have a built-in LED is on D7.
  - **B Series SoM** system-on-a-module. &nbsp; No built-in LED, must custom install.  
  - **Photon 2** is a WiFi device with built in LED is on D7.
  - **P2** is an SMD module with WiFi and a microcontroller.  No built-in LED.  Replaces the depreciated P1. 
  - **Argon** is a depreciated WiFi device with built in LED is on D7.
  - **Photon** is a depreciated WiFi device with built-in LED on D7.
  - **Electron** is a depreciated cellular device with built-in LED on D7.
  - **E Series** is a cellular device.  No built-in LED.
  - **Core** is a WiFi device with built-in LED on D7. 
 
# 1. Prepare Your Hardware
All Particle devices will require suitable power connection and either a cellular or WiFi connection. &nbsp; If your device has a built-in LED connected to GPIO D7 (Boron, Argon, Photon 2, Photon, Electron, Core) then nothing more is required. &nbsp; Devices without a built-in LED will be detected by the firmware and a virtual LED (Blynk widget) will be used instead. &nbsp;

# 2. Get the Firmware and a Blynk AuthToken
If you don't have a Blynk account, then create one. &nbsp; From the Blynk.Console, navigate to 'Templates' -> 'All Blueprints' -> 'Particle Device Blueprint'. &nbsp;
After selecting the blueprint, choose the option 'Activate New Device' to generate a Blynk AuthToken. &nbsp; Keep that AuthToken in a safe place because you will need it later for TBD. &nbsp;  Generating the AuthToken will also make available the firmware for your Particle hardware. &nbsp; Upload that firmware to your Particle hardware. &nbsp;

Note that the Blynk blueprint has created a device template with the following datastreams configured:

<table>
  <tr>
    <th>Datastream/Virtual Pin</th>
    <th>&nbsp;</th>
    <th>Data Type</th>
    <th>Comment</th>
  </tr>
  <tr>
    <td>V6</td>
    <td>last_publish</td>
    <td>String</td>
    <td>Last date/time data was published</td>
  </tr>
  <tr>
    <td>V14</td>
    <td>millis()</td>
    <td>integer</td>
    <td>Number of seconds since the device started</td>
  </tr>
  <tr>
    <td>V15</td>
    <td>3.14159</td>
    <td>double</td>
    <td>Simulated sensor data</td>
  </tr>
  <tr>
    <td>V16</td>
    <td>0/1</td>
    <td>integer</td>
    <td>Simulated LED on device</td>
  </tr>
</table>

# 3. Create Integration (Webhook) on the Particle Cloud
We are going to create a Particle integration webhook running on the Particle cloud that will accept the data from the Particle.publish() function executing on the device, and transform it into a HTTPs GET that will post data to the Blynk cloud, updating the corresponding Blynk datastream values. &nbsp;

Login to your [Particle Console](https://docs.particle.io/getting-started/console/console/) and click on the ‘[Integrations](https://console.particle.io/integrations)’ sidebar option. &nbsp; Click on the ‘NEW INTEGRATION’ shown on the page, and then select the ‘Webhook’ option. 

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/626bbdfa3d42f9a1716117e9dbb36f4d7ad7d650/particle_device_blueprint%20(1).jpeg "New Integration")

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/626bbdfa3d42f9a1716117e9dbb36f4d7ad7d650/particle_device_blueprint%20(2).jpeg "New Integration")

Fill out the Webhook form as shown below. &nbsp; The ‘Event Name’ is what will be called later by your device firmware with the Particle.publish() function. &nbsp; The server address of ‘ny3.blynk.cloud’ for the ‘URL’ field should be replaced with the server address from [this list](https://docs.blynk.io/en/blynk.cloud/troubleshooting) that matches what you defined for your Blynk HTTP GET request. &nbsp; Change the ‘Request Type’ to ‘GET’, and then click on the ‘CREATE WEBHOOK’ button at the bottom of the form. 

![alt text](https://github.com/markwkiehl/particle_device_blueprint/blob/939550ad3a628790ea4cafb09045f453b1617d15/particle_device_blueprint%20(5).jpeg "New Integration")

The webhook is not complete yet. &nbsp; The query parameters need to be defined. &nbsp; Click on the ‘EDIT’ button at the top right of the screen. &nbsp;

![alt text](https://github.com/markwkiehl/particle_device_blueprint/blob/e68aadfd6b90c7b41d69fd671cbedc7702aaac41/particle_device_blueprint%20(6).jpeg "New Integration")

Click on the ‘Advanced Settings’ link at the bottom of the page and then under the ‘QUERY PARAMETERS’ section, choose the ‘Custom’ option. &nbsp; Build the query parameters as shown below, using the ‘+ ADD ROW’ button at the bottom to add a row for each query parameter. 

# 4. Create a Webhook in the Blynk.Console

# 5. Prepare Required Software

# 6. Dashboard Breakdown

# 7. Testing

# Troubleshooting

# Conclusion
Using this Blynk blueprint quickly configures a Particle device for bi-directional communication with Blynk. &nbsp; Blynk provides a web dashboard and mobile app for visualizing data from the Particle hardware, and remotely controlling the Particle device.  

# Next Steps
The sketch sends an integer and floating point value every 5 minutes to Blynk. &nbsp; The source of the integer and floating point value can be adapted to a particular application by the developer. &nbsp; The interval default of 5 minutes can be reduced down to every 1 second. &nbsp; It will be apparent to the developer from the content of the blueprint how to add additional data items to be pushed to Blynk.  &nbsp;

# Related Links
[Connect Particle Device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk)

[How to Control a Particle Device with Blynk](https://blynk.io/blog/how-to-control-particle-device-with-blynk)


