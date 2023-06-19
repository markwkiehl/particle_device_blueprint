# particle_device_blueprint
Connect any Particle IoT device to Blynk

# Introduction
This blueprint will configure bi-directional communication between any Particle device and Blynk IoT. &nbsp; Both a Blynk web dashboard and mobile app will be generated that will display data from the Particle device, and they will be used to control the Particle device remotely.  

# How It Works
Data is pushed from the Particle device to a Particle integration webhook. &nbsp; The webhook makes a call to the Blynk HTTP API and updates the Blynk datastreams on the Blynk cloud. &nbsp; The updated datastreams cause the Blynk web dashboard and mobile app widgets to be updated.  

Widgets on the Blynk web dashboard and mobile app trigger a Blynk webhook.  The webhook makes a Particle HTTP API call that sends data to the Particle device. &nbsp; A firmware function on the Particle device reacts to the data received and causes the hardware perform a custom action. 

All communication between the Particle device and Blynk will utilize the Blynk HTTP API rather than the [Blynk library](http://library.to/). &nbsp; This minimizes data usage when a cellular device is employed.  

## Functional Requirements
- 

# Components Used in This Project

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

