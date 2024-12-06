# ESP8266 Home Automation with Telegram and MQTT

## Overview

This project enables home automation using the **ESP8266** microcontroller, **MQTT** for communication, and **Telegram** for remote control of various devices like lights and fans. It connects to Adafruit's MQTT service to control devices through messages received from Telegram commands. This system allows for controlling multiple appliances (e.g., lights and fans) by sending simple commands through the Telegram bot interface.

## Features

- Control devices (lights, fans, etc.) remotely via Telegram bot.
- Communication between the ESP8266 and Adafruit MQTT broker.
- Toggle devices ON/OFF using Telegram inline buttons.
- Uses MQTT to subscribe and control device states.

## Components Required

- **ESP8266 microcontroller**
- **Relay modules** for controlling appliances (lights, fans, etc.)
- **Wi-Fi network** for connecting the ESP8266 to the internet
- **Telegram bot** for sending commands to the ESP8266
- **MQTT broker (Adafruit IO)** to handle communication

## Code Functionality

1. **Wi-Fi Setup**: Connects to a Wi-Fi network using the specified SSID and password.
2. **MQTT Setup**: Subscribes to MQTT topics to receive commands for controlling devices.
3. **Telegram Commands**: Handles Telegram bot commands to control the relay modules.
4. **Relay Control**: Toggles relay pins based on commands received from the Telegram bot.

## How It Works

1. The ESP8266 connects to your Wi-Fi network.
2. It subscribes to the MQTT topics and listens for commands.
3. The Telegram bot sends commands (like ON/OFF for devices) to the MQTT broker.
4. The ESP8266 receives the commands and toggles the relays accordingly to control connected devices.

## Commands

- **/options**: Displays buttons for turning devices ON and OFF.
- **/start**: Provides information on available commands.

## Example Button Layout

```json
[
  [{"text" : "L1 ON", "callback_data" : "l1-on"}, {"text" : "L1 OFF", "callback_data" : "l1-off"}],
  [{"text" : "L2 ON", "callback_data" : "l2-on"}, {"text" : "L2 OFF", "callback_data" : "l2-off"}],
  [{"text" : "L3 ON", "callback_data" : "l3-on"}, {"text" : "L3 OFF", "callback_data" : "l3-off"}],
  [{"text" : "L4 ON", "callback_data" : "l4-on"}, {"text" : "L4 OFF", "callback_data" : "l4-off"}],
  [{"text" : "F1 ON", "callback_data" : "f1-on"}, {"text" : "F1 OFF", "callback_data" : "f1-off"}],
  [{"text" : "F2 ON", "callback_data" : "f2-on"}, {"text" : "F2 OFF", "callback_data" : "f2-off"}],
  [{"text" : "F3 ON", "callback_data" : "f3-on"}, {"text" : "F3 OFF", "callback_data" : "f3-off"}],
  [{"text" : "F4 ON", "callback_data" : "f4-on"}, {"text" : "F4 OFF", "callback_data" : "f4-off"}]
]
