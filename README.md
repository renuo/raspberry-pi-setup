# 🖥 Raspberry Pi Setup for Renuo Office Dashboard
_Or the big screen at the office entrance_

Setup for Raspberry Pi

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

Runs Chomium in a restart loop.
Kills it every 5min.

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
*/5 * * * * pkill -HUP chromium
```
