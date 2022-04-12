# raspberry-pi-setup
Setup for Raspberry Pi

## Plug in 
Always plug in at HDMI 1

## Access via SSH

Change hostname in: `/etc/hosts` and `/etc/hostname`

IP-Adress and passwort in 1Password


## Install

```
git clone https://github.com/renuo/raspberry-pi-setup.git
cp -R raspberry-pi-setup/config/lxsession/ .config
```
## Troubleshooting

### If only, there is no menu

You can simply restart the Raspberry Pi.
Unplug it from the TV and back in or `ssh pi@FIND-IP-IN-1PASSWORD sudo shutdown -r now`

### If the Raspberry Pi do not start
Unplug it from the TV and plug it in via HDMI 1 to another monitor. You can also plug in a keyboard and optional a mouse. When the Pi is started, you can exit the full screen with right click and click “exit full screen”. After that, you can start the terminal.

