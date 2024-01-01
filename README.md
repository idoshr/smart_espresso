# Smart Espresso
This project is a work in progress. 
The goal is to create a smart espresso machine using a Raspberry Pi 4 and a few sensors to monitor the pressure of the water in the boiler and in the head of the machine.
The pressure sensors are connected to a MCP3008 ADC and the Raspberry Pi is connected to a Home Assistant instance to monitor the pressure.




## Installation

```bash
pip install -r requirements.txt
```

## Usage

### Params
| Key            | Description                                   |
|----------------|-----------------------------------------------|
| analog_devices | List of analog devices (PressureAnalogSensor) | 
| client_ha      | Home Assistant Client                         |
| display        | luma oled device                              |


## Example
```python
import os

from luma.core.interface.serial import i2c
from luma.oled.device import sh1106
from homeassistant_api import Client

from smart_espresso.analog_sensor.perssure_analog_sensor import PressureAnalogSensor
from smart_espresso.smart_espresso import SmartEspresso
from smart_espresso.utils import strtobool


HA_ENABLE = strtobool(os.environ.get("HA_ENABLE") or False)
client_ha = None
if HA_ENABLE:
    HA_URL = os.environ.get("HA_URL")   # e.g. http://192.168.0.123:8123
    HA_TOKEN = os.environ.get("HA_TOKEN")
    HA_VERIFY_SSL = strtobool(os.environ.get("HA_VERIFY_SSL") or True)
    if not HA_URL or not HA_TOKEN:
        raise ValueError("HA_URL and HA_TOKEN are required")

    # Initialize HomeAssistant Client before you use it.
    print('Connecting to Home Assistant')
    client_ha = Client(f"{HA_URL}/api",
                       HA_TOKEN,
                       verify_ssl=HA_VERIFY_SSL)


# NB ssd1306 devices are monochromatic; a pixel is enabled with
#    white and disabled with black.
# NB the ssd1306 class has no way of knowing the display resolution/size.

display = sh1106(i2c(port=1, address=0x3c), width=128, height=64, rotate=0)

# set the contrast to minimum.
display.contrast(1)

# show some info.
print(f'device size {display.size}')
print(f'device mode {display.mode}')


if __name__ == '__main__':
    analog_devices = [
        PressureAnalogSensor(pin=0, name='Head'),         # Head Pressure
        PressureAnalogSensor(pin=3, name='Boiler')        # Boiler Pressure
    ]
    se = SmartEspresso(analog_devices=analog_devices, 
                       client_ha=client_ha, 
                       display=display)
    se.run()

```


### Prerequisite list

| Items | Description                                                                                                                      | Link                              | Notes                                                                           |
|-------|----------------------------------------------------------------------------------------------------------------------------------|-----------------------------------|---------------------------------------------------------------------------------|
| 1     | Raspberry Pi 4 Model B 2GB RAM                                                                                                   |                                   |                                                                                 |
| 1     | MCP3008-I/P MCP3008 DIP-16                                                                                                       | https://a.aliexpress.com/_olcJc4g |                                                                                 |
| 2     | Pneumatic Plumbing Brass Pipe Fitting Male/Female Thread 1/8" 1/4" 3/8" 1/2" BSP Tee Type Copper Fittings Water Oil Gas Adapter  | https://a.aliexpress.com/_okOIGjW | Choose the F-F-M 1/8"                                                           |
| 1     | Water Oil Fuel Gas Air Pressure Sensor Transducer (PT1/4 G1/2 G1/8) 5-12V 0.5-4.5V 0-300Bar Gauge Optional Consumer Electronics  | https://a.aliexpress.com/_omToNFi | Choose the 0-0.5Mpa G1-8 and write to the seller that you need the 3.3v version |   
| 1     | Water Oil Fuel Gas Air Pressure Sensor Transducer (PT1/4 G1/2 G1/8) 5-12V 0.5-4.5V 0-300Bar Gauge Optional Consumer Electronics  | https://a.aliexpress.com/_omToNFi | Choose the 0-2Mpa G1-8 and write to the seller that you need the 3.3v version   |          
| 1     | 60 Values 1/4W 1％ Metal Film Resistors Kit 300Pcs 1 ohm - 4.7M ohm 1/4 Watt W High Precision MF Resistance Set Assortment        | https://a.aliexpress.com/_oBZ1yNa |                                                                                 |
| 1     | 530pcs/Set Polyolefin Shrinking Assorted Heat Shrink Tube Wire Cable Insulated Sleeving Tubing Set 2:1 Waterproof Pipe Sleeve    | https://a.aliexpress.com/_okRXZ5m |                                                                                 |
| 1     | DIY Prototype Raspberry Pi Expansion Board PCB Shield  Compatible for Raspberry Pi 4B/3B/3B  Board For DIY Starter Projects      | https://a.aliexpress.com/_opgseK8 |                                                                                 |
| 1     | 1.3" OLED Display Module White/Blue Color Drive Chip SH1106 128X64 1.3 inch OLED LCD LED IIC I2C Communicate For ArduinoProjects | https://a.aliexpress.com/_oEzEfpA | Nice but too small                                                              |


### Tools

* Soldering iron
* Soldering wire
* Multimeter
* Ratchet wrench
* Screwdriver
* Teflon tape

### Wiring diagram


![display](docs/img/display.png)


![analog](docs/img/analog.png)


### References / credits / inspiration
* https://wiki.dfrobot.com/Gravity__Water_Pressure_Sensor_SKU__SEN0257
* https://m.youtube.com/watch?v=JlgXGrb4lVE
* https://sites.google.com/view/coffee4randy/home?authuser=0
* https://hackaday.com/2023/10/07/smart-coffee-replaces-espresso-machine-controller-with-arduino-sensors/#more-628135