[![](https://img.shields.io/github/license/arijitkhaldar/AIOC)](https://github.com/arijitkhaldar/AIOC/blob/usb-a/LICENSE.md)

# AIOC-BMT

This is the Ham Radio _All-in-one-Cable_ with a USB Type-A instead of USB-C which is inspired from [AIOC](https://github.com/skuep/AIOC). **It is currently not tested, and is still a proof of concept only!** Please read this README carefully before ordering anything.

![AIOC-BMT front face](doc/images/AIOC-BMT_Front.png?raw=true "AIOC-BMT front face")

## What does it do?

The AIOC-BMT is a small adapter with a USB-A connector that enumerates itself as a sound-card (e.g. for APRS purposes), a virtual tty ("COM Port") for programming and asserting the PTT (Push-To-Talk) as well as a CM108 compatible HID endpoint for CM108-style PTT (new in firmware version 1.2.0).

![AIOC-BMT front USB face](doc/images/AIOC-BMT_Front_USB_View.png?raw=true "AIOC-BMT front USB face")
![AIOC-BMT front green face](doc/images/AIOC-BMT-front.png?raw=true "AIOC-BMT front green face")
![AIOC-BMT back face](doc/images/AIOC-BMT_Back.png?raw=true "AIOC-BMT back face")
![AIOC-BMT back tilted face](doc/images/AIOC-BMT_Back_Tilted.png?raw=true "AIOC-BMT back tilted face")
![AIOC-BMT front tilted face](doc/images/AIOC-BMT_Front_Tilted.png?raw=true "AIOC-BMT front tilted face")

## Features

- Cheap & Hackable Digital mode USB interface (similar to digirig, mobilinkd, etc...)
- Programming Cable Function via virtual Serial Port
- Compact form-factor (DIY overmolded enclosure is currently TBD)
- Based on easy-to-hack **STM32F302** with internal ADC/DAC (Programmable via USB bootloader using [DFU](#how-to-program))
- Can support Dual-PTT HTs
- Supports all popular OSes (Linux, Windows and MacOS with limitations[\*](https://github.com/skuep/AIOC/issues/13))

## Compatibility

### Software

- [Direwolf](#notes-on-direwolf) as AX.25 modem/APRS en+decoder/...
- [AllStarLink](#notes-on-allstarlink-asl3) as ASL Node
- [APRSdroid](#notes-on-aprsdroid) as APRS en+decoder
- [CHIRP](#notes-on-chirp) for programming
- [VaraFM](#notes-on-varafm)
- ... and more

### Tested Radios (so far)

TODO

## How To Fab

- Go to JLCPCB.com and upload the AIOC-BMT.zip package (under `aioc-bmt\production`)
  - Select PCB Thickness 1.6mm (you can select any thickness of your choice, as long as the USB connector can be mounted)
  - You may want to select LeadFree HASL
  - Select Silkscreen/Soldermask color to your liking
  - Choose Mark on PCB: Order Number(Specify Position)
- Check "PCB Assembly"
  - PCBA Type "Economic"
  - Assembly Side "Top Side"
  - Tooling Holes "Added by Customer"
  - Press Confirm
  - Click "Add BOM File" and upload `bom.csv`
  - Click "Add CPL File" and upload `positions.csv`
  - Press Next
  - Look Through components, see if something is missing or problematic and press Next
  - Check everything looks roughly good (rotations are already baked-in and should be correct). Save to Cart

This gives you 5 (or more) SMD assembled AIOC-BMT. The only thing left to do is soldering on the audio cables.
You can choose to fabricate 5 PCBs, but populate 2, which will reduce the cost, but you have to solder the rest of the boards yourself in that case.

**Note** that the following message from JLCPCB is okay and can be ignored.

```
The below parts won't be assembled due to data missing.
J1,J2,J3,J4 designators don't exist in the BOM file.
```

**Note** for people doing their own PCB production: I suggest using the LCSC part numbers in the BOM file as a guide on what to buy (especially regarding the MCU).

## How To Build

For building the firmware, clone the repository and initialize the submodules. Create an empty workspace with the STM32CubeIDE and import the project.

- `git clone <repositry url>`
- `git submodule update --init`
- Start STM32CubeIDE and create a new workspace under `<project-root>/stm32`
- Choose File->Import and import the `aioc-fw` project in the same folder without copying
- Select Project->Build All and the project should build. Use the Release build unless you specifically want to debug an issue

## How To Program

### Initial programming

The following steps are required for initial programming of the AIOC-BMT:

- Short outermost pins on the programming header. This will set the device into bootloader mode in the next step.
  ![Shorting pins for bootloader mode](doc/images/k1-aioc-dfu.jpg?raw=true "Shorting pins for bootloader mode")
- Connect USB-A cable to the AIOC-BMT PCB
- Use a tool like `dfu-util` to program the firmware binary from the GitHub Releases page like this:
  ```
  dfu-util -a 0 -s 0x08000000:leave -D aioc-fw-x-y-z.bin
  ```
  **Note** that a `libusb` driver is required for this. On Windows there are additional steps required as shown [here](https://yeswolf.github.io/dfu) (_DFuSe Utility and dfu-util_). On other operating systems (e.g. Linux, MacOS), this just works ™ (provided libusb is installed on your system).
  On Linux (and MacOS), your user either needs to have the permission to use libusb (`plugdev` group) or you might need to use `sudo`.
- Remove short from first step, unplug and replug the device, it should now enumerate as the AIOC-BMT device

### Firmware updating

Once the AIOC-BMT has firmware loaded onto it, it can be re-programmed without the above BOOT sequence by following these steps.

**Note** This requires firmware version >= 1.2.0. For older firmwares, the initial programming sequence above is required for updating the firmware.

- Run `dfu-util` like this
  ```
  dfu-util -d 1209:7388 -a 0 -s 0x08000000:leave -D aioc-fw-x-y-z.bin
  ```

This will reboot the AIOC-BMT into the bootloader automatically and perform the programming.
After that, it automatically reboots the AIOC-BMT into the newly programmed firmware.

**Note** Should you find yourself with a bricked AIOC-BMT, use the initial programming sequence above

## How To Use

The serial interface of the AIOC-BMT enumerates as a regular COM (Windows) or ttyACM port (Linux) and can be used as such for programming the radio as well as PTT (Asserted on `DTR=1` and `RTS=0`).

**Note** before firmware version 1.2.0, PTT was asserted by `DTR=1` (ignoring RTS) which caused problems with certain radios when using the serial port for programming the radio e.g. using CHIRP.

The soundcard interface of the AIOC-BMT gives access to the audio data channels. It has one mono microphone channel and one mono speaker channel and currently supports the following baudrates:

- 48000 Hz (preferred)
- 32000 Hz
- 24000 Hz
- 22050 Hz (specifically for APRSdroid, has approx. 90 ppm of frequency error)
- 16000 Hz
- 12000 Hz
- 11025 Hz (has approx. 90 ppm of frequency error)
- 8000 Hz

Since firmware version 1.2.0, a CM108 style PTT interface is available for public testing. This interface works in parallel to the COM-port PTT.
Direwolf on Linux is confirmed working, please report any issues. Note that currently, Direwolf reports some warnings when using the CM108 PTT interface on the AIOC.
While they are annoying, they are safe to ignore and require changes in the upstream direwolf sourcecode. See https://github.com/wb2osz/direwolf/issues/448 for more details.

## Notes on Direwolf

- Follow the regular setup guide with [Direwolf](https://github.com/wb2osz/direwolf) to determine the correct audio device to use.
  For the serial and CM108 PTT interfaces on Linux, you need to set correct permissions on the ttyACM/hidraw devices. Consult Direwolf manual.
- Configure the device as follows
  ```
  [...]
  ADEVICE plughw:<x>,0  # <- Linux
  ADEVICE x 0           # <- Windows
  ARATE 48000
  [...]
  PTT CM108             # <- Use the new CM108 compatible style PTT interface
  PTT <port> DTR -RTS   # <- Alternatively use an old school serial device for PTT
  [...]
  ```

## Notes on AllStarLink (ASL3)

Once your cable is emulating a CM108, it becomes quite simple to plug into your HT and setup a simple simplex AllStarLink node that talks to your favorite repeater or node.

In `asl-menu`, set:

- Device type: `usbradio` (menu sections: 2 > A1 > N4 > I2)
- Duplex type: `1` (menu sections: 2 > A1 > N5)

Edit `usbradio.conf` (menu sections: 6 > H) (for a Puxing PX-888 cheap Chineese HT)

- `rxboost = 0`
- `rxsqhyst = 500`
- `carrierfrom = usbinvert`
- `ctcssfrom = no`
- `rxdemod = speaker`
- `txprelim = no`
- `invertptt = no`

## Notes on APRSdroid

[APRSdroid](https://aprsdroid.org/) support has been added by AIOC by implementing support for the fixed 22050 Hz sample rate that APRSdroid requires.
It is important to notice, that the exact sample rate can not be achieved by the hardware, due to the 8 MHz crystal.
The actual sample rate used is 22052 Hz (which represents around 90 ppm of error). From my testing this does not seem to be a problem for APRS at all.

However, since APRSdroid does not have any PTT control, sending data is currently not possible using the AIOC except using the radio VOX function. See https://github.com/ge0rg/aprsdroid/issues/324.
My previous experience is, that the Android kernel brings support for ttyACM devices (which is perfect for the AIOC) so implementing this feature for APRSdroid should theoretically be no problem.

Ideas such as implementing a digital-modes-spefic VOX-emulation to workaround this problem and let the AIOC activate the PTT automatically are currently being considered.
Voice your opinion and ideas in the GitHub issues if this seems interesting to you.

## Notes on CHIRP

CHIRP is a very popuplar open-source programming software that supports a very wide array of HT radios. You can use CHIRP just as you would like with a regular programming cable.

Download:

- Start CHIRP
- Select Radio->Download from Radio
- Select the AIOC COM/ttyACM port and start

Upload:

- Select Radio->Upload to Radio
- That's it

## Notes on VaraFM

Select "DTR only" for PTT Pin, so that the correct RTS/DTR sequence is generated for PTT

# Known issues

TODO

# Future work

TODO
