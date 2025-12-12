# 🖥 Raspberry Pi Setup for Renuo Entrance and Big Monitor Screen

Setup for Raspberry Pi

## WiFi

Please make sure you are connected to the 5 GHz frequency, and not 2.4 GHz. Also make sure the resolution is 1920x1080, since that impacts the speed of WiFi.

The Raspberry Pis have static IP addresses assigned. If you mess with the
network configuration you need to make sure to not break this. See
<http://192.168.100.1/webfig/#IP:DHCP_Server.Leases>

## Plug in 
Always plug in at HDMI 1

## Access via SSH

Change hostname in: `/etc/hosts` and `/etc/hostname`

IP Address and password in 1Password

## Install

```
# Install emojis 
curl https://gist.githubusercontent.com/luminousmen/7ba2f1f672213a396e8c42a3802348df/raw/434c8ef6bad8bb0df8c794bf598021eef7eebc7c/ubuntu-chrome-emoji.sh | bash 

# Clone repo
git clone https://github.com/renuo/raspberry-pi-setup.git

# Override configs
cp -R raspberry-pi-setup/config/lxsession/ .config
# --> Replace now the url in .config/lxsession/LXDE-pi/autostart

# Set boot config
sudo cp raspberry-pi-setup/boot/firmware/config.txt /boot/firmware/config.txt
```

### Use x11 instead of wayland

Use the raspi-config to update the window manager:

https://www.geeks3d.com/20240509/how-to-switch-from-wayland-to-x11-on-raspberry-pi-os-bookworm/

## Changing code

The raspberry pis are in read only mode thanks to overlayFs. If you want to make changes to anything(settings, scripts, etc.) you will need to disable overlayFS. You can do this after you SSH into the raspberry.

First you will need to open the config gui with:
```
sudo raspi-config
```

There you will go to the performance tab and disable the Overlay File system. Reboot the raspi with
```
sudo reboot
```

Afterwards you can SSH into it again and make your changes. Don't forget to enable overlayFs again after you made you changes as shutting down the raspi incorrectly would lead to data corruption otherwise.

## Troubleshooting

### If only, there is no menu

You can simply restart the Raspberry Pi.
Unplug it from the TV and back in or `ssh pi@FIND-IP-IN-1PASSWORD sudo shutdown -r now`

### If the Raspberry Pi do not start
Unplug it from the TV and plug it in via HDMI 1 to another monitor. You can also plug in a keyboard and optional a mouse. When the Pi is started, you can exit the full screen with right click and click "exit full screen". After that, you can start the terminal.

### Change your URL 
You can change your URL and the scaling factor in `.config/lxsession/LXDE-pi/autostart`
by changing this line

```
@chromium-browser --kiosk <YOUR-URL> -force-device-scale-factor=<SCALING-FACTOR>
```

# Big Monitor

```
ssh pi@raspberrypi-big-monitor.lan.renuo.ch
```

# Entrance Monitor

```
ssh pi@raspberrypi-entrance.lan.renuo.ch
```

Runs Chomium in a restart loop.
Reloads it every 5min.

```
# /home/pi/.config/lxsession/LXDE-pi/autostart
#!/bin/sh

@xset s 0 0
@xset s noblank
@xset s noexpose
xscreensaver -no-splash
xrandr --output XWAYLAND0 --primary --mode 3840x2160
DISPLAY=:0 unclutter -idle 0 -root &

@/home/pi/kiosk.sh
```

```
# /home/pi/kiosk.sh
#!/bin/bash

while true; do
  chromium-browser --kiosk \
    "<YOUR-URL>/pub?start=true&loop=true&delayms=5000&slide=<FIRST_SLIDE>"
  sleep 2
done
```

```
# crontab -e
*/5 * * * *          DISPLAY=:0.0 xdotool key F5
```

## Songs

The songs which are played when pressing the button are stored on [Google
drive](https://drive.google.com/drive/folders/10stn3i1LZsxPFS9V6kx0v5ma3RJS8Yqz)

If you name a song lunch.mp3 it will be played at lunch time. Otherwise they
will be played on random. The song script is saved in the `/home/pi/script.py`:

```python
import time
import logging
from gpiozero import Button
import pygame
from datetime import datetime, date, timedelta
import os
import random

SONG_DIR = "/home/pi/songs/Entrance Monitor Songs"
LUNCH_SONG_FILENAME = "lunch.mp3"
COUNTER_FILE = "/home/pi/counter.txt"
LUNCH_HOUR = 12
LUNCH_MINUTE = 30

logging.basicConfig(filename="/home/pi/button_press.log",
                    level=logging.INFO,
                    format="%(asctime)s - %(message)s")

pygame.mixer.init()
button = Button(18, pull_up=True, bounce_time=0.2)

def ensure_counter_file():
    if not os.path.exists(COUNTER_FILE):
        with open(COUNTER_FILE, "w") as f:
            f.write("0")

def increment_counter():
    ensure_counter_file()
    with open(COUNTER_FILE, "r+") as f:
        try:
            count = int(f.read())
        except ValueError:
            count = 0
        f.seek(0); f.write(str(count + 1)); f.truncate()

def list_mp3_files():
    try:
        return [f for f in os.listdir(SONG_DIR) if f.lower().endswith(".mp3")]
    except Exception as e:
        logging.error(f"Directory error: {e}")
        return []

def lunch_actual_name(files):
    m = {f.lower(): f for f in files}
    return m.get(LUNCH_SONG_FILENAME.lower())

def play_path(path):
    try:
        pygame.mixer.music.load(path)
        pygame.mixer.music.set_volume(1.0)
        pygame.mixer.music.play()
        logging.info(f"Playback started: {path}")
    except Exception as e:
        logging.error(f"Playback failed: {e}")

def play_random(files):
    p = os.path.join(SONG_DIR, random.choice(files))
    logging.info(f"Selected random song: {p}")
    play_path(p)

def next_lunch_after(dt):
    t = dt.replace(hour=LUNCH_HOUR, minute=LUNCH_MINUTE, second=0, microsecond=0)
    return t if dt < t else t + timedelta(days=1)

def on_button_press():
    logging.info("Button pressed.")
    increment_counter()
    files = list_mp3_files()
    if not files:
        logging.warning("No .mp3 files found.")
        return
    play_random(files)

button.when_pressed = on_button_press

print("Ready: random on button, lunch autoplay daily.")

next_lunch = next_lunch_after(datetime.now())

while True:
    try:
        now = datetime.now()
        if now < next_lunch:
            time.sleep(1)
            continue

        files = list_mp3_files()
        if not files:
            logging.warning("No .mp3 files found.")
            next_lunch = next_lunch_after(now + timedelta(seconds=1))
            time.sleep(60)
            continue

        lunch = lunch_actual_name(files)
        if lunch:
            play_path(os.path.join(SONG_DIR, lunch))
        else:
            logging.warning("lunch.mp3 not found, playing random instead.")
            play_random(files)

        next_lunch = next_lunch_after(now + timedelta(seconds=1))
        time.sleep(60)

    except Exception as e:
        logging.error(f"Autoplay loop error: {e}")
        time.sleep(2)
```

Adding a new song to the folder will not make it available immediately. The
raspberry will only fetch the songs when it gets turned on. So if you want to
make the songs available immediately, reboot the device.

Changes to the fetching algorithm can be made in the
`/etc/systemd/system/fetch-songs.service` / `/usr/local/bin/fetch-songs.sh`:

```toml
[Unit]
Description=Fetch songs from Google Drive at boot
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/fetch-songs.sh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

```bash
#!/bin/bash
set -e

# Variables
VENV_PATH="/home/pi/.gdownenv"
SONG_DIR="/home/pi/songs"
GDRIVE_FOLDER="https://drive.google.com/drive/folders/10stn3i1LZsxPFS9V6kx0v5ma3RJS8Yqz"

# Create directory
mkdir -p "$SONG_DIR"
cd "$SONG_DIR"

# Activate gdown venv and download folder
source "$VENV_PATH/bin/activate"
gdown --folder "$GDRIVE_FOLDER"
deactivate

echo "Songs downloaded to $SONG_DIR"
```
