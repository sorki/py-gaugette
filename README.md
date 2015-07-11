py-gaugette
===========

A library for interfacing hardware with the Raspberry Pi and BeagleBone Black.
Supports SSD1306 displays, rotary encoders, capacitance switches, RGB leds and other devices.

Platform Compatibility
======================

This library runs on both the Raspberry Pi and BeagleBone Black.  However
some classes have not yet been ported or tested on both platforms, so refer
to the table below for compatibility information.

The library has been developed using Python v2.7, with a goal of also
maintaining Python 3 compatibility.  However as of Feb 2014 some of the
required libraries are not yet available for Python 3.
 - py-spidev for the Raspberry Pi is not available for Python 3
 - Adafruit_BBIO for the BeagleBone is not available for Python 3

Here's the current compatibility matrix:

| Class         | RPi + Python 2.7 | RPi + Python 3   | BBB + Python 2.7 | BBB + Python 3   |
|:--------------|:----------------:|:----------------:|:----------------:|:----------------:|
| CapSwitch     | yes              | yes              | yes              | no               |
| RgbLed        | yes              | yes              | no               | no               |
| RotaryEncoder | yes              | yes              | yes              | no               |
| SSD1306       | yes              | no               | yes              | no               |
| Switch        | yes              | yes              | yes              | no               |

Prerequisites for the Raspberry Pi
==================================

### WiringPi and WiringPi2-Python

Modules that use GPIO require [wiringpi](https://projects.drogon.net/raspberry-pi/wiringpi/) and [WiringPi2-Python](https://github.com/WiringPi/WiringPi2-Python).

To install wiringpi:
```
git clone git://git.drogon.net/wiringPi
cd wiringPi
sudo ./build
```

To install wiringpi2-python:
```
git clone https://github.com/Gadgetoid/WiringPi2-Python.git
cd WiringPi2-Python/
sudo python setup.py install
cd ..
```

### py-spidev

gaugette.ssd1306 requires [spidev](https://github.com/doceme/py-spidev).

### gdata-python-client

gaugette.oauth2 requires [Google's gdata](http://code.google.com/p/gdata-python-client/).

Prerequisites for the BeagleBone Black
======================================

### Adafruit BBIO

Modules that use GPIO or SPI require [Adafruit_BBIO](https://github.com/adafruit/adafruit-beaglebone-io-python/).

To install:
```
/usr/bin/ntpdate -b -s -u pool.ntp.org
opkg update && opkg install python-pip python-setuptools python-smbus
pip install Adafruit_BBIO
```

SSD1306 OLED Usage
==================

```python
    import gaugette.ssd1306
    RESET_PIN = 15
    DC_PIN    = 16
    led = gaugette.ssd1306.SSD1306(reset_pin=RESET_PIN, dc_pin=DC_PIN)
    led.begin()
    led.clear_display()
    led.draw_text2(0,0,'Hello World',2)
    led.display()
```

SSD1306 Font Usage
==================

```python
    from gaugette.fonts import arial_16
    font = arial_16  # fonts are modules, does not need to be instantiated
    # draw_text3 returns the col position following the printed text.
    x = led.draw_text3(0,0,'Hello World',font)

    # text_width returns the width of the string in pixels, useful for centering:
    text_width = led.text_width('Hello World',font)
    x = (128-text_width)/2
    led.draw_text3(x,0,'Hello World',font)
```

The fonts include the printable ASCII characters ('!' through '~') and because of the usefulness of the degree symbol '&deg;', it has been added as a non-standard character 127 (0x7F hex and 177 octal).  Use the degree symbol in a python literal like this:
```python
textSize = led.draw_text3(0,0,'451\177F', font)
```

OAuth Usage
===========

```python
    import gaugette.oauth
    CLIENT_ID       = 'your client_id here'
    CLIENT_SECRET   = 'your client secret here'

    oauth = gaugette.oauth.OAuth(CLIENT_ID, CLIENT_SECRET)
    if not oauth.has_token():
        user_code = oauth.get_user_code()
        print "Go to %s and enter the code %s" % (oauth.verification_url, user_code)
        oauth.get_new_token()
```

CapSwitch Usage
===============

```python
    import gaugette.capswitch
    SWICH_PIN = 3
    switch = gaugette.capswitch.CapSwitch(SWITCH_PIN)
    while True:
        if switch.sense():
            print 'sensed'
```

Rgb Led Usage
=============

```python
    import gaugette.rgbled
    R_PIN = 4
    G_PIN = 5
    B_PIN = 6
    led = gaugette.rgbled.RgbLed(R_PIN,R_PIN,B_PIN)
    led.set(0,50,100)
    led.fade(100,0,0)
```

Pin numbers are Wiring pin numbers. They differ from hardware pin or GPIO ids.

Rotary Encoder Usage
====================

```python
    import gaugette.rotary_encoder
    A_PIN = 7
    B_PIN = 9
    encoder = gaugette.rotary_encoder.RotaryEncoder(A_PIN, B_PIN)
    while True:
      delta = encoder.get_delta()
      if delta!=0:
        print "rotate %d" % delta
```

Pin numbers are Wiring pin numbers. They differ from hardware pin or GPIO ids.
Connect your C pin of the encoder to Ground.

Switch Usage
====================

```python
    import gaugette.switch
    SW_PIN = 8
    sw = gaugette.switch.Switch(SW_PIN)
    last_state = sw.state()
    while True:
      state = sw.state()
      if state != last_state:
        print "switch %d" % state
        last_state = state
```

Pin numbers are Wiring pin numbers. They differ from hardware pin or GPIO ids.

You can wire the switch to either GND or Vcc.  By default this class
assumes the free leg of the switch is wired to GND.

```python
    # switch is wired to GND
    sw = gaugette.switch.Switch(SW_PIN)  # pullUp defaults to True
    # which is equivalent to...
    sw = gaugette.switch.Switch(SW_PIN, pullUp=True)
```

If you wire to Vcc you must set the optional pullUp parameter in the constructor to False.

```python
    # switch is wired to Vcc
    sw = gaugette.switch.Switch(SW_PIN, pullUp=False)
```

Regardless of the wiring polarity, the returned results are 0 for switch
open, 1 for switch closed.


Discussion At
=============

[http://guy.carpenter.id.au/gaugette/](http://guy.carpenter.id.au/gaugette/)

Using This Module In Your Code
==============================

You can use this code by checking it out into your source directory as follows:

```
cd myApplication
git clone git://github.com/guyc/py-gaugette.git
ln -s py-gaugette/gaugette gaugette
```
The soft link is created so that the local directory `gaugette` will point to
`py-gaugette/gaugette` which simplifies your import statements.

Alternatively if you want to install this module as a system-wide python module, you can
do so as follows:

```
cd myPythonStuff
git clone git://github.com/guyc/py-gaugette.git
cd py-gaugette
sudo python setup.py install
```


