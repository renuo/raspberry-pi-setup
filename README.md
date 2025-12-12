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

## Changing code

The raspberry pi is in read only mode thanks to overlayFs. If you want to make changes to anything(settings, scripts, etc.) you will need to disable overlayFS. You can do this after you SSH into the raspberry.

First you will need to open the config gui with:
```
sudo raspi-config
```

There you will go to the performance tab and disable the Overlay File system. Reboot the raspi with 
```
sudo reboot
```

Afterwards you can SSH into it again and make your changes. Don't forget to enable overlayFs again after you made you changes as shutting down the raspi incorrectly would lead to data corruption otherwise.

## Songs

The songs which are played when pressing the button are stored on [Google drive](https://drive.google.com/drive/folders/10stn3i1LZsxPFS9V6kx0v5ma3RJS8Yqz)

If you name a song lunch.mp3 it will be played at lunch time. Otherwise they will be played on random.
The song script is saved in the home/pi folder. it is called script.py if you want to make changes.

Adding a new song to the folder will not make it available immediatly. The raspberry will only fetch the songs when it gets turned on. So if you want to make the songs available immediatly, reboot the device.

Changes to the fetching allgorithm can be made in the etc/systemd/system/fetch-songs.service

