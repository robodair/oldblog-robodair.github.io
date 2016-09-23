---
layout: single
title: Raspberry Pi "Native" DMX
excerpt: "Part 1 of my Twitter Moodlighting: Setting up my Raspberry Pi to output DMX without a fancy expensive USB box."
categories:
  - Electronics
tags:
  - moodlighting
  - electronics
  - raspberrypi
  - dmx
  - lighting
  - tutorial
  - SpectrumUC
---
## Intro
This is Part 1 of my Twitter controlled moodlighting project.

So I've been toying with this project for a while - I've been wanting to get my University to implement some more interactive things on campus. I also like building things, so I thought I'd give interactive lights a shot.

<img src="/images/rpi-dmx/finished.jpg">

### What I want to do
It's pretty straight forward really - I want to put lights up that everyone can change the color of just by tweeting at a specific hashtag. But first - I needed some lights that I could actually control.

I've had my share of experience with professional LED fixtures through my work for Canberra Theatre and wanted to be able to mount one somewhere and use it. Lighting fixtures use [DMX512](https://en.wikipedia.org/wiki/DMX512) for their control. Even most non-professional lighting gear accepts a DMX signal in some form or another.

Simply put, DMX has 512 channels of information in each packet, and in most lighting use it's one-way. Info is streamed from the controller to the fixture and the fixture doesn't talk back. Fixtures that have multiple functions use several channels for the functions. for example RGB fixtures like [this one](http://www.jaycar.com.au/18-x-1w-rgb-led-par-stage-light/p/ST3600) use three channels for red, green, and blue intensity.

### Why a Pi?
Why do I want to output from the Pi? Well firstly - Pi's are cheap. Much cheaper than professional lighting gear. Secondly, Pi's are an easily hackable computer that I can later make do something else (like hit a web API and change the DMX output based on the result). Which is really pretty perfect for this project, don't you think?

### Bits you'll need
![](/images/rpi-dmx/all-pieces.jpg)

1. Raspberry Pi & SD Card(I used a model B)
2. MAX RS485 chip [like this one](http://www.ebay.com.au/itm/MAX485-Module-RS485-Module-TTL-to-RS-485-Module-Converter-5v-Arduino-AU-STOCK-/181840203968?hash=item2a568550c0:g:Zl4AAOSwjVVV1W~m) (about $3 AUD for 4 if you're prepared to wait for them to come from China)
3. [Logic Level Converter (LLC)](http://www.jaycar.com.au/arduino-compatible-logic-level-converter-module/p/XC4486)
4. DMX or XLR Female connector (Depending on the socket your fixture has)
5. Breadboard & Wires

And, of course - a fixture to control
<img style="width:80%;" src="/images/rpi-dmx/fixture.jpg">


Also download [OLA](https://www.openlighting.org/ola/tutorials/ola-on-raspberry-pi/) and flash it to your SD Card for your Pi.

Now you're ready to put it together!

## Why the chips?

### MAX RS485
<img style="width:50%;" src="/images/rpi-dmx/MAX-RS845.jpg">

I had quite a bit of help here - the pointer to use a MAX RS485 came from Jonathan Andrews [post](http://www.jonshouse.co.uk/rpidmx512.cgi) where he's bit-bashing the gpio using Direct Memory Access.

Why do we need to use a chip at all? Well DMX uses [RS485 Differential Signalling](https://en.wikipedia.org/wiki/RS-485) which is essentially two mirrored waveforms, and the Pi can't output that. But as Jon shows, you can use a MAX RS485 to get the right DMX signal - provided you can get a pin on the Pi to output the base signal at pretty close to the right rate.

### Logic Level Converter
<img style="width:30%;" src="/images/rpi-dmx/logic-converter.jpg">

Again thank you Jon. If you look at his post you'll see a helpful hint that the Raspberry Pi isn't 5v tolerant - that is if you apply 5v to any of the GPIO pins you're likely to "Fry the Pi" (technical term) and you won't have a tasty treat to eat afterwards either. 

Unfortunately the MAX RS485 operates on 5v, and has pull-up resistors on it's IO pins to bring them to 5v. If you directly connect them to the Pi you might get some smoke.

Jon came up with the rather crafty solution of removing those pull-up resistors on the MAX, which works because the 3.3v that the Pi gives is enough for the MAX to count the signal as high. This wasn't really good enough for me, (and I don't have access to soldering equipment at the moment) so I opted for a converter to safely marry the 3.3v and 5v so nothing goes awry.

## Ready the Chips!
Before we connect things to the Pi let's get everything seated properly on the breadboard. The wiring is pretty straight forward really, we only care about a few pins.

### Seating the MAX RS485
<img style="width:50%;" src="/images/rpi-dmx/seating_1.jpg"><img style="width:50%;" src="/images/rpi-dmx/seating_2.jpg">

My breadboard is a bit too thin to properly seat the MAX chip, so I did a bit of MacGyvering and used a piece of cardboard and some tape to make sure the chip was steady (mind you this is just for a prototype)

### Making "Extension" wires
<img style="" src="/images/rpi-dmx/extension.png">

Because the MAX chip overhangs the breadboard and the RPi only has pins, not sockets, I converted a few pin-pin wires into socket-pin wires. This is as simple as prying the black casing off the end (there's a little clip) and using pliers to cut off the pin. 
Slip the case back on and volià! Easy as.

### Wiring
As I mentioned before, wiring is pretty straight forward. I'll just list what goes to what and let you handle the logistics. You'll need one `+3.3v` and one `+5v` rail, and one common ground (`GND`) rail. (You'll provide power/ground to these rails from the Pi).
One quick note - the particular LLC I'm using has two pins for each channel (and there are two channels but I only need one). One pin is pulled high by default (marked `TX`), and the other is pulled low (marked `RX`). I'm using the one pulled high because that's what the MAX does - which also means I don't need to apply 5v to the LLC because the MAX will be doing that for me. I just apply the `+3.3v` and ground.

#### Breadboard Connection
<img style="" src="/images/rpi-dmx/wiring.jpg">

- MAX RS485 `GND` to `GND` rail (black)
- MAX RS485 `VC` to `+5v` rail (red)
- LLC `GND` to `GND` rail (black)
- LLC `LV` (Low Voltage) to `+3.3v` rail (orange)
- MAX `DE` (Data Enable) to `+5v` rail (blue)(Alternatively you could connect this to a GPIO via the second channel of the LLC and only set it high when you were ready to make pretty colors, this MIGHT stop the lights flashing when the Pi boots, but the GPIO's can be pretty noisy when the Pi boots so YMMV)
- MAX `DI` (Data In) to LLC `TX` on `5v` side (purple)

#### Pi Connection
<img style="" src="/images/rpi-dmx/pi_wiring.jpg">

[Here is a really excellent pinout diagram for the RPi B](http://opensourceecology.org/w/images/f/fd/Rasp_v3.pdf)

[And another with pin numbers](http://www.raspberrypi-spy.co.uk/wp-content/uploads/2014/07/Raspberry-Pi-GPIO-Layout-Model-B-Plus-rotated.png)

There are several supply and ground points on the Pi, these are just the ones I used, we only need 4 wires coming off the Pi:

- Pin `1` (`+3.3v` supply) to your `+3.3v` rail (orange)
- Pin `2` (`+5v` supply) to your `+5v` rail (red)
- Pin `25` (`GND`) to your `GND` rail (black)
- Pin `8` (`GPIO 18`, `UART TX`, Serial Transmit) to LLC `TX` on `3.3v` side (purple)

#### DMX Connection
<img style="width:80%;" src="/images/rpi-dmx/xlr_plug.jpg">

I found that I needed to wire:

- MAX Output `A` to Pin `3` of the XLR connector I was using
- MAX Output `B` to Pin `2` of the XLR connector I was using

In theory I should probably have a `GND` there too, but it doesn't matter for this prototype, everything still works fine.

## Stuff On The Pi

### How are we going to output DMX?
So Jon's post is very clever - but it didn't quite strike me as stable or reliable enough for what I wanted, so I went searching, and came across [OLA](www.openlighting.org).

Which is actually perfect for what I want. Actually it's a lot more - it supports full linux systems with a multitude of different protocols to talk to many DMX extension systems through plugins.

And it's one particular plugin we're most interested in. The UART plugin.
What's it do? Well it allows you to output one side of the DMX signal that we want from the RPi's native serial line. The guy who wrote it also has a really nice blog post about [setting it up](http://http://eastertrail.blogspot.com.au/2014/04/command-and-control-ii.html)

If you install OLA from the image it's likely that most of the steps I'll list have already been done for you. But best to check to be sure.

### Setting up OLA
Once you've installed OLA as per [their guide](https://www.openlighting.org/ola/tutorials/ola-on-raspberry-pi/)  there's only a few more things you need to do:

- Login to the Pi (via ssh or using a monitor/keyboard, the default username is `pi` and password `openlighting`)
- Disable Pi UART Terminal (tty over serial)
`sudo raspi-config` then Advanced Options > Serial > No
- Change your hostname to whatever you want (can be done from inside raspi-config)
- Reboot
- Setup any Wifi networks (you can use a USB wifi dongle for the Pi B)
- Check that the `olad` and `pi` users are in the `dialout` group with `id` and `id olad`, you should see a list of the groups they're in. If not, google how to add them.
- install [Avahi](https://en.wikipedia.org/wiki/Avahi_(software)) if you want [mDNS](https://en.wikipedia.org/wiki/Multicast_DNS) for the Pi (otherwise you'll have to access it via IP address)
`sudo apt-get update && sudo apt-get install avahi-daemon`
- Disable all plugins (the following scripts will be in your path if you installed the image so you can just type the commands)
`sudo ola_conf_plugins.sh disable all`
- Enable just the UART plugin
`sudo ola_conf_plugins.sh enable ola-uartdmx`
- Make sure that `ola-uartdmx.conf` points to the right device.
in `/var/lib/ola/conf/ola-uartdmx.conf` change all instances of `/dev/ttyACM0` to whatever device your serial is. (`/dev/ttyAMA0` for the Pi B, this might/will be different on the other Pi's)
- Make sure the line `init_uart_clock=16000000` is in `/boot/config.txt` so the baud rate is raised for the serial.
- Reboot the Pi again to make every service restart

Should you wish to view it, the code for the OLA plugin is available [here](https://github.com/OpenLightingProject/ola/blob/master/plugins/uartdmx/UartDmxPlugin.cpp).

## Fire it up!
Everything should be ready to go, plug in your fixture and the Pi. (Make sure to address your fixture so it starts at channel 1)

### Web Console
You should be able to access a web console at port `9090` on the Pi from any computer connected to the same network. The hostname of my pi is `spectrumuc-2` so I can access the interface at `http://spectrumuc-2.local:9090`.

The final thing you'll need to do is tell OLA about the UART device and assign it a DMX universe. You can do this from the web interface.

1. Click "Add Universe" 

<img style="width:80%;" src="/images/rpi-dmx/ola/add_universe.png">

2. Give the universe an ID (`1`) and name, select the "UART native DMX" checkbox to assign the UART device to that universe, Click "Add Universe" to save. 

<img style="width:80%;" src="/images/rpi-dmx/ola/name_and_select_device.png">

3. Up the top you'll see a tab called DMX Console: 

<img style="width:80%;" src="/images/rpi-dmx/ola/access_console.png">

Access that and there are your DMX sliders! 

<img style="width:80%;" src="/images/rpi-dmx/ola/light_active.png">

Drag them around a bit and - your fixture intensities will change - we've done it!

<img src="/images/rpi-dmx/finished.jpg">

### Hacky Programmatic Control (I can do better)
OLA happens to have a bunch of command line utilities that are quite useful. I'm interested in `ola_streaming_client` which allows you to issue dmx values from the command line. Try it:

`ola_streaming_client -d "255,0,0"`

Your light should go red, because this command sets channel 1 (red) to 255 (max) and turns the others off.

Because Python offers a way for us to issue shell commands, we can write a script that takes some data, converts it, and issues a shell command to change the light. This is effective, if hacky. It's exactly what I'll do in Part 2. OLA does offer an API that can be accessed from Python but that's out-of scope for now.

### Beware
The MAX chip can get pretty hot - so just be a little careful. One of the reasons I didn't have any wires for the RE & RO pins is to minimise unnecessary current.

## Conclusion
Long-term reliability of this remains to be seen, but it's certainly a very cool and cheap project to achieve exactly what I wanted. I'll definitely be making a few more! I hope you've enjoyed the read anf might even build one yourself. Happy coding!