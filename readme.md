# OLED Control for PCP 7.x, 8.X & 9.X & Configuration for Audiophonics EvoSabre DAC

## Background

This project began to allow me to control the second OLED screen on the excellent Audiophonics EvoSabre DAC when using PCP, since the supplied option for LMS was to use a very old version of DietPi.  However, overtime I realised that the project was really 2 things.  Firstly, it was info on how to configure PCP to work with this DAC, but much more it was just a general program to display LMS information on an OLED display.  Since it uses the luma.core and luma.oled modules, it should support any device supported by luma.oled, which you can see at  https://luma-oled.readthedocs.io/en/latest/.  You can specify SPI or I2C interfaces by editing the configuration for your OLED in the oled4pcp.cfg.

As such, there's now 2 sections to this document.  The first section involves the installation and configuration of the OLED scripts for PCP, and the second relates to configuration of PCP for the Audiophonics EvoSabre DAC, setup Infrared etc..

Both of these processes assume that you're starting with a fresh install of PCP 7 or 8, which is connected to your network via ethernet or WiFi and that you have the ability to SSH onto your PCP.  You can use either 32bit or 64bit versions of PCP.

## Functionality

The purpose of the script is to display information from the LMS on the OLED.

- Alternate between Title/Track-Info and Artist/Album when playing, with automatic scrolling when required
- Auto discover LMS IP and player MAC address
- Use combination of subscription for state changes (power, play etc.) and jsonrpc polling when playing, with separate backup polling in case subscription doesn't update
- Font sizes, text locations etc. are configurable in external .cfg file
- Potential support for any device supported by luma.oled
- Auto switching between day and night contrast, with contrast levels defined in external .cfg
- Separate screen saver contrast setting, with screen saver contrast and screen saver delay in external .cfg
- Display clock when player is stopped or off
- With Audiophonics EvoSabre and RaspDAC Mini, correctly configured Infrared, using native Squeezelite IR control.

 

## Installing OLED Control for PCP

The OLED control for PCP requires various components, including some standard PCP extensions, such as Python 3, a dedicated extension for OLED, some python scripts and fonts.  Before installing these extensions, you're likely to need to expand the file system on your SD card.  This can be done through the PCP interface under Main Page, Additional Functions, Resize FS.  Once you've expanded the file system, you can install all of these by downloading and executing a setup script.

SSH onto your PCP then

```bash
wget https://raw.githubusercontent.com/peteS-UK/EvoSabre-DAC-PCP/main/SetupOLED4.sh
chmod +x ./SetupOLED4.sh
./SetupOLED4.sh
```

This will execute a script to download and install the OLED control script and components and add the startup command for the script to the USER_COMMAND_1 entry in Tweaks in PCP.  The setup script will detect your architecture and install the appropriate components, so you can use this on both 32 bit and 64 bit versions of PCP.  All installation files will be placed under ~/.oled4pcp directory.

During installation, you're asked for your OLED device.  For example, for EvoSabre, this is SSD1322 and for RaspDAC Mini, it's SSD1306.  You can see the potentially supported device names at https://luma-oled.readthedocs.io/en/latest/.

When running, the program will take its configuration, including device control, font sizes, text locations etc. from a file, oled4pcp.cfg.  On installation, this file has sections for SSD1322, SSD1306 and SSD1309 displays.  If you're using a different display, you'll need to create an appropriate section in the oled4pcp.cfg file, by copying the SSH1322 section, and adjusting accordingly.

At the end of the installation, you'll be instructed to reboot your PCP

```bash
pcp rb
```

After installation, if you selected either SSD1322, SSD1306 or SSD1309, your device should begin to work on reboot.  For other devices, you'll need to modify oled4pcp.cfg, backup and reboot.

If you do add other devices, please contribute them back to repo.

### Command Line Options

The command to start the script for the display is added during installation to the USER_COMMAND_1 line in the Tweaks page on PCP.  By default, this will be 

```
python3 /home/tc/oled4pcp_4.py
```

#### LOGFILE=

If you specify LOGFILE=Y, the logger output from the script will be written to /var/log/oled4pcp.log, rather than to standard out.


### OLED4PCP.CFG

The oled4pcp.cfg file contains a [CONFIG] section, which contains some general settings for the app, and then a section for each display type - e.g. [SSD1322]

#### [CONFIG]

The [CONFIG] section contains a number of settings for the app.  These include

```
[CONFIG]
oled_section=SSD1322
latitude=0
longitude=0
```

##### oled_section=
This setting identifies the section in the .cfg file which actually contains the settings for the display.

##### Latitude and Longitude
If you specify the latitude and longitude, the app will use these to lookup the dusk and dawn for your location and use these to adjust the contrast to the  `contrast_day` and `contrast_night` settings.  If you leave these both equal to zero, the app will try and lookup your location based on your public IP address and use this instead.
 
#### [DISPLAY SECTION]
The oled4pcp.cfg contains entries specific to the OLED device in a section referenced by the oled)section= entry.  For the [SSD1322], these are

```
type=ssd1322
serial_interface=spi
serial_params={"port":0, "device":0, "gpio_DC":27, "gpio_RST":24}
device_params={"rotate":0, "mode":"1"}
```

Take care to make sure the "type" matches your actual display and is listed at https://luma-oled.readthedocs.io/en/latest.  Check the "serial_interface" is correctly set to either spi or i2c and that the "serial_params" and "device_params" are set correctly for your device and connection and don't interfere with any other GPIOs in use.

If you make any changes to the oled4pcp.cfg file, you need to backup and reboot, either through the PCP UI, or through SSH

```bash
pcp bu
pcp rb
```

## Configuring PCP for Audiophonics EvoSabre DAC

Once you've setup the OLED, if you're using the Audiophonics EvoSabre, you need to also setup Squeezelite etc.  

### Setup Squeezelite

Configure the Squeezelite player for the Audiophonics DAC.

1. in the PCP Web UI, select the Squeezelite Settings tab
3. In **Audio output device settings**, set card to **Audiophonics ISabre Q2M**
4. Save and Reboot

After reboot, again in the Squeezelite Settings Tab

1. Click on **Card Control**
2. In **Raspberry Pi Built-in Audio**, untick Enable/disable built-in audio to disable the built in audio.
3. Save and Reboot

After reboot, again in the Squeezelite Settings Tab

1. Type **Digital** in **ALSA volume control** setting to enable Hardware Volume Control
2. Enter a **Name of your player**
3. Type **0:dop** in **Device supports DSD/DoP** to enable DOP support
4. Choose **Save** to save the Squeezelite changes

### Poweroff/Shutdown Overlays

The DAC has built in power monitor to allow smooth shutdown. When you powerdown the DAC using the supplied remote, this script will shutdown the pi before powering off the DAC

1. In the PCP Web UI, select the Tweaks tab
2. In the **Poweroff/Shutdown Overlays** section
3. Set gpio-shutdown to Yes, GPIO-17, Active Low, Pull Up
4. Click gpio-shutdown to save the values.
5. Reboot

### HDMI Power

Unless you intend to use HDMI, you should disable HDMI power.

1. In the PCP Web UI, select the Tweaks tab
2. In PCP System Tweaks section
3. Select Off for HDMI Power
4. Save

## Configure IR Remote Control

The supplied infrared can be used to control squeezelite through configuring Infrared in PCP.  Actions are configured for 

- Volume Up 
- Volume Down
- Next -> Forward
- Prev -> Rewind
- Up -> Next
- Down -> Previous
- Stop
- Power

 To configure the IR Remote Control

1. In PC Web UI
2. Tweaks Tab
3. In IR Remote Control, select **IR Page**
4. select **Install** to Install LIRC for Squeezelite
5. In **IR device Settings** set **IR GPIO in number (IR receiver)** to 4

After rebooting, you can install the device specific control files.  SSH into the PCP

```bash
wget https://raw.githubusercontent.com/peteS-UK/EvoSabre-DAC-PCP/main/SetupLirc.sh
chmod +x ./SetupLirc.sh
./SetupLirc.sh
pcp rb
```

## Credits

I've leant heavily on other people's work to try and build this extension and script.  In particular, I'd like to reference

**Python2 Audiophonics script**.  I started with the original python2 DietPi script, updated it to Python3, and then over time, I've removed pretty much all of the original code, but this was the genesis.

**luma.core and luma.oled and examples**.   The OLED control is based on luma.core and luma.oled, and I've also used the example from https://github.com/rm-hull/luma.examples.  This is copyright (c) 2017-2020 Richard Hull & Contributors.

**PyLMS**.  I reused from the server.py telnet cli wrapper, but had to extend it to include a read function to implement subscription.

Dawn & Dusk timings are loaded from https://sunrise-sunset.org/api

## License 

In line with the reuse of luma.core, the software is available under MIT license

The MIT License (MIT)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
