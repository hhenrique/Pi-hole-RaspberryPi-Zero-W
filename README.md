# Pi-hole on Raspberry Pi Zero W

Step by step instructions for installing Pi-hole on a Raspberry PI Zero W with Adafruit miniPiTFT 1.14" 240x135.

## Components

1. [Raspiberry PI Zero W](https://www.adafruit.com/product/3400)
2. Mini SD Card 16 GB
3. [Adafruit miniPiTFT 1.14" 240x135](https://www.adafruit.com/product/4393)
4. Mini-HDMI to HDMI cable
5. Monitor
6. Power supply 5V 2.5A

## Installing the OS

1. Download the latest [**Raspbian Buster Lite**](https://www.raspberrypi.org/downloads/raspbian/).
2. Flash the Raspbian image to the SD Card.
3. Re-plug the SD Card into the laptop.
4. Open drive D: (Boot) with Windows Explorer
5. Edit file `D:\config.txt` to enable UART. It allows a USB console cable to be attached later for troubleshooting. Add these lines at the end of the file:
```
# Enable UART
enable_uart=1
```
6. Create file `D:\wpa_supplicant.conf` to configue WiFi with the content below. Replace _YOURSSID_ and _YOURPASSWORD_ accordingly:
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US

network={
    ssid="YOURSSID"
    psk="YOURPASSWORD"
    scan_ssid=1
}
```
7. Create file `D:\ssh` to enable SSH. The file shall be empty.
8. Eject the SD Card safely from the laptop.
9. Plug the SD Card into the Raspberry Pi.
10. Power the Raspberry Pi on.
11. SSH to the Raspberry Pi using Putty. The default password is `raspberry`.
```
putty.exe pi@raspberrypi.local
```
12. change the default password with `passwd`.
13. Update the OS with:
```
sudo apt-get update
sudo apt-get upgrade
```
14. Reboot
```
sudo reboot
```

## Network setup

1. SSH to the Raspberry Pi using Putty. Note the password was changed.
```
putty.exe pi@raspberrypi.local
```
2. Change the hostname with `sudo nano /etc/hostname` file. The file has only one word: `raspberrypi`. Replace it with `pi-hole`.
3. Change the hostname with `sudo nano /etc/hosts` file.  Replace `raspberrypi` with `pi-hole` (last line):
```
27.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1               pi-hole
```
4. Check the current IP address with `ip -4 addr show | grep global`:
```
    inet 192.168.0.18/24 brd 192.168.0.255 scope global noprefixroute wlan0
```
5. Assign the static IP address `192.168.0.2` with `sudo nano /etc/dhcpcd.conf` file. Add the text below at the end of the `/etc/dhcpcd.conf` file:
```
interface wlan0
static ip_address=192.168.0.2/24
static ip6_address=fd51:42f8:caae:d92e::ff/64
static routers=192.168.0.1
static domain_name_servers=8.8.8.8 8.8.4.4 2001:4860:4860::8888 2001:4860:4860::8844
```
6. Reboot
```
sudo reboot
```
7. SSH to the Raspberry Pi using Putty.
```
putty.exe pi@pi-hole.local
```
4. Check the current IP address is indeed `192.168.0.2` with `ip -4 addr show | grep global`:
```
    inet 192.168.0.2/24 brd 192.168.0.255 scope global noprefixroute wlan0
```

## Python 3 Setup

1. SSH to the Raspberry Pi using Putty.
```
putty.exe pi@pi-hole.local
```
2. Install `pip`
```
sudo apt-get install python3-pip
```

## Hardware setup

1. SSH to the Raspberry Pi using Putty.
```
putty.exe pi@pi-hole.local
```
2. Install I2C support packages
```
sudo apt-get install -y python-smbus
sudo apt-get install -y i2c-tools
```
3. Install Kernel support with `sudo raspi-config`:
`5 Interfacing Options` > `P4 SPI` > `Yes` > `Ok`
`5 Interfacing Options` > `P5 I2C` > `Yes` > `Ok`
`Finish`
4. Reboot
```
sudo reboot
```
5. SSH to the Raspberry Pi using Putty.
```
putty.exe pi@pi-hole.local
```
7. Check I2C with `sudo i2cdetect -y 1`. The result should be:
```
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```
8. Check SPI with `ls -l /dev/spidev*`. The result should be:
```
crw-rw---- 1 root spi 153, 0 May 18 18:02 /dev/spidev0.0
crw-rw---- 1 root spi 153, 1 May 18 18:02 /dev/spidev0.1
```
9. Install Python libraries with `pip3 install RPI.GPIO`
```
Looking in indexes: https://pypi.org/simple, https://www.piwheels.org/simple
Collecting RPI.GPIO
  Downloading https://www.piwheels.org/simple/rpi-gpio/RPi.GPIO-0.7.0-cp37-cp37m-linux_armv6l.whl (69kB)
    100% |████████████████████████████████| 71kB 163kB/s
Installing collected packages: RPI.GPIO
Successfully installed RPI.GPIO-0.7.0
```

## Install Pi-hole

1. Download and run Pi-hole installer. [Screenshots](https://learn.adafruit.com/pi-hole-ad-blocker-with-pi-zero-w/install-pi-hole)
```
curl -sSL https://install.pi-hole.net | bash
```
2. `Ok` for installer greetings.
3. `OK` for Pi-hole donation screen.
4. `Ok` for _Static IP Needed_ warning.
5. `Google (ECS)` for Upstream DNS provider.
6. `Ok` for third party lists.
7. `Ok` for IPv4 and IPv6.
8. `Yes` for IP address `192.168.0.2/24` and gateway `192.168.0.1`.
9. `Ok` for _FYI: IP Conflict_ warning.
10. `Ok` for IPv6 supported confirmation.
11. `(*) On (Recommended)` and `Ok` for installing web admin interface.
12. `(*) On (Recommended)` and `Ok` for installing web server (lighttpd).
13. `(*) On (Recommended)` and `Ok` for logging queries.
14. `(*) 0 Show everything` and `Ok` for privacy mode.
15. Test the [web admin interface](http://pi-hole.local/admin/index.php). Note the generated password, e.g., _MYq2XsXX_.

## Install Mini PiTFT

reference: https://learn.adafruit.com/pitft-linux-python-animated-gif-player/python-setup-2

1. Install RBG Display Library
```
sudo pip3 install adafruit-circuitpython-rgb-display
sudo pip3 install --upgrade --force-reinstall spidev
```
2. Install DejaVu TTF font
```
sudo apt-get install ttf-dejavu
```
3. Install Pillow library
```
sudo apt-get install python3-pil
```
4. Install NumPy library
```
sudo apt-get install python3-numpy
```
5. Create `rbg_display_minipitfttest.py` file with this content:
```
import digitalio
import board

from adafruit_rgb_display.rgb import color565
import adafruit_rgb_display.st7789 as st7789

# Configuration for CS and DC pins for Raspberry Pi
cs_pin = digitalio.DigitalInOut(board.CE0)
dc_pin = digitalio.DigitalInOut(board.D25)
reset_pin = None
BAUDRATE = 64000000  # The pi can be very fast!
# Create the ST7789 display:
display = st7789.ST7789(
    board.SPI(),
    cs=cs_pin,
    dc=dc_pin,
    rst=reset_pin,
    baudrate=BAUDRATE,
    width=135,
    height=240,
    x_offset=53,
    y_offset=40,
)

backlight = digitalio.DigitalInOut(board.D22)
backlight.switch_to_output()
backlight.value = True
buttonA = digitalio.DigitalInOut(board.D23)
buttonB = digitalio.DigitalInOut(board.D24)
buttonA.switch_to_input()
buttonB.switch_to_input()

# Main loop:
while True:
    if buttonA.value and buttonB.value:
        backlight.value = False  # turn off backlight
    else:
        backlight.value = True  # turn on backlight
    if buttonB.value and not buttonA.value:  # just button A pressed
        display.fill(color565(255, 0, 0))  # red
    if buttonA.value and not buttonB.value:  # just button B pressed
        display.fill(color565(0, 0, 255))  # blue
    if not buttonA.value and not buttonB.value:  # none pressed
        display.fill(color565(0, 255, 0))  # green
```
6. Execute the test with `sudo python3 rgb_display_minipitfttest.py`. The top button should make the display light up _red_, the bottom _blue_, and pressing both at the same time should make it _green_. Press `CTRL+C` to stop.
7. Install requests module
```
sudo pip3 install requests
```
8. Test Pi-hole API using `python3`:
```
import json
import requests

api_url = 'http://localhost/admin/api.php'
r = requests.get(api_url)
data = json.loads(r.text)
print(data)
```
The result should be similar to:
```
{'domains_being_blocked': 84889, 'dns_queries_today': 687, 'ads_blocked_today': 79, 'ads_percentage_today': 11.499272, 'unique_domains': 358, 'queries_forwarded': 423, 'queries_cached': 184, 'clients_ever_seen': 4, 'unique_clients': 4, 'dns_queries_all_types': 687, 'reply_NODATA': 89, 'reply_NXDOMAIN': 21, 'reply_CNAME': 254, 'reply_IP': 312, 'privacy_level': 0, 'status': 'enabled', 'gravity_last_updated': {'file_exists': True, 'absolute': 1589825649, 'relative': {'days': 0, 'hours': 0, 'minutes': 32}}}
```
type `exit()` to exit.

## Display Pi-hole stats on miniPiTFT

1. Create file `stats.py` with this content:
```
# -*- coding: utf-8 -*-
# Import Python System Libraries
import time
import json
import subprocess

# Import Requests Library
import requests

#Import Blinka
import digitalio
import board

# Import Python Imaging Library
from PIL import Image, ImageDraw, ImageFont
import adafruit_rgb_display.st7789 as st7789

api_url = 'http://localhost/admin/api.php'

# Configuration for CS and DC pins (these are FeatherWing defaults on M0/M4):
cs_pin = digitalio.DigitalInOut(board.CE0)
dc_pin = digitalio.DigitalInOut(board.D25)
reset_pin = None

# Config for display baudrate (default max is 24mhz):
BAUDRATE = 64000000

# Setup SPI bus using hardware SPI:
spi = board.SPI()

# Create the ST7789 display:
disp = st7789.ST7789(spi, cs=cs_pin, dc=dc_pin, rst=reset_pin, baudrate=BAUDRATE,
                     width=135, height=240, x_offset=53, y_offset=40)

# Create blank image for drawing.
# Make sure to create image with mode 'RGB' for full color.
height = disp.width   # we swap height/width to rotate it to landscape!
width = disp.height
image = Image.new('RGB', (width, height))
rotation = 90

# Get drawing object to draw on image.
draw = ImageDraw.Draw(image)

# Draw a black filled box to clear the image.
draw.rectangle((0, 0, width, height), outline=0, fill=(0, 0, 0))
disp.image(image, rotation)
# Draw some shapes.
# First define some constants to allow easy resizing of shapes.
padding = -2
top = padding
bottom = height-padding
# Move left to right keeping track of the current x position for drawing shapes.
x = 0


# Alternatively load a TTF font.  Make sure the .ttf font file is in the
# same directory as the python script!
# Some other nice fonts to try: http://www.dafont.com/bitmap.php
font = ImageFont.truetype('/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf', 24)

# Turn on the backlight
backlight = digitalio.DigitalInOut(board.D22)
backlight.switch_to_output()
backlight.value = True

# Add buttons as inputs
buttonA = digitalio.DigitalInOut(board.D23)
buttonA.switch_to_input()

while True:
    # Draw a black filled box to clear the image.
    draw.rectangle((0, 0, width, height), outline=0, fill=0)

    # Shell scripts for system monitoring from here:
    # https://unix.stackexchange.com/questions/119126/command-to-display-memory-usage-disk-usage-and-cpu-load
    cmd = "hostname -I | cut -d\' \' -f1"
    IP = "IP: "+subprocess.check_output(cmd, shell=True).decode("utf-8")
    cmd = "hostname | tr -d \'\\n\'"
    HOST = subprocess.check_output(cmd, shell=True).decode("utf-8")
    cmd = "top -bn1 | grep load | awk '{printf \"CPU Load: %.2f\", $(NF-2)}'"
    CPU = subprocess.check_output(cmd, shell=True).decode("utf-8")
    cmd = "free -m | awk 'NR==2{printf \"Mem: %s/%s MB  %.2f%%\", $3,$2,$3*100/$2 }'"
    MemUsage = subprocess.check_output(cmd, shell=True).decode("utf-8")
    cmd = "df -h | awk '$NF==\"/\"{printf \"Disk: %d/%d GB  %s\", $3,$2,$5}'"
    Disk = subprocess.check_output(cmd, shell=True).decode("utf-8")
    cmd = "cat /sys/class/thermal/thermal_zone0/temp |  awk \'{printf \"CPU Temp: %.1f C\", $(NF-0) / 1000}\'" # pylint: disable=line-too-long

    # Pi Hole data!
    try:
        r = requests.get(api_url)
        data = json.loads(r.text)
        DNSQUERIES = data['dns_queries_today']
        ADSBLOCKED = data['ads_blocked_today']
        CLIENTS = data['unique_clients']
    except KeyError:
        time.sleep(1)
        continue

    y = top
    if not buttonA.value:  # just button A pressed
        draw.text((x, y), IP, font=font, fill="#FFFF00")
        y += font.getsize(IP)[1]
        draw.text((x, y), CPU, font=font, fill="#FFFF00")
        y += font.getsize(CPU)[1]
        draw.text((x, y), MemUsage, font=font, fill="#00FF00")
        y += font.getsize(MemUsage)[1]
        draw.text((x, y), Disk, font=font, fill="#0000FF")
        y += font.getsize(Disk)[1]
        draw.text((x, y), "DNS Queries: {}".format(DNSQUERIES), font=font, fill="#FF00FF")
    else:
        draw.text((x, y), IP, font=font, fill="#FFFF00")
        y += font.getsize(IP)[1]
        draw.text((x, y), HOST, font=font, fill="#FFFF00")
        y += font.getsize(HOST)[1]
        draw.text((x, y), "Ads Blocked: {}".format(str(ADSBLOCKED)), font=font, fill="#00FF00")
        y += font.getsize(str(ADSBLOCKED))[1]
        draw.text((x, y), "Clients: {}".format(str(CLIENTS)), font=font, fill="#0000FF")
        y += font.getsize(str(CLIENTS))[1]
        draw.text((x, y), "DNS Queries: {}".format(str(DNSQUERIES)), font=font, fill="#FF00FF")
        y += font.getsize(str(DNSQUERIES))[1]

    # Display image.
    disp.image(image, rotation)
    time.sleep(.1)
```
2. Test with `sudo python3 stats.py`
3. Run the script at boot with `sudo nano /etc/rc.local` and add `sudo python3 /home/pi/stats.py` **before** the `exit 0`:
4. Reboot
```
sudo reboot
```
