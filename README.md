# One relay antenna switch base on ESPHome

This directory contains a yaml file for an ESPHome project that can be used to switch between two HF antennas using a relay.

For more information on how to program an ESP32 with ESPHome, please refer to the [ESPHome documentation](https://esphome.io/).

The switch between the two antennas can be controlled by an OpenWebRX+ plugin using the project [OWRX_Antenna_manager](https://github.com/fustinoni-net/OWRX_Antenna_manager)

## Hardware 
The hardware used in this project is:
- an ESP32 microcontroller compatible with ESPHome
- A relay module like KY-019.
- a 5V power supply.
- 3 antennas connectors.
- a metal case.
- some wires.

The relay is connected to the GPIO pin 12 of the ESP32, and it is used to switch between two HF antennas.

## Yaml configuration
Go through the antenna-switch.yaml file and change the following parameters:
- `mqtt_broker`: the IP address of your MQTT broker.
- `mqtt_user`: the username for your MQTT broker.
- `mqtt_password`: the password for your MQTT broker.

Optionally you can configure the WI-FI parameters or just configure them later using the captive portal reachable via the device access point.
Of course, you can change the name of the device and the GPIO pin used to control the relay and all the other parameters.

## Control methods

The switch between the two antennas can be controlled by:
- using the messages sent by OpenWebRX+ over a MQTT server see: [OpenWebRX+ Advanced MQTT Reporting](https://fms.komkon.org/OWRX/#HOW-MQTT).
- using the web interface of the ESPHome device.
- using HTTP requests.
- integrated into [Home Assistant](https://www.home-assistant.io/).
- controlled by an OpenWebRX+ plugin using the project [OWRX_Antenna_manager](https://github.com/fustinoni-net/OWRX_Antenna_manager)

### Control by OpenWebRX+ MQTT messages

The MQTT messages are elaborated inside the lambda function in the `on_json_message` block. 
  
```
  on_json_message:
    topic: openwebrx/RX
    then:
      - lambda: |-
          if ( id(owrx_enable).state){
            if ( x.containsKey("source_id") && x["source_id"] == "rtlsdr") {
              if (x.containsKey("profile_id") && strstr(x["profile_id"], "antenna_1") != NULL) {
                id(last_used_profile_id) = std::string(x["profile_id"].as<const char*>());
                id(antenna).turn_on();
              } else {
                id(antenna).turn_off();
              }
              if (x.containsKey("state") && strstr(x["state"], "Running") != NULL && id(last_used_profile_id).find("antenna_1") != std::string::npos) {
                id(antenna).turn_on();
              }
            }
          }
```

The lambda function checks if the relay should be controlled directly by the OpenWebRX+ messages.
```
 if ( id(owrx_enable).state)
```
and then check if the source_id is "rtlsdr" and the profile_id contains "antenna_1" to turn on the relay.
```
if ( x.containsKey("source_id") && x["source_id"] == "rtlsdr") {
  if (x.containsKey("profile_id") && strstr(x["profile_id"], "antenna_1") != NULL) {

```
So you should change the two strings "rtlsdr" and "antenna_1" to match your OpenWebRX+ configuration.

For more information on how to program an ESP32 with ESPHome, please refer to the [ESPHome documentation](https://esphome.io/).

### Control using the web interface of the ESPHome device

Connect to the web interface of the device using: http://antenna-switch.local or http://<ip_address_of_the_device>. 
Then click on the "Switch" button to switch between the two antennas.

### Control using HTTP requests
You can control the relay using HTTP requests. The following endpoints are available:
- http://antenna-switch.local/switch/antenna_switch_1_2/turn_off
- http://antenna-switch.local/switch/antenna_switch_1_2/turn_on

### Control using Home Assistant
Integrate the ESPHome device into Home Assistant.

### Control using OWRX_Antenna_manager

Follow the instruction in the project [OWRX_Antenna_manager](https://github.com/fustinoni-net/OWRX_Antenna_manager)
