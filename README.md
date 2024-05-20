

# Tesla_M60_GPU_Cooler
Open source, esp8366 based, esphome + home assistant solution to Nvidia Tesla card cooling
# Intro
Having started a journey to try and hack together some cheap system that I might be able to run llm's using ollama on (my current headless server is a SFF and couldn't be upgraded) I came across the Nivida Tesla cards, these old cards are designed for server farms, have "loads" of VRAM and can be got for a pretty cheap price nowadays on ebay.

After having a look about and stumbling on [this github issue](https://github.com/ollama/ollama/issues/2250) it would appear that the Nvidia Tesla M60 should have support for [Ollama](https://ollama.com/), and being that I had seen that floating about on eBay, I pulled the trigger and purchased it.

The [M60](https://www.techpowerup.com/gpu-specs/tesla-m60.c2760) is a "dual GPU" card, meaning its basically 2 graphics cards in 1, each with 8GB of DDR4 ram, meaning 16GB all together, not bad for a card picked up for <£100...

One thing about these cards is that they have no active cooling, and looking around online there seemed to be a few solutions out there that people have come up with (I'm not the only person with this idea it would seem ;)) 

The first one I found was [this](https://www.thingiverse.com/thing:6038375), a 3d printable shroud that you can put on the end of the card and then use an old Dell radial fan - 
![enter image description here](https://cdn.thingiverse.com/assets/81/43/b7/08/b5/large_display_0d693aff-a07b-4d63-ab91-155a881d37fa.png)

I liked this idea, and I have a 3d printer, so ordered the fan off ebay £6, then started printing the shroud.....it didn't fit, this print was designed for an M40, and although the description said it should fit, it didn't seem to fit my M60, being it was a little wider, and the psu connector was in a different location, luckily, the original poster had shared the Fusion 360 file, so I remixed "hacked" it to fit the [M60](https://www.thingiverse.com/thing:6611626). (STL is on this repo), This design needs you to repin the fan and then it just connects to a fan header on the mobo, annoyingly, I didn't seem to have any spare.

The next project I came across, and one that I have based a lot of my logic on is the [Tesla Cooler](https://github.com/tesla-cooler) project, using a piPico and a number of smaller fans to cool the card, this time the project has extended the functionally by having a pico control the fans, allowing it to speed up/down depending on the temps, I liked this, but didn't like the fact that the temp was being "estimated" based on the calculated difference that the `nvidia smi` cli tool gave and the readings of his `ds18b20` temperature sensor connected to the side of the card.
![enter image description here](https://www.esologic.com/wp-content/uploads/2021/11/MG_5596-644x429.jpg)
I thought there must be a better way to get accurate temp readings from the box and get them into MQTT or Home Assistant, and again, someone else had the idea and created [Sensors2mqtt](https://github.com/koriwi/sensors2mqtt), scraping the nvidia smi, then posting the information to an MQTT server, as well as making entities in Home Assistant, exactly what I wanted, saved me some coding..

Having seen these 3 projects, I decided I would build my own as a mesh of the best items I saw in each.

 - 3d printed shroud that I have modified to fit my card
 - Dell fan purchased from ebay
         - No requirement to repin the fan
 - ESP3688 (D1 Mini and ESP-12e) as I have loads sitting about
	 - Design a PCB for ease of use in case
 - Use `nvidia smi` information for accurate readings
 - Use `ds18b20` as a fall back
	 - Option to switch between the 2 sensors
	 - Option to override either sensor and set speed
 - Basic PID Controller logic
 	- Set to 70°C
  	- Can be overwritten by Home Assistant 
 - Use Single 12V line from existing ATX PSU
 	- Power 12V Fan
# The Idea

So, Having looked at the provided projects, this is my approach, and should be yours if you are copying/following this...

 - Use sensors2mqtt to push nvidia-smi data to my already present MQTT server
	 - This will publish readings into `sensors2mqtt` topic
 - Using esp3688 and [ESPHome](https://esphome.io/index.html) to create a configuration that can 
	 - Connect to wifi
	 - Read temperature from published sensor reading on MQTT server
	 - Read temperature from local ds18b20 sensor
	 - All values and controls be visable via [Home Assistant](https://www.home-assistant.io/)
	 - Have a basic/configurable "Fan Scaling" against the GPU temp
		 - This is set to -
			- Below 70°C: Fan speed set to 25%.
			- Between 70°C and 82°C: Fan speed scales linearly from 31% to 75%.
			- Above 82°C: Fan speed set to 100%.

![image](https://github.com/karl0ss/Tesla_M60_GPU_Cooler/assets/2493260/772dd6d8-6c78-424f-b040-0d405d7db1bc)

![image](https://github.com/karl0ss/Tesla_M60_GPU_Cooler/assets/2493260/5208d1f0-97dd-4c7e-82b3-e253288f31ae)

![image](https://github.com/karl0ss/Tesla_M60_GPU_Cooler/assets/2493260/f3e7f10e-381e-4672-bde9-14616baf444d)


[Link to working driver](https://www.nvidia.com/Download/driverResults.aspx/222684/en-us/)
![image](https://github.com/karl0ss/Tesla_M60_GPU_Cooler/assets/2493260/93abcb32-f536-4157-9d0a-08d4d52167a3)


# TeslaGPUFan - ESP8266 Fan Controller

TeslaGPUFan is a project designed to control a fan based on GPU temperature readings obtained via MQTT or a Dallas temperature sensor, using an ESP8266 microcontroller. This setup provides flexibility to switch between automatic temperature-based control and manual override, integrating seamlessly with Home Assistant and supporting MQTT for advanced automation.

## Overview

TeslaGPUFan leverages the ESPhome framework to manage a GPU fan, ensuring efficient cooling based on real-time temperature readings from GPUs. The configuration includes various features that enhance its functionality and usability.

## Features

### Wi-Fi Connectivity
- **Primary Wi-Fi Connection**: Connects to a specified Wi-Fi network for primary operations.
- **Fallback Access Point**: Provides an access point fallback to ensure the device remains accessible if the primary network is unavailable.
- **Static IP Configuration**: Uses a static IP address for stable network communication.

### MQTT Integration
- **MQTT Communication**: Connects to an MQTT broker to receive GPU temperature data from `sensors2mqtt`.
- **GPU Temperature Monitoring**: Subscribes to MQTT topics to get temperature readings for GPU 1 and GPU 2.
- **Automatic Fan Control**: Sets the fan to 100% speed in case of MQTT disconnection to prevent overheating.
- **Maximum Temperature Control**: Uses the maximum temperature of GPU 1 and GPU 2 to control the fan speed, ensuring the highest temperature is always considered.

### Dallas Temperature Sensor
- **Fallback Temperature Monitoring**: Uses a Dallas temperature sensor as a fallback for monitoring temperature if GPU temperature readings are not available.
- **Lower Accuracy**: Not as accurate as GPU readings, hence used only as a fallback.

### Temperature-Based Fan Speed Control
- **Below 30°C**: Fan speed set to 10%.
- **Between 30°C and 60°C**: Fan speed scales linearly from 10% to 70%.
- **Above 60°C**: Fan speed set to 100%.

### Manual and Automatic Control
- **Automatic Control**: Adjusts fan speed based on the highest GPU temperature or the Dallas sensor temperature when GPU readings are unavailable.
- **Manual Override**: Allows manual control of the fan speed via Home Assistant, overriding automatic adjustments when needed.

### Home Assistant Integration
- **Device Monitoring**: Provides sensors for Wi-Fi signal strength, uptime, and fan speed.
- **Control Switches**: Includes switches to toggle between using GPU temperature or Dallas sensor and to enable/disable manual override.
- **Fan Speed Adjustment**: Offers a slider in Home Assistant for manual fan speed control.

### OTA Updates
- **Over-the-Air Updates**: Supports OTA updates to allow easy firmware upgrades without physical access to the device.

### Logging and Diagnostics
- **Detailed Logging**: Enables comprehensive logging for troubleshooting and diagnostics.
- **Diagnostic Sensors**: Includes sensors for monitoring Wi-Fi signal strength, uptime, and fan RPM.

### Scripts and Intervals
- **Fan Speed Adjustment Script**: A script that adjusts the fan speed based on the selected temperature sensor (GPU or Dallas) and predefined thresholds.
- **Regular Updates**: Intervals are set to frequently update GPU temperatures and execute the fan speed adjustment script to ensure timely response to temperature changes.

## Summary

TeslaGPUFan provides a comprehensive solution for managing a GPU fan using an ESP8266 microcontroller. Its features ensure efficient cooling through automatic temperature-based control while offering the flexibility of manual override. The integration with Home Assistant and support for MQTT further enhance its capabilities, making it a robust and versatile fan controller for various applications. The fallback to a Dallas temperature sensor ensures continued operation even when GPU readings are unavailable, though it is less accurate. This setup guarantees that the highest GPU temperature is always prioritized for fan control, providing optimal cooling performance based on the following scaling:

- **Below 30°C**: Fan speed set to 10%.
- **Between 30°C and 60°C**: Fan speed scales linearly from 10% to 70%.
- **Above 60°C**: Fan speed set to 100%.

With these features, TeslaGPUFan ensures that your GPU stays cool under varying thermal conditions, providing both reliability and flexibility.
