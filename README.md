# thinkpad-x395-freebsd-config
Files and configurations to make FreeBSD work on the ThinkPad x395 laptop

## Files included
All included files are in the `filesystem/` directory, relative to my system root (`/`).
Portions of my system's `/etc/rc.conf` and `/boot/loader.conf` are included. I tried to include some comments saying which entries are necessary vs. nice-to-have.

# Getting stuff working

## Booting
Put this in `/boot/loader.conf`:

```
hint.uart.0.at=0
```

## Video (`acpi_video`, `amdgpu`)
In `/etc/rc.conf`:

```
kld_list="amdgpu acpi_video"
```

In `/boot/loader.conf`:

```
acpi_video_load="YES"
```

### External displays

Running `xrandr` once does the trick for KDE. (HDMI tested, USB-C apparently works for others)

## Battery information (`apm`)
You can use the `apm` program to view and control AC-line and battery stuff. Includes levels, status codes, system suspend, etc. It's included with the base install - run `man apm` on your system to view the manpage.

I've included by own battery information script that formats stuff for my KDE topbar in `filesystem/home/tim/bin/battstat`.

## WiFi (`iwm0`, Intel 9260)
My system uses an Intel 9260 WiFi card as `iwm0`. The `filesystem/etc/rc.conf` file contains all the WiFi configuration entries I'm using in my own `/etc/rc.conf`. I paired these settings with the following settings in `/etc/wpa_supplicant.conf`:

```
ctrl_interface=/var/run/wpa_supplicant

network={
 ssid="NetworkName"
 priority=146
 scan_ssid=1
 psk="NetworkPassword"
}
```

You can also use the `wpa_cli` tool post-login.

## Keyboard function keys (volume, brightness)

In `/etc/rc.conf`:

```
kld_list="acpi_ibm"
```

In `/boot/loader.conf`:

```
acpi_ibm_load="YES"
```

Running KDE 5, I can confirm visually (and obviously audibly) that the volume keys work with no configuration, but I haven't bothered getting the speaker mute and microphone mute lights to come on and off. (TODO)

To get the brightness keys working, see the example brightness adjusting script at `filesystem/usr/local/bin/bright`. Note I'm not using brightness increments in line with what ACPI thinks should be happening, but it still works and I like the granularity. You will also have to copy `filesystem/etc/devd/acpi_brightness.conf` to your computer, which will configure the function keys to work as long as `devd` is running (`devd_enable="YES"` in `/etc/rc.conf` - you can also (re)start it with `service devd restart`).

The codes `0x10` and `0x11` refer to the brightness-up and -down functions, respectively. To see other function keys' outputs and do stuff when they're pressed, run the following as root and then start hitting the function keys to get the info you'll need to write your own config:

```
cat /var/run/devd.pipe
```

devd can be restarted after configuration changes by running `service devd restart`. You should enable devd to start automatically by writing `devd_enable="YES"` to `/etc/rc.conf`.

# Final steps after system setup
If you haven't, consider downloading and installing a system upgrade with `freebsd-update`.

If you're a developer, there is an Atom port but it was officially discontinued (I think after 12?) earlier in 2021. @tagattie on GitHub has provided an experimental port based on the official one but the official release binary seems to be broken for FreeBSD 13 if your harfbuzz package is at version 3, but apparently works if harfbuzz is downgraded to 2.9.1, but I haven't tried. I've been using VSCode/Code-OSS (editors/vscode) in lieu of Atom :(

