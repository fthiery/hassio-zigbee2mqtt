# Hass.io Add-on: zigbee2mqtt

Add-on for running [zigbee2mqtt](https://github.com/Koenkk/zigbee2mqtt) in [Hass.io](https://github.com/home-assistant/hassio).

## Usage

### Installation

- Add the [repository URL](https://github.com/danielwelch/hassio-zigbee2mqtt) in your **Hass.io > Add-on Store**

The addon should now be available for installation.

### Configuration

To configure this add-on, you must set the following parameters via the Hass.io user interface. See the [zigbee2mqtt docs](https://github.com/Koenkk/zigbee2mqtt/wiki/Running-the-bridge) and the [default configuration file](https://github.com/Koenkk/zigbee2mqtt/blob/master/data/configuration.yaml) for more information.

|Parameter|Type|Required|Description|
|---------|----|--------|-----------|
|`data_path`|string|Yes|Set this to the path you'd like the add-on to persist data. Must be within the `/share` directory. Defaults to `/share/zigbee2mqtt`.|
|`homeassistant`|bool|yes|Set this to `true` if you want MQTT autodiscovery. See [Integrating with Home Assistant](https://github.com/Koenkk/zigbee2mqtt/wiki/Integrating-with-Home-Assistant) for details.|
|`permit_join`|bool|yes|Recommended to leave this to `false` and use [runtime pairing](https://github.com/danielwelch/hassio-zigbee2mqtt#pairing). Set this to `true` when you setup new devices - make sure you set it back to `false` when done.|
|`mqtt_server`|string|yes|Prefix for your MQTT topic|
|`mqtt_base_topic`|string|yes|The MQTT server address. Make sure you include the protocol. Example: `mqtt://homeassistant`|
|`serial_port`|string|yes|Serial port for your CC2531 stick.|
|`mqtt_user`|string|no|Your MQTT username, if set.|
|`mqtt_pass`|string|no|Your MQTT Password, if set.|
|`debug`|bool|no|Set to true to enable debug mode for zigbee-shepherd and zigbee2mqtt. See [the wiki](https://github.com/Koenkk/zigbee2mqtt/wiki/How-to-debug) for more information.|
|`err`|bool|no|Set to true to redirect zigbee2mqtt `stdout` to `out.log` and `stderr` to `err.log`. Both `out.log` and `err.log` will be located within `data_path` above.|
|`commit`|string|no|Set this to a specific `zigbee2mqtt` commit SHA hash to use a specific version of `zigbee2mqtt` (in case of regressions)|

Notes:
- Depending on your configuration, the MQTT server URL will need to include the port, typically `1883` or `8883` for SSL communications. For example, `mqtt://homeassistant:1883`.
- To find out which serial ports you have exposed go to **Hass.io > System > Host system > Show Hardware**

### Pairing

The suggested way to pair your devices is to enable zigbee2mqtt's `permit_join` option from within Home Assistant using MQTT rather than through the add-on's User Interface. Below is an example configuration that will allow you to enable and disable device pairing from the Home Assistant front end:

<img width="503" alt="screen shot 2018-06-02 at 14 41 42" src="https://user-images.githubusercontent.com/7738048/40874668-bdd1645a-667a-11e8-88ff-03b78212910b.png">

```yaml
mqtt:
  broker: homeassistant # This will have to be your mqtt broker
  discovery: true

input_boolean:
  zigbee_permit_join:
    name: Allow devices to join
    initial: off
    icon: mdi:cellphone-wireless

timer:
  zigbee_permit_join:
    name: Time remaining
    duration: 600 # Updated this to the number of seconds you wish

sensor:
  - platform: mqtt
    name: Bridge state
    state_topic: "zigbee2mqtt/bridge/state"
    icon: mdi:router-wireless

group:
  zigbee_group:
    name: Zigbee
    entities:
      - input_boolean.zigbee_permit_join
      - timer.zigbee_permit_join
      - sensor.bridge_state

automation:
  - id: enable_zigbee_join
    alias: Enable Zigbee joining
    hide_entity: true
    trigger:
      platform: state
      entity_id: input_boolean.zigbee_permit_join
      to: 'on'
    action:
    - service: mqtt.publish
      data:
        topic: zigbee2mqtt/bridge/config/permit_join
        payload: 'true'
    - service: timer.start
      data:
        entity_id: timer.zigbee_permit_join
  - id: disable_zigbee_join
    alias: Disable Zigbee joining
    trigger:
    - entity_id: input_boolean.zigbee_permit_join
      platform: state
      to: 'off'
    action:
    - data:
        payload: 'false'
        topic: zigbee2mqtt/bridge/config/permit_join
      service: mqtt.publish
    - data:
        entity_id: timer.zigbee_permit_join
      service: timer.cancel
    hide_entity: true
  - id: disable_zigbee_join_timer
    alias: Disable Zigbee joining by timer
    hide_entity: true
    trigger:
    - platform: event
      event_type: timer.finished
      event_data:
        entity_id: timer.zigbee_permit_join
    action:
    - service: mqtt.publish
      data:
        topic: zigbee2mqtt/bridge/config/permit_join
        payload: 'false'
    - service: input_boolean.turn_off
      data:
        entity_id: input_boolean.zigbee_permit_join
```
Notes:
- There is a [gist](https://gist.github.com/ciotlosm/59d160ad49c695a801d9a940a2a387d2) with the above code
- `permit_join` will be enabled for 10 minutes (based on code automation)


### Updating the Add-on and `zigbee2mqtt` Library

Currently, `zigbee2mqtt` is adding new features and functionality quite quickly, and is not using versioned releases. This makes it difficult to increment versioning for this add-on, as we simply pull the latest master branch when building the Docker image. Until `zigbee2mqtt` stabilizes, we will likely not use versioned releases. Therefore, in order to update the add-on and to update `zigbee2mqtt`, you must uninstall and reinstall the add-on via the Hassio UI.

Note: If you have reinstalled the add-on and believe that the latest version has not been installed, try removing the repository before reinstalling.

### Issues

If you find any issues with the addon, please check first the [issue tracker](https://github.com/danielwelch/hassio-zigbee2mqtt/issues).

Feel free to create a PR for fixes and enhancements. 

## Credits
- [danielwelch](https://github.com/danielwelch)
- [ciotlosm](https://github.com/ciotlosm)
- [Koenkk](https://github.com/Koenkk) for [zigbee2mqtt](https://github.com/Koenkk/zigbee2mqtt)