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
- Communication frequency must be configured to minimize data usage by default for the case when a cellular device is employed.  

# How It Works
All communication between the Particle hardware and Blynk will utilize the Blynk HTTP API rather than the [Blynk library](http://library.to/). &nbsp; This minimizes data usage when a Particle cellular device is employed.  

The BLYNK AUTH TOKEN will be sent by the device with each data packet and the Particle webhook will pass this on when it communicates with Blynk via the Blynk HTTP API.  This allows the same Particle integration webhook to be used with multiple Particle devices.  

The firmware sketch sends an integer value from the millis() function (number of milliseconds elapsed since the device started) and a constant floating point value every 5 minutes. &nbsp; The data is pushed from the Particle hardware to a Particle integration webhook. &nbsp; The webhook makes a call to the Blynk HTTP API and updates the Blynk datastreams on the Blynk cloud. &nbsp; The updated datastreams cause the Blynk web dashboard and mobile app widgets configured for those datastreams to be updated.  

The Particle hardware will also be controlled remotely from the Blynk web dashboard or mobile app. &nbsp; When the state of the switch widget on the Blynk web dashboard and mobile app is changed, a Blynk webhook is called. &nbsp; The webhook makes a Particle HTTP API call to a Particle cloud function with a device unique token that sends data to the Particle hardware. &nbsp; A firmware function on the Particle hardware reacts to the data received, returns a result code, and causes the hardware perform the custom action of toggling the state of the built-in LED if the Particle hardware has one, otherwise a web dashboard and mobile app LED widget.  &nbsp;

The blueprint includes a sketch (.ino file) that is uploaded to the Particle hardware. &nbsp; The Blynk device template, datastreams, web dashboard ,and mobile app (iOS/Android) are all pre-configured. &nbsp; Instructions are included later in this readme.md on how to configure the Particle integration webhook, generate a Particle access token, and how to create a Blynk webhook. &nbsp;

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

<!-- Tracker One, Tracker SOM, Boron, B Series SoM, Photon 2, P2, Argon, Photon, Electron, E Series, Core -->

# 1. Prepare Your Hardware
All Particle devices will require suitable power connection and either a cellular or WiFi connection. &nbsp; If your device has a built-in LED connected to GPIO D7 (Boron, Argon, Photon 2, Photon, Electron, Core) then nothing more is required. &nbsp; Devices without a built-in LED will be detected by the firmware and a virtual LED (Blynk widget) will be used instead. &nbsp;

# 2. Get the Firmware and a Blynk AuthToken
If you don't have a Blynk account, then create one. &nbsp; From the Blynk.Console, navigate to 'Templates' -> 'All Blueprints' -> 'Particle Device Blueprint'. &nbsp;
After selecting the blueprint, choose the option 'Activate New Device' to generate a Blynk AuthToken and to get the firmware code. &nbsp; Keep that AuthToken in a safe place because you will need it later for TBD. &nbsp;  Generating the AuthToken will also make available the firmware for your Particle hardware. &nbsp; Upload that firmware to your Particle hardware. &nbsp;

Upload the firmware code to your Particle hardware (device). &nbsp; The Particle device should be breathing cyan if it has the built-in RGB and is connected to the Particle cloud. &nbsp; 

# 3. Create Integration (Webhook) on the Particle Cloud
We are going to create a Particle integration webhook running on the Particle cloud that will accept the data from the Particle.publish() function executing on the device, and transform it into a HTTPs GET that will post data to the Blynk cloud, updating the corresponding Blynk datastream values. &nbsp;

Login to your [Particle Console](https://docs.particle.io/getting-started/console/console/) and click on the ‘[Integrations](https://console.particle.io/integrations)’ sidebar option. &nbsp; Click on the ‘NEW INTEGRATION’ shown on the page, and then select the ‘Webhook’ option. 

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/626bbdfa3d42f9a1716117e9dbb36f4d7ad7d650/particle_device_blueprint%20(1).jpeg "New Integration")

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/626bbdfa3d42f9a1716117e9dbb36f4d7ad7d650/particle_device_blueprint%20(2).jpeg "New Integration")

Fill out the Webhook form as shown below. &nbsp; The ‘Event Name’ is what will be called later by your device firmware with the Particle.publish() function, so it is critical that it matches what is called in the firmware. &nbsp; The server address of ‘ny3.blynk.cloud’ for the ‘URL’ field should be replaced with the server address from [this list](https://docs.blynk.io/en/blynk.cloud/troubleshooting) that matches what you defined for your Blynk HTTP GET request. &nbsp; Change the ‘Request Type’ to ‘GET’, and then click on the ‘CREATE WEBHOOK’ button at the bottom of the form. 

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/c1409a9c9ea508fe0db27a512207736e520356f9/particle_device_blueprint%20(5).png "New Integration")

The webhook is not complete yet. &nbsp; The query parameters need to be defined. &nbsp; Click on the ‘EDIT’ button at the top right of the screen. &nbsp;

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/e68aadfd6b90c7b41d69fd671cbedc7702aaac41/particle_device_blueprint%20(6).jpeg "New Integration")

Click on the ‘Advanced Settings’ link at the bottom of the page and then under the ‘QUERY PARAMETERS’ section, choose the ‘Custom’ option. &nbsp; Build the query parameters as shown below, using the ‘+ ADD ROW’ button at the bottom to add a row for each query parameter. &nbsp;

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/793c4a262e7c851ebd94a324c49ecb0291337265/particle_device_blueprint%20(7).png "New Integration")

The keys on the left (token, V6, V14, V15) refer to Blynk virtual pins (datastreams), and the values on the right for ‘{{t}} {{PARTICLE_PUBLISHED_AT}} {{v14}} {{v15}}’ reference variables from the firmware that will be passed from the Particle.publish() function. &nbsp; Those values passed by the Particle.publish() function will replace the placeholders in the '{{}}' Mustache. &nbsp; The value ‘PARTICLE_PUBLISHED_AT’ for virtual pin V6 is a Particle pre-defined variable that provides a UTC timestamp for when the webhook is executed. &nbsp;

At the bottom of the form, make sure the ‘ENFORCE SSL’ option is set to ‘Yes’, and then click the ‘SAVE’ button to save your changes. 

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/329686e5ebe01144375108bea2e7a770c57c4100/particle_device_blueprint%20(8).jpeg "New Integration")

After you save your webhook, a summary of the configuration will be shown. &nbsp; Make sure it matches exactly what is shown below (except for the server address).

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/565ce0266ea6be870f859dd653dd0d3bd595fbf2/particle_device_blueprint%20(9).png "New Integration")

Note that blueprint is designed to pass the unique BLYNK_AUTH_TOKEN defined in the firmware for each device to the Particle webhook as the variable ‘{{t}}’. &nbsp; This allows each device to call the same webhook, at the expense of increasing the cellular payload for each transmission by 32 bytes. &nbsp; You can learn more about Particle webhooks by visiting this [documentation link](https://docs.particle.io/reference/cloud-apis/webhooks/). 

# 4. Create a Particle Access Token

The Blynk webhook will need a Particle access token in order to make a Particle HTTP API call to the Particle cloud function. &nbsp;

Create an access token by logging into your Particle account, and then visiting the Particle documentation section ‘[Create a token (browser-based)](https://docs.particle.io/reference/cloud-apis/access-tokens/#create-a-token-browser-based-)’. Use your Particle login email and password. &nbsp; If you have MFA (multi-factor authentication) enabled on your account, you will need your MFA code to generate the access token. Click the ‘Create token’ button to generate a token. &nbsp; Keep this token confidential. 

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/fa812deaaa3100b08566136fa750a2a3e0389f4e/particle_device_blueprint%20(10).jpeg "Particle Access Token")

# 5. Create a Webhook in the Blynk.Console

Control of the Particle hardware remotely from the Blynk web dashboard or mobile app is accomplished using a Blynk webhook and the Particle HTTP API. &nbsp; When the state of the switch widget on the Blynk web dashboard or mobile app is changed, a Blynk webhook assigned to the same datastream is called. &nbsp; The webhook makes a Particle HTTP API call to a Particle cloud function with a device unique token that sends data to the Particle hardware. &nbsp; 

In the [Blynk.Console](https://blynk.cloud/), navigate to ‘Settings -> Webhooks’ and create one new webhooks for datastream V16 based on the information shown in the image below. &nbsp; Substitute your Particle [device ID](https://console.particle.io/) and the 40 character access token you generated earlier in the corresponding form fields. &nbsp;  Your 24 character Particle device ID is available in the [Particle console[(https://console.particle.io/). &nbsp; 

The 'DEVICE' field in the form is the Blynk device, selectable from a drop down list. &nbsp;

For the 'WEBHOOK URL' field in the form, the format is:
<pre><code>https://api.particle.io/v1/devices/[your 24 character Particle device ID]/blynk_led</code></pre>

The "blynk_led" at the end of the WEBHOOK URL is the Particle cloud function key that is referenced in the firmware as:

<pre><code>
void setup() {
  ...
  Particle.function("blynk_led", blynkLED);
  ...
}
</code></pre>

The 'HTTP Headers' with key "Authorization" has a value consisting of the string "Bearer " (with a space after it), and then followed by the 40 character Particle access token. &nbsp;

After you are finished configuring each webhook, click the ‘Test webhook’ to verify it doesn’t throw an error (it won’t send the datastream value here, so don’t expect to see the LED on your Particle device change). &nbsp; Click the ‘Create Webhook’ button to save it and close the dialog. 

Note that the Blynk request quota is ‘1 per minute’ so any datastream value changes sooner than 60 seconds will not execute the webhook. &nbsp;

![alt text](https://github.com/markwkiehl/particle_device_blueprint/raw/4551a69255324fa9f0c551440f7514cb6764b464/particle_device_blueprint%20(11).png "Blynk Webhook")

# 8. Testing

Test the Particle cloud function running in the firmware by calling it from the Particle console. &nbsp;  With your Particle hardware running, visit [here](https://docs.blynk.io/en/hardware-guides/particle-part-ii#firmware) for detailed instructions on how to call 'blynk_led'. &nbsp; Go to your [Particle console](https://console.particle.io/), select the Particle device, and then under the section ‘FUNCTIONS’ on the right side of the screen you will see the function key of ‘blynk_led’ listed. &nbsp; Enter ‘on’ in the ‘Argument’ input area and click the ‘CALL’ button. &nbsp; Observe the Particle device to confirm that the built-in blue LED on D7 turns on. &nbsp; Repeat, this time with the ‘off’ argument to turn off the LED. &nbsp;

![alt text](https://github.com/markwkiehl/particle_device_blueprint/blob/b1bf502a9f0161de09dced117001832adae37930/particle_device_blueprint%20(12).jpg "Particle Function")

Verify that your Particle access token is correct by using the Particle API to test it. &nbsp; Detailed instructions on how to do this using [Postman](https://www.postman.com/) can be found [here](https://docs.blynk.io/en/hardware-guides/particle-part-ii#particle-api). &nbsp;

Test the Blynk webhook by installing the firmware on your Particle hardware, and then click the ‘Test webhook’ to verify it doesn’t throw an error (it won’t send the datastream value here, so don’t expect to see the LED on your Particle device change). &nbsp;  Then from the Blynk web dashboard or mobile app, toggle the switch assigned to datastream V16 and observe the built-in LED on the hardware if it exists, or the Blynk LED widget if no built-in LED exists. &nbsp;  Wait 60 seconds between each toggle of the switch widget. &nbsp;

# Reference Information

The Blynk blueprint has created a device template with the following datastreams configured:
<table>
  <tr>
    <th>Datastream <br/> Virtual Pin</th>
    <th>Data Type</th>
    <th>&nbsp;</th>
    <th>Comment</th>
  </tr>
  <tr>
    <td>V6</td>
    <td>String</td>
    <td>last_publish</td>
    <td>Last UTC date/time data was published from the Particle hardware</td>
  </tr>
  <tr>
    <td>V14</td>
    <td>integer</td>
    <td>millis()</td>
    <td>Simulated sensor data</td>
  </tr>
  <tr>
    <td>V15</td>
    <td>double</td>
    <td>3.14159</td>
    <td>Simulated sensor data</td>
  </tr>
  <tr>
    <td>V16</td>
    <td>integer</td>
    <td>0/1</td>
    <td>Controlled by switch widget on Blynk web dashboard and mobile app. &nbsp; Changes in the datastream value will be received by the hardware and then the state of the built-in LED will be changed (if available), and the state of the LED widget on the Blynk web dashboard and mobile app will be updated.</td>
  </tr>
  <tr>
    <td>V17</td>
    <td>integer</td>
    <td>0/1</td>
    <td>LED widget on Blynk web dashboard and mobile app that echos changes received by the hardware for V16. 
	</td>
  </tr>
</table>

# Troubleshooting
If your Particle device has a built-in RGB then it should be breathing cyan if it is connected to the Particle cloud.  &nbsp; Make sure the firmware code provided by activating the Blynk device was uploaded to the hardware. &nbsp;

Perform all of the tests under **Testing** to be sure that each communication step from the Particle hardware to Blynk and back works properly. &nbsp;

# Conclusion
Using this Blynk blueprint accelerates the configuration of a Particle device for bi-directional communication with Blynk. &nbsp; The Blynk blueprint generates the firmware to be installed on the Particle hardware, creates a Blynk device template with datastreams defined, a web dashboard with widgets, and a mobile app with widgets. &nbsp; The developer needs to create a Blynk webhook, a Particle webhook, provision a Blynk device and upload the firmware to the hardware, and acquire a Particle access token. &nbsp;

# Next Steps
The resources generated by this blueprint are easily modified to include more functionality. &nbsp; Blynk provides a NO CODE web dashboard and mobile app for visualizing data from the Particle hardware, and remotely controlling the Particle device.  &nbsp;

The firmwate sketch sends an integer and floating point value every 5 minutes to Blynk. &nbsp; The source of the integer and floating point value can be adapted to a particular application by the developer, such as a sensor value. &nbsp; Additional data can be sent from the hardware to Blynk by modifying the firmware, the Particle webhook, and adding datastreams to the Blynk template. &nbsp; 

The data publishing interval default of 5 minutes set in the firmware can be reduced down to every one second. &nbsp; 

# Related Links
[Connect Particle Device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk)

[How to Control a Particle Device with Blynk](https://blynk.io/blog/how-to-control-particle-device-with-blynk)


