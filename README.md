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

## Code

```cpp
#include <ESP8266WiFi.h>
#include "Adafruit_MQTT.h"
#include "Adafruit_MQTT_Client.h"

// Define relay pin
#define Relay1 D0

// WiFi credentials
#define WLAN_SSID       "WiFi Name"             // Your SSID
#define WLAN_PASS       "H8VyT^vt4CtyBY"             // Your password

/* Adafruit.io Setup */
#define AIO_SERVER      "io.adafruit.com"
#define AIO_SERVERPORT  1883                   // Use 8883 for SSL
#define AIO_USERNAME    "HomeAutomation"             // Replace with your username
#define AIO_KEY         "aio_jbtS71OQNYyVTyvzQkDdcfoqnvgvhveS" // Replace with your Project Auth Key

/** Global State (you don't need to change this!) **/

// Create an ESP8266 WiFiClient class to connect to the MQTT server.
WiFiClient client;
// or... use WiFiFlientSecure for SSL
// WiFiClientSecure client;

// Setup the MQTT client class by passing in the WiFi client and MQTT server and login details.
Adafruit_MQTT_Client mqtt(&client, AIO_SERVER, AIO_SERVERPORT, AIO_USERNAME, AIO_KEY);

/** Feeds ***/

// Setup a feed called 'onoff' for subscribing to changes.
Adafruit_MQTT_Subscribe Light1 = Adafruit_MQTT_Subscribe(&mqtt, AIO_USERNAME "/feeds/Bulb for test"); // FeedName

void MQTT_connect();

void setup() {
  Serial.begin(115200);

  pinMode(Relay1, OUTPUT);
  digitalWrite(Relay1, HIGH);

  // Connect to WiFi access point.
  Serial.println(); Serial.println();
  Serial.print("Connecting to ");
  Serial.println(WLAN_SSID);

  WiFi.begin(WLAN_SSID, WLAN_PASS);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  // Setup MQTT subscription for onoff feed.
  mqtt.subscribe(&Light1);
}

void loop() {
  MQTT_connect();

  Adafruit_MQTT_Subscribe *subscription;
  while ((subscription = mqtt.readSubscription(5000))) {
    if (subscription == &Light1) {
      Serial.print(F("Got: "));
      Serial.println((char *)Light1.lastread);
      int Light1_State = atoi((char *)Light1.lastread);
      digitalWrite(Relay1, !(Light1_State));
    }
  }
}

void MQTT_connect() {
  int8_t ret;

  // Stop if already connected.
  if (mqtt.connected()) {
    return;
  }

  Serial.print("Connecting to MQTT... ");

  uint8_t retries = 3;

  while ((ret = mqtt.connect()) != 0) { // connect will return 0 for connected
    Serial.println(mqtt.connectErrorString(ret));
    Serial.println("Retrying MQTT connection in 5 seconds...");
    mqtt.disconnect();
    delay(5000);  // Wait 5 seconds
    retries--;
    if (retries == 0) {
      // Basically die and wait for WDT to reset me
      while (1);
    }
  }
  Serial.println("MQTT Connected!");
}

// Telegram bot part

else {
  String chat_id = String(bot.messages[i].chat_id);
  String text = bot.messages[i].text;

  if (text == F("/options")) {

    String keyboardJson = F("[[{ \"text\" : \"L1 ON\", \"callback_data\" : \"l1-on\" },{ \"text\" : \"L1 OFF\", \"callback_data\" : \"l1-off\" }],[{ \"text\" : \"L2 ON\", \"callback_data\" : \"l2-on\" },{ \"text\" : \"L2 OFF\", \"callback_data\" : \"l2-off\" }],[{ \"text\" : \"L3 ON\", \"callback_data\" : \"l3-on\"},{ \"text\" : \"L3 OFF\", \"callback_data\" : \"l3-off\" }],[{ \"text\" : \"L4 ON\", \"callback_data\" : \"l4-on\" },{ \"text\" : \"L4 OFF\", \"callback_data\" : \"l4-off\" }],[{ \"text\" : \"F1 ON\", \"callback_data\" : \"f1-on\" },{ \"text\" : \"F1 OFF\", \"callback_data\" : \"f1-off\" }],[{ \"text\" : \"F2 ON\", \"callback_data\" : \"f2-on\" },{ \"text\" : \"F2 OFF\", \"callback_data\" : \"f2-off\" }],[{ \"text\" : \"F3 ON\", \"callback_data\" : \"f3-on\" },{ \"text\" : \"F3 OFF\", \"callback_data\" : \"f3-off\" }],[{ \"text\" : \"F4 ON\", \"callback_data\" : \"f4-on\" },{ \"text\" : \"F4 OFF\", \"callback_data\" : \"f4-off\" }]]");
    bot.sendMessageWithInlineKeyboard(chat_id, "Home AUTOMATION BUTTONS(L,F indicate Light,Fan: )", "", keyboardJson);
  }
  if (text == F("/start")) {
    bot.sendMessage(chat_id, "/options : Returns the buttons for ON & OFF\n", "Markdown");
  }
}

void loop() {
  if (millis() - bot_lasttime > BOT_MTBS) {
    int numNewMessages = bot.getUpdates(bot.last_message_received + 1);
    while (numNewMessages) {
      Serial.println("got response");
      handleNewMessages(numNewMessages);
      numNewMessages = bot.getUpdates(bot.last_message_received + 1);
    }

    bot_lasttime = millis();
  }
}
