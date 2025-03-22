# Martin Jerry MJ-SD01 Dimmer Switch ESPHome Configuration

This is a modern, functional ESPHome configuration for the Martin Jerry MJ-SD01 in-wall Smart Dimmer switch. [Link to product page](https://www.martinjerry.com/product-page/3-pack-smart-dimmer-switch-single-pole-model-mj-sd01). [Link to Amazon page](https://www.amazon.ca/dp/B07Y53VZ17).

## Motivations

The original YAML config floating around (likely from [this repository](https://github.com/mjoshd/esphome_martin-jerry-mj-sd01-dimmer/blob/master/martin_jerry_mj_sd01_dimmer.yaml)) technically worked, but had a lot of quirks and issues that I wanted to solve for my own dimmers. Those tweaks bring us here, to my updated YAML configuration.

## Key Features

* The config is normalized for recent (2025+) ESPHome versions, including an easier setup process. A pre-built binary in AP mode is available, ready to be flashed with `tuya-convert` and adopted easily by HomeAssistant (see below).

* The brightness level is properly retained between on/off sessions - if you turned it off at say 100%, it turns back on at 100%, rather than always going to 50%.

* The minimum PWM level is set to 5% to permit instant-off, rather than lingering for ~2s, as detailed in [this forum post](https://community.home-assistant.io/t/esphome-martin-jerry-sd01/622134). This does have some implications for brightness levels and especially the functionality of LED bulbs, so...

* The maximum and minimum brightness levels are configurable in the UI (HomeAssistant Entities and ESPHome WebUI), preventing the need to recompile to change these for e.g. different light types (LED vs incandescent) or environments, and the handling of levels is significantly revamped to work with this functionality. Simply adjust the brightness level manually to the lowest point where your lights reliably turn on/off, and then use this as your minimum level; set a lower maximum if you want "full brightness" to be something other than the max or to increase incandescent bulb longevity. These values save every 15 seconds to local flash. For the brightness levels, Level 1 is the minimum brightness configured, Level 5 is the maximum brightness configured, and the intervening levels are automatically calculated, giving you full physical control with the 5 visible LEDs throughout the range you specify. Note that the brightness slider permits bypassing these levels; they only apply to the buttons (physical and virtual in HomeAssistant).

* Trying to go above/below the max/min levels respectively now does nothing, instead of flashing the LEDs. I found this annoying myself, especially since LED1 (the bottom one) wouldn't blink, so I just removed the functionality.

* The multiple brightness levels are exposed as virtual "buttons" through the UI to permit quick changing there as if you were at the physical switch, instead of using the slider. There are buttons both for Up and Down, as well as for the 5 configured individual levels (min, 2/5, 3/5, 4/5, max). You may use these in your cards/dashboards or automations as desired, or just ignore them and stick with manual level control. This enables the ability to enforce the minimum/maximum levels in the UI and in automations as well.

* When pressed while the light is off, both the up and down buttons (physical and virtual) function as "power on" identical to the main button, rather than changing or jumping to a particular brightness. For my use this is fine, though I'm sure an enterprising hacker can come up with a neat way to select individual brightness levels with these.

* Additional data sensors and control buttons are now provided, including both ESP restart and factory reset buttons, to ease setup and moving of dimmers around if needed.

All in all, these changes make this setup function more like the WeMo dimmers I was used to, and then some, and I hope this is useful to others too.

## Installing

As of my purchase of 3 units [from Amazon](https://www.amazon.ca/dp/B07Y53VZ17) in March 2025, `tuya-convert` seems like the most reliable option; it worked perfectly on the first try, though future revisions might not work any longer.

0. Install `tuya-convert` following their instructions.

1. Download the `mj-sd01.bin` file from the Releases section, or compile the configuration yourself.

2. Put the `mj-sd01.bin` binary into your `tuya-convert/files/` directory.

3. Power on the switch and put it in "fast blink" mode by holding down the power button for ~5 seconds.

4. Run `./start_flash.sh` and let it perform the initial jailbreak.

5. When prompted "Ready to flash third party firmware!", select the `mj-sd01.bin` file.

Once flashed, the switch will boot into AP mode, ready to be configured via another device to connect to your WiFi and be adopted by the ESPHome Builder in Home Assistant. It leverages a standard "pull the remote config" to permit configuring only the minimum required in the ESPHome Builder, while all functionality is managed by this repository.

## Reporting Bugs or Feature Requests

If you find a bug, please report it by opening an issue. For a new feature, please open an issue.

I'm open to feature requests, as long as they don't compromise the core functionality detailed above.

## Entities

<p><img alt="Controls entities" src="https://raw.githubusercontent.com/joshuaboniface/martinjerrymjsd01-esphome/refs/heads/master/images/device.png"/><br/>
<i>Main device information.</i>
</p>
<p><img alt="Controls entities" src="https://raw.githubusercontent.com/joshuaboniface/martinjerrymjsd01-esphome/refs/heads/master/images/controls.png"/><br/>
<i>Controls entities, including buttons for all 5 brightness levels (1 is lowest), up/down brightness, the main switch control (renamed for my use, defaults to "Light"), the status LED (opposite of main switch), and the min and max brightness level setters (0-50 and 50-100 respectively).</i>
</p>
<p><img alt="Sensors entities" src="https://raw.githubusercontent.com/joshuaboniface/martinjerrymjsd01-esphome/refs/heads/master/images/sensors.png"/><br/>
<i>Sensors representing the physical buttons, for use in automations.</i>
</p>
<p><img alt="Diagnostic entities" src="https://raw.githubusercontent.com/joshuaboniface/martinjerrymjsd01-esphome/refs/heads/master/images/diagnostic.png"/><br/>
<i>Diagnostic entities, including reboot and factory reset if required.</i>
</p>

## Neat Example

This is an automation I have which leverages the brightness level button and the ability to store the current brightness in another component. Perhaps not special, but shows why exposing the level buttons can be useful.

### `input_number` for last brightness level

```
{
  "version": 1,
  "minor_version": 1,
  "key": "input_number",
  "data": {
    "items": [
      {
        "id": "basement_theatre_overhead_lights_last_brightness",
        "min": 0.0,
        "max": 255.0,
        "name": "Basement Theatre Overhead Lights Last Brightness",
        "icon": "mdi:brightness-percent",
        "mode": "box",
        "step": 1.0
      }
    ]
  }
}
```

### "Down" automation

```
alias: "Action: Lower Basement Theatre lights when Jellyfin playback starts on FireTV"
description: "Action: Lower Basement Theatre lights when Jellyfin playback starts on FireTV"
triggers:
  - device_id: 00000000000000000000000000000000
    domain: media_player
    entity_id: 00000000000000000000000000000000
    type: playing
    trigger: device
conditions:
  - condition: state
    state: "on"
    entity_id: light.martin_jerry_mj_sd01_XXXXXX_light
actions:
  - target:
      entity_id: input_number.basement_theatre_overhead_lights_last_brightness
    data:
      value: >-
        {{ state_attr('light.martin_jerry_mj_sd01_XXXXXX_light', 'brightness') |
        default(0) }}
    action: input_number.set_value
  - action: button.press
    target:
      entity_id: button.martin_jerry_mj_sd01_XXXXXX_brightness_1
    data: {}
mode: single
```

### "Up" automation

```
alias: "Action: Raise Basement Theatre lights when playback stops on FireTV"
description: "Action: Raise Basement Theatre lights when playback stops on FireTV"
triggers:
  - device_id: 00000000000000000000000000000000
    domain: media_player
    entity_id: 00000000000000000000000000000000
    type: idle
    trigger: device
  - device_id: 00000000000000000000000000000000
    domain: media_player
    entity_id: 00000000000000000000000000000000
    type: turned_off
    trigger: device
conditions:
  - condition: state
    state: "on"
    entity_id: light.martin_jerry_mj_sd01_XXXXXX_light
actions:
  - data:
      brightness: >-
        {{
        states('input_number.basement_theatre_overhead_lights_last_brightness')
        | int }}
      transition: 2
    action: light.turn_on
    target:
      entity_id: light.martin_jerry_mj_sd01_XXXXXX_light
mode: single
```
