CLI relies heavily on the configuration file to do most actions.

For example the configuration file will provide information such as:

* Flashing configuration
* Whether the device should be router or not
* What peripherals are available on the board
* How frequently to poll the sensors
* What sensors are attached

By default, configuration file is called `zigbee.yml` and CLI will try to find it in the working directory. 
It is possible to point the CLI to this configuration file by providing a flag. For this see [Using the CLI](./index.md)

## Example configuration
!!! note
    The complete and latest file is located in the repository, in [`zigbee.yml`](https://github.com/ffenix113/zigbee_home/blob/develop/zigbee.yml). It may contain newer/changed configuration options.

```yaml
# Format for this file is not stable, and can change at any time.

general:
  # Defines how much time a loop will sleep after each iteration.
  # Note: This does not necesseraly mean that each 2 minutes
  # sensor values will be updated. Currently all sensors are polled
  # synchronously, which means that if there are 3 sensors defined
  # and each needs 5 seconds to obtain the result then it will take
  # (3 * 5)s + runevery, which in this case will be 1m15s.
  runevery: 1m
  # Board name, as defined in Zephyr.
  board: nrf52840dongle_nrf52840

  # Defines specific major & minor version that should be used.
  # Later patch version can be automatically selected by the program, 
  # if specific version is not found. 
  # I.e. if requested v2.7.0, but only v2.7.2 is found - v2.7.2 will be used instead.
  # Higher then minimum supported version can be specified (but may be incompatible),
  # but lower then minimum version will result in an error.
  # This is optional configuration.
  ncs_version: v2.7.0
  
  # This paths are necessary to build the generated firmware.
  # If nRF Connect setup was done with VS Code extension -
  # most probably this can be left empty, but if version changes -
  # new path would need to be set here.
  # Paths specified here can start with `~/`, and contain
  # environmental variables, which will be resolved.
  # This values have sane defaults and point to ~/ncs/...
  # Env variables NCS_TOOLCHAIN_BASE & ZEPHYR_BASE have
  # priority over this fields.
  #ncs_toolchain_base: ~/toolchains/7795df4459/
  #zephyr_base: ~/ncs/zephyr

  # zigbee_channels will provide optional list of channels to use.
  # Note: if not defined device will use all available channels,
  # so it is not needed to define this if not specifically necessary.
  zigbee_channels: [11,13,15,16,17]

  # Flasher tells which flashing method to use.
  # Currently `nrfutil`, `mcuboot` and `west`
  # are defined(but not equally tested). Nrfutil works though.
  flasher: nrfutil
  # Flasheroptions are flasher-specific options.
  flasheroptions:
    port: /dev/ttyACM1

# This section is for defining peripherals
# on the board. I.e. uart, spi, i2c, etc.
# NOTE: Only changes should be defined here.
# See https://github.com/zephyrproject-rtos/zephyr/tree/main/boards/<vendor>/<board_name>/<board_name>.dts
# for existing definitions for the board.
# For example nRF52840 Dongle would have board devicetree at
# https://github.com/zephyrproject-rtos/zephyr/blob/26603cefaf41298c417f2eee834ed40d9ab35d3a/boards/nordic/nrf52840dongle/nrf52840dongle_nrf52840.dts
board:
  # Specifically define bootloader for the board.
  # This field is optional, and in most cases this might not be needed.
  # But it is possible that board can be flashed
  # with some non-default bootloader, or that the generated firmware is not
  # placed correctly in memory. This setting can help in above situations.
  # Also this option can be used to try and support boards that
  # are not labeled as officialy supported.
  # 
  # Note: if this field is present - this will force bootloader
  # to be selected one, even if the value is empty string.
  #bootloader: nrf52_legacy

  # This will make "button0" factory reset button. 
  # Pressing this button for at least 5 seconds 
  # would remove current network configuration,
  # allowing it to be connected to another network.
  factory_reset_button: button0

  # This option will add UART loging functionality.
  # User can choose which console to use(usb or uart).
  # UART has to be defined either in default device tree,
  # or in `board.uart` section.
  # Quite limited for now, but can be easily extended
  debug:
    # Enable logging and possibly LEDs
    # for debugging.
    enabled: false
    # Console is where logs will be spit out.
    # Choices are:
    #  * usb - no need to define any other configuration for `usb`.
    #  * uart1, uart2, ... - this either should be already defined for used board,
    #                        or be defined in `uart` section.
    console: uart0
    # LEDs will use LED to show some board state(i.e. power, connection, etc).
    # Leds used here MUST be always defined in `leds`, even if they already
    # present in board definition.
    leds:
      # If false - leds will not be enabled to signal state.
      enabled: true
      # Define led which will be used to show that the board is powered.
      power: led_green
  # Change device type from sleepy end device to router.
  # This allows for device to always be available from coordinator
  # and other devices.
  # By default device will be configured as sleepy end device.
  # Note: Enabling router will increase the size of the firmware.
  is_router: false
  # Buttons is optional, to provide information about available buttons on the board.
  # Available button is not necessary a physical button, but can also be a pin.
  buttons:
    # If only ID is provided it means that this button is already present in board definition,
    # and it will be just referenced from it.
    - id: button0
    # Button can also have a pin, which will create new "button" on specified pin.
    # To make it active-low - set "inverted" configuration option to "true".
    - id: button1
      pin:
        port: 0
        pin: 18
        inverted: true
  
  # I2C is optional, only to provide different pins for i2c instance(s)
  i2c:
      # ID of instance is the same as defined in the SoC definition.
      # Generally they are in form of `i2c[0-9]`.
      # Number of i2c instances is also limited, and this allows only to
      # re-define pins for specified i2c instance.
    - id: i2c0
      sda: 0.29
      scl:
        port: 0
        pin: 31
  uart:
    - id: uart0
      # Pins are optional, if this is existing uart interface. 
      tx: 1.03
      rx: 1.10
  leds:
    - id: led_green
      # The pin definition is optional, if led is already present in board definition.
      pin:
        port: 0
        pin: 6
        inverted: true

# Sensors define a list of devices that 
# can provide sensor values or be controlled
sensors:
  # All sensors have type, and most will, 
  # also have sensor-specific configuration.
  - type: bme680
    i2c:
      id: i2c0
      # Some devices might have changable I2C address, 
      # which can be defined here.
      # Note: this does not change the device address,
      # only tells which address to use.
      addr: '0x76'
  - type: scd4x
    i2c:
      id: i2c0
  # - type: device_temperature
  # on_off is a sensor that will respond to on/off state of the pin.
  # For now verifyied to be controlled by the client only,
  # so not by actually changing the state of the pin manually.
  # - type: on_off
  #   pin:
  #     # This is Green LED(LD1) on nrf52840 Dongle
  #     port: 0
  #     pin: 6
  #     inverted: true
  # Add a dht sensor from aosong manufacturer
  # If DHT22 or AM2302 is used, variant must be set to "dht22". DHT11 does not need variant.
  # - type: dht
  #   variant: "dht22"
  #   pin:
  #     port: 1
  #     pin: 13
  #
  # power_config uses ADC to measure positive voltage
  # on pin 0.04 and report it as a battery voltage.
  # ZCL 4.9 will be implemented to do general 
  # electricity measurements through ADC.
  # Capacitive mositure sensors would be added
  # with ZCL 4.7.
  # - type: power_config
  #   adc_pin: 0.04
  #   battery_rated_voltage: 3000
  #   battery_voltage_min_threshold: 2500
  #
  # This example is crude example of using soil_moisture_adc
  # sensor to get water moisture percentage.
  #
  # Note: min & max mv values may differ for your sensor,
  # so please test it and adjust them accordingly!
  # - type: soil_moisture_adc
  #   adc_pin: 
  #     pin: 0.04
  #     oversampling: 4
  #   min_moisture_mv: 730
  #   max_moisture_mv: 430
```

## Sensor options
Each sensor can have unique configuration options. To define such options they must be provided in their configuration in `sensors` section of the configuration file.

If sensors do not provide required configuration - the generated source code might not compile with some errors, or sensor might not report values correctly.

Some sensors would not require any configuration:
```yaml
sensors:
  - type: device_temperature
```
, while others might need some:
```yaml
sensors:
  - type: bme680
    i2c:
      id: i2c0
      addr: '0x76'
```
Here we define that the board will have a Bosch BME680 sensor attached to I2C instance with id `i2c0`, and will have address `0x76`.

Other options may also be specified in the future, for example:
```yaml
sensors:
  - type: bme680
    temperature:
      oversampling: x4 # Notice the `oversampling` option
    i2c:
      id: i2c0
      addr: '0x76'
```