# Tesla_M60_GPU_Cooler: An ESP8266-Based Solution for Nvidia Tesla Card Cooling

## Introduction

Starting a journey to build a cost-effective system for running large language models (LLMs) using [Ollama](https://ollama.com/) led me to discover Nvidia Tesla cards. These older cards, designed for server farms, boast substantial VRAM and are now available at affordable prices on eBay. After finding out that the Nvidia Tesla M60 supports Ollama and purchasing one, I realized these cards lack active cooling, necessitating a DIY solution for effective cooling.

## Overview of Existing Solutions

### 3D Printable Shroud with Dell Radial Fan

The first solution I encountered was a 3D printable shroud designed for the M40 card, which could be paired with a Dell radial fan. However, the shroud didn't fit my M60 due to dimensional differences and PSU connector placement. Modifying the Fusion 360 file, I created a shroud specifically for the M60. This design still required repinning the fan, which wasn't ideal as I lacked spare fan headers on my motherboard.

![3D Printable Shroud](https://cdn.thingiverse.com/assets/81/43/b7/08/b5/large_display_0d693aff-a07b-4d63-ab91-155a881d37fa.png)

### PiPico-Based Fan Controller

The next project, [Tesla Cooler](https://github.com/tesla-cooler), used a PiPico and several small fans to cool the card. This setup controlled fan speed based on estimated temperatures derived from the `nvidia-smi` tool and a `ds18b20` temperature sensor. While innovative, I wanted a more accurate method for obtaining temperature readings and integrating them into Home Assistant.

![PiPico-Based Fan Controller](https://www.esologic.com/wp-content/uploads/2021/11/MG_5596-644x429.jpg)

### Sensors2mqtt for Accurate Temperature Readings

[Sensors2mqtt](https://github.com/koriwi/sensors2mqtt) provided a solution by scraping `nvidia-smi` data and posting it to an MQTT server, creating entities in Home Assistant. This approach offered the accuracy I needed without extensive coding, setting the foundation for my custom project.

## My Custom Solution

Integrating the best elements of these projects, I developed a comprehensive cooling system for the Tesla M60:

- **3D Printed Shroud**: Custom-designed to fit the M60, avoiding the need for repinning the fan.
- **Dell Fan**: Purchased from eBay, requiring no repinning.
- **ESP8266 Microcontroller**: Utilized for controlling the fan and connecting to Wi-Fi.
- **Nvidia SMI Data**: Used for accurate temperature readings.
- **DS18b20 Sensor**: Added as a fallback.
- **PID Controller Logic**: Maintains GPU temperature at 70°C, adjustable via Home Assistant.
- **Single 12V Line from ATX PSU**: Powers the 12V fan.
- **Custom PCB Design**: Designed a custom PCB for the ESP-12e to simplify the setup and ensure robust connections.

## The Implementation Plan

### Components and Tools

- **3D Printer**: To print the custom shroud.
- **Dell Radial Fan**: For active cooling.
- **ESP8266 (D1 Mini and ESP-12e)**: For microcontroller-based control.
- **Home Assistant**: For integration and control.
- **MQTT Server**: For data communication.
- **Nvidia SMI Tool**: For accurate temperature readings.
- **DS18b20 Temperature Sensor**: As a fallback option.
- **Custom PCB for ESP-12e**: To ensure reliable connections and ease of setup.

### Configuration with ESPHome and Home Assistant

Using [ESPHome](https://esphome.io/index.html), the ESP8266 microcontroller is configured to:

- **Connect to Wi-Fi**: Ensuring stable network communication.
- **Subscribe to MQTT Topics**: Receiving GPU temperature data.
- **Monitor Temperatures**: From both the GPU and DS18b20 sensor.
- **Control Fan Speed**: Based on temperature readings with predefined thresholds.
- **Integrate with Home Assistant**: Allowing manual override and monitoring.

#### Fan Speed Control Logic

- **Below 30°C**: Fan speed set to 10%.
- **30°C to 60°C**: Fan speed scales linearly from 10% to 70%.
- **Above 60°C**: Fan speed set to 100%.

### Home Assistant Integration

In [Home Assistant](https://www.home-assistant.io/), the following functionalities are implemented:

- **Device Monitoring**: Sensors for Wi-Fi signal strength, uptime, and fan speed.
- **Control Switches**: To toggle between GPU and Dallas sensor, and enable/disable manual override.
- **Fan Speed Adjustment**: Via a slider for manual control.
- **OTA Updates**: For easy firmware upgrades.

## Summary

TeslaGPUFan provides a robust solution for controlling a GPU fan using an ESP8266 microcontroller. It combines the accuracy of `nvidia-smi` temperature readings with the fallback reliability of a DS18b20 sensor. The integration with Home Assistant and MQTT allows for advanced automation and manual control, ensuring efficient cooling under varying conditions. This project showcases the potential of DIY solutions in enhancing the functionality of affordable, high-performance hardware like the Nvidia Tesla M60.

![TeslaGPUFan Setup](https://github.com/karl0ss/Tesla_M60_GPU_Cooler/assets/2493260/772dd6d8-6c78-424f-b040-0d405d7db1bc)

![TeslaGPUFan](https://github.com/karl0ss/Tesla_M60_GPU_Cooler/assets/2493260/5208d1f0-97dd-4c7e-82b3-e253288f31ae)

![TeslaGPUFan](https://github.com/karl0ss/Tesla_M60_GPU_Cooler/assets/2493260/f3e7f10e-381e-4672-bde9-14616baf444d)

[Link to working driver](https://www.nvidia.com/Download/driverResults.aspx/222684/en-us)
