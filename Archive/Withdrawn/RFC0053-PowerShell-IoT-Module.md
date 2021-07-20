---
RFC: '0053'
Author: Andrew Menagarishvili
Status: Withdrawn
SupercededBy: N/A
Version: 1.0
Area: External PowerShell Module / PowerShell IoT
Comments Due: Apr 5th, 2018
Plan to implement: Yes
---

# PowerShell IoT Module

This RFC outlines creation of a PowerShell module containing interfaces (cmdlets) for working with
peripheral devices in IoT applications of single board ARM computers such as Raspberry Pi.
Cmdlets discussed here are PowerShell wrappers over standard elecrical interfaces (I2C, SPI, GPIO)
that provide PowerShell-based automation access to peripheral hardware management.

## Motivation

The PowerShell team is investigating IoT scenarios for PowerShell.
Single board computers (e.g. Raspberry Pi) gained popularity in recent years in fields like home automation, education and robotics.
SBCs have a set of connectors for attaching peripheral devices like sensors, screens, motors, etc...
PowerShell, being a powerfull cross-platform automation engine, is a perfect tool for managing such hardware.
Proposed PowerShell module will allow users to implement IoT solutions without knowledge of low-level electrical details.

## Specification

For the initial release of the module we are targeting to cover these interfaces in the following priority:
* GPIO - General-purpose input/output line
* I2C - Inter-Integrated Circuit bus
* SPI - Serial Peripheral Interface bus

### GPIO

Cmlets get and set voltage level (aka logic true/false signal) of general-purpose input/output lines.
Used to turn on/off components like LEDs, power relays, reading from simple sensors.

```powershell
Get-GpioPin [[-Id] <int[]>] [[-PullMode] {Off | PullDown | PullUp}] [-Raw]
Set-GpioPin [-Id] <int[]> [-Value] {Low | High} [-PassThru]
```
Notes:
* `Get-GpioPin` without parameters returns all available GPIO lines with their properties.
* `-PullMode` parameter controlles built-in pull-up/pull-down resistor on the line that determines line state when nothing is connected to it.
* `-Raw` parameter makes cmdlet return just the signal value as opposed to {PinId,Value,PinInfo} object.

### I2C
Proposed cmlets provide an interface for working with components that support I2C bus.
I2C bus supports up to 127 devices on the same bus, which is 4 wires: power, ground, clock line and data line.
A single operation (write or read) for just one device can be active on the bus at any given time.
Proposed I2C cmdlets read and set values of logical registers on I2C devices.

```powershell
# Prepare a device object descriptor that will be later used in get/set operations
Get-I2CDevice -Id <int> [-FriendlyName <string>]

# Read a number of bytes from a device starding from specified register address
Get-I2CRegister -Device <I2CDevice> -Register <uint16> [-ByteCount <byte>] [-Raw]

# Write a number of bytes to a device as values of specified register
Set-I2CRegister -Device <I2CDevice> -Register <uint16> -Data <byte[]> [-PassThru]
```

Notes:
* `Get-I2CDevice -Id` parameter specifies device address on I2C bus (can be found by tools like `sudo i2cdetect -y 1`).
* `-Raw` parameter makes cmdlet return just the register value as opposed to {Device,Register,Data} object.

### SPI
Proposed cmlets provide an interface for working with components that support SPI bus.
SPI bus supports high clock speeds (data transfer rate) and allows parallel read/write to/from the component.
Only one device can be active on the bus at any given time (doing parallel read and write).
Proposed SPI cmdlet is operating with byte streams.

```powershell
# Send a byte stream to a device and return a responce byte stream
Send-SPIData -Channel <uint> -Data <byte[]> [-Frequency <uint>] [-Raw]
```

Notes:
* Default frequency is 8 Mhz which is typical in modern hardware.
* `-Raw` parameter makes cmdlet return just the responce byte stream as opposed to {Channel,Data,Responce,Frequency} object.

### Examples
Several example component-specific modules are necessary to show the usage of proposed cmdlets.
Following devices have been selected based on popularity in the community:
* BME280 environmental sensor (pressure, humidity and temperature)
* SSD1306 OLED display

## Alternate Proposals and Considerations

### Dependecies

Making an interface for I2C or SPI bus with all the details is a difficult task, so we are reusing existing community-created libraries that provide simplified APIs.
After comparing several libraries https://unosquare.github.io/raspberryio was selected as it is a nice .NET Core compatible wrapper over WiringPi - very popular in RaspberryPi community library.
Proposed PowerShell module is expected to be redistributed with raspberryio and WiringPi binaries.

### Supported OSes
With the initial release of the module we are targeting Raspbian Stretch and Windows 10 IoT Core with Raspbian being a priority as it is more popular in the community.
