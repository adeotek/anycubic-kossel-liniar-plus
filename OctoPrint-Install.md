# OctoPrint on RaspberryPi

## RaspberryPi OS

### Install 64-bit version on SSD/SD
Install Raspberry Pi OS using Raspberry Pi Imager.

### RaspberryPi OS Configuration
```bash
sudo apt install curl wget nano mc netcat
```

## OctoPrint

### Install

#### Install dependencies & configure Python environment
```bash
sudo apt install python3-pip python3-dev python3-setuptools python3-venv git libyaml-dev build-essential
mkdir OctoPrint && cd OctoPrint
python3 -m venv venv
source venv/bin/activate
```

#### Update PIP & install OctoPrint
```bash
pip install pip --upgrade
pip install octoprint
```

```bash
sudo usermod -a -G tty pi
sudo usermod -a -G dialout pi
```

#### Create and configure OctoPrint service
```bash
wget https://github.com/OctoPrint/OctoPrint/raw/master/scripts/octoprint.service && sudo mv octoprint.service /etc/systemd/system/octoprint.service
sudo systemctl start octoprint.service
sudo systemctl enable octoprint.service
```

#### Install HAProxy
```bash
sudo apt install haproxy
sudo nano /etc/haproxy/haproxy.cfg
```
Sample Configuration:
```yaml
global
    log /dev/log    127.0.0.1 local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

    maxconn 4096

    # Default SSL material locations
    ca-base /etc/ssl/certs
    crt-base /etc/ssl/private

    # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&confi>        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA2>        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_>        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

    retries 3
    option redispatch
    option http-server-close
    option forwardfor
    maxconn 2000

frontend public
    bind :::80 v4v6
    use_backend webcam if { path_beg /webcam/ }
    default_backend octoprint

backend octoprint
    #reqrep ^([^\ :]*)\ /(.*)     \1\ /\2
    option forwardfor
    server octoprint1 127.0.0.1:5000

backend webcam
    #reqrep ^([^\ :]*)\ /webcam/(.*)     \1\ /\2
    http-request replace-path /webcam/(.*)   /\1
    server webcam1  127.0.0.1:8080
```

Start & enable HAProxy service
```bash
sudo systemctl start haproxy
sudo systemctl enable haproxy
```

### OctoPrint Configuration

#### Open config file
```bash
nano ~/.octoprint/config.yaml
```
#### Add configuration
```
server:
    host: 127.0.0.1

```

### Server commands

#### Enable `pi` user to emit `shutdown` commands
```bash
sudo nano /etc/sudoers.d/octoprint-shutdown
```
Add content:
```
pi ALL=NOPASSWD: /sbin/shutdown
```

#### Enable `pi` user to emit `service` commands
```bash
sudo nano /etc/sudoers.d/octoprint-service
```
Add content:
```
pi ALL=NOPASSWD: /usr/sbin/service
```

### Webcam

#### Dependecies
```bash
cd ~
sudo apt install subversion libjpeg62-turbo-dev imagemagick ffmpeg libv4l-dev cmake
git clone https://github.com/jacksonliam/mjpg-streamer.git
cd mjpg-streamer/mjpg-streamer-experimental
export LD_LIBRARY_PATH=.
make
```

#### Webcam daemon
```bash
cd ~
mkdir scripts
nano /home/pi/scripts/webcamDaemon
```
Add content:
```bash
#!/bin/bash

MJPGSTREAMER_HOME=/home/pi/mjpg-streamer/mjpg-streamer-experimental
MJPGSTREAMER_INPUT_USB="input_uvc.so"
MJPGSTREAMER_INPUT_RASPICAM="input_raspicam.so"

# init configuration
camera="auto"
camera_usb_options="-r 640x480 -f 10"
camera_raspi_options="-fps 10"

if [ -e "/boot/octopi.txt" ]; then
    source "/boot/octopi.txt"
fi

# runs MJPG Streamer, using the provided input plugin + configuration
function runMjpgStreamer {
    input=$1
    pushd $MJPGSTREAMER_HOME
    echo Running ./mjpg_streamer -o "output_http.so -w ./www" -i "$input"
    LD_LIBRARY_PATH=. ./mjpg_streamer -o "output_http.so -w ./www" -i "$input"
    popd
}

# starts up the RasPiCam
function startRaspi {
    logger "Starting Raspberry Pi camera"
    runMjpgStreamer "$MJPGSTREAMER_INPUT_RASPICAM $camera_raspi_options"
}

# starts up the USB webcam
function startUsb {
    logger "Starting USB webcam"
    runMjpgStreamer "$MJPGSTREAMER_INPUT_USB $camera_usb_options"
}

# we need this to prevent the later calls to vcgencmd from blocking
# I have no idea why, but that's how it is...
vcgencmd version

# echo configuration
echo camera: $camera
echo usb options: $camera_usb_options
echo raspi options: $camera_raspi_options

# keep mjpg streamer running if some camera is attached
while true; do
    if [ -e "/dev/video0" ] && { [ "$camera" = "auto" ] || [ "$camera" = "usb" ] ; }; then
        startUsb
    elif [ "`vcgencmd get_camera`" = "supported=1 detected=1" ] && { [ "$camera" = "auto" ] || [ "$camera" = "raspi" ] ; }; then
        startRaspi
    fi

    sleep 120
done
```

#### Change permissions
```bash
chmod +x /home/pi/scripts/webcamDaemon
```

#### Create service
```bash
sudo nano /etc/systemd/system/webcamd.service
```
Add content:
```
[Unit]
Description=Camera streamer for OctoPrint
After=network-online.target OctoPrint.service
Wants=network-online.target

[Service]
Type=simple
User=pi
ExecStart=/home/pi/scripts/webcamDaemon

[Install]
WantedBy=multi-user.target
```

#### Enable & start service
```bash
sudo systemctl daemon-reload
sudo systemctl enable webcamd.service
sudo systemctl start webcamd.service
```

#### Disable remote access (access allowed only through HAProxy)
```bash
sudo /sbin/iptables -A INPUT -p tcp -i wlan0 ! -s 127.0.0.1 --dport 8080 -j DROP    # for ipv4
sudo /sbin/ip6tables -A INPUT -p tcp -i wlan0 ! -s ::1 --dport 8080 -j DROP         # for ipv6

sudo apt install iptables-persistent
sudo /sbin/ip6tables-save > /etc/iptables/rules.v6
sudo /sbin/iptables-save > /etc/iptables/rules.v4
```

## RaspberryPi Packages & Utilities

### Video4Linux Control Panel !!!
```
v4l2ucp
```

## OctoPrint Configuration

### Printer
- Serial port: AUTO
- Baudrate: 115200
- Request exclusive access to the serial port: ON
- Enable automatic firmware detection: ON
- Blocked commands: M0, M1
- Pausing commands: M0, M1, M25
- Emergency commands: M112, M108, M410
- What to do on a firmware error (Error: or !!): Disconnect from the printer
- Send M112 on disconnect due to error: ON
- Log position on pause: ON

### Printer Profile
- Name: Anycubic KOSSEL
- Identifier: _default
- Model: Anycubic Kossel Liniar Plus
- Form Factor: Circular
- Heated bed: ON
- Diameter: 220 mm
- Height: 295 mm
- X: 1000 mm/min
- Y: 1000 mm/min
- Z: 200 mm/min
- E: 300 mm/min
- Nozzle Diameter: 0.4 mm
- Number of Extruders: 1

### Temperatures
- ABS: 210 C / 100 C
- PLA: 200 C / 60 C

### GCODE Scripts
#### After print job is cancelled
```
; disable motors
M84

;disable all heaters
{% snippet 'disable_hotends' %}
{% snippet 'disable_bed' %}
;disable fan
M106 S0
```
#### After print job is paused
```
{% if pause_position.x is not none %}
; relative XYZE
G91
M83

; retract filament, move Z slightly upwards
G1 Z+5 E-5 F4500

; absolute XYZE
M82
G90

; move to a safe rest position, adjust as necessary
G1 X0 Y0
{% endif %}
```

#### Before print job is resumed
```
{% if pause_position.x is not none %}
; relative extruder
M83

; prime nozzle
G1 E-5 F4500
G1 E5 F4500
G1 E5 F4500

; absolute E
M82

; absolute XYZ
G90

; reset E
G92 E{{ pause_position.e }}

; move back to pause position XYZ
G1 X{{ pause_position.x }} Y{{ pause_position.y }} Z{{ pause_position.z }} F4500

; reset to feed rate before pause if available
{% if pause_position.f is not none %}G1 F{{ pause_position.f }}{% endif %}
{% endif %}
```

## Webcam
- Enable webcam support: ON
- Stream URL: http://adeotek.go.ro/webcam/?action=stream
- Snapshot URL: http://127.0.0.1:8080/?action=snapshot
- Path to FFMPEG: /usr/bin/ffmpeg

## Server
- Restart OctoPrint: sudo service octoprint restart
- Restart system: sudo shutdown -r now
- Shutdown system: sudo shutdown -h now

## Folders
- Upload Folder: /home/pi/.octoprint/uploads
- Timelapse Folder: /home/pi/.octoprint/timelapse
- Timelapse Temp Folder: /home/pi/.octoprint/timelapse/tmp
- Logs Folder: /home/pi/.octoprint/logs
- Watched Folder: /home/pi/.octoprint/watched

## Plugins
- UI Customizer
- Themeify
- Terminal Commands Extended
- Status Line
- Stateful Sidebar
- Simple Filament Change Buttons
- Octoprint Display ETA
- PrintJobHistory
- Resource Monitor
- Dashboard
- Active Filters Extended
- Macro
- DisplayLayerProgress Plugin