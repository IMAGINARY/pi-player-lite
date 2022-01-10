# Pi Player Lite

# Player setup

## On your desktop system

- Download [Raspberry Pi OS Lite (Legacy)](https://downloads.raspberrypi.org/raspios_oldstable_lite_armhf/images/raspios_oldstable_lite_armhf-2021-12-02/2021-12-02-raspios-buster-armhf-lite.zip) ([Torrent](https://downloads.raspberrypi.org/raspios_oldstable_lite_armhf/images/raspios_oldstable_lite_armhf-2021-12-02/2021-12-02-raspios-buster-armhf-lite.zip.torrent)). SHA256: `9276d71a4793accb4e29ad337f58865fcb92f831716305fc93adf0adb4784129`
- Flash the disk image to a SD card large enough to fit the OS (~2GiB) and your media files.
- Enlarge the root partition (ext4) to ~2GiB
- Create a _third_ partition on the SD card:
- File system: FAT32 (files < 4GiB) or ExFAT (large files)
- Label: `MEDIA`
- Modify files on the `boot` partition:
  - Remove `init=/usr/lib/raspi-config/init_resize.sh` from `cmdline.txt`
  - (Optional) Enable `ssh` by creating an empty file called `ssh`

## On the Raspberry Pi

With the Raspberry Pi connected to the internet:

- Execute the install script:

```
wget https://github.com/IMAGINARY/pi-player-lite/releases/download/v1.1.0/install.sh
chmod +x install.sh
sudo ./install.sh
```

- Activate overlay file system:

```
sudo raspi-config
```

Select `Performance Options` → `Overlay File System` and answer the questions with `Yes`.

- Reboot

## Configuration

The player supports basic configuration by adding the file `/MEDIA/config.txt`.
Each line of the file holds one configuration option and its value separated by `=`.
Supported options include:

```
randomize={0,1}
transform={90,180,270,hflip,vflip,transpose,antitranspose}
wait=<seconds>
```

The `wait` option can be used to delay the start of the media player by the given number of
seconds which is helpful if you need to log into the machine via a local terminal since the login and command prompt would otherwise be overlaid by the media player.

### Screen rotation

It is recommended to enable rotation by adding either

```
transform=90
```

or

```
transform=270
```

to `/MEDIA/config.txt`. This will enable hardware accelerated rotation via VLC,
leaving OS-level screen rotation untouched.

Raspberry Pi also supports the following options in `/boot/config.txt`:

```
lcd_rotate=1
display_lcd_rotate=1
display_hdmi_rotate=1
```

However, our tests led to mixed results. In some cases, the rotation did not work at all.
In other cases, the rotation has been applied, but the video playback suffered
from stuttering, dropouts and especially video tearing.

Therefore, you should prefer the method described above.

### Miscellaneous

- If you power the display and the Raspberry Pi from the same power source, you might need to add `bootcode_delay=<seconds>` to `/boot/config.txt` in order to delay access of the display until it is fully powered on.

## Media files

- Add your media files to the `MEDIA` of the SD card
- By default, all media files are played in a loop.
- The playlist file `autoplay.m3u` can be added to restrict to a subset of the media files or enforce a specific order.
- Encode your videos in h.264.
- The container format does not matter much. VLC supports `.mp4`, `.mkv`, `.mov` and many others.

## Remote control

The player exposed the [VLC telnet interface](https://wiki.videolan.org/Talk:Console/) on port 4212. This can be used for controlling the player remotely, e.g. for triggering actions based through hardware buttons.

Netcat (`nc`) makes it easy to utilise this interface.

Interactive:

```
(echo "pi-player-lite" && cat) | nc [host] 4212
```

Send single command:

```
(echo -e "pi-player-lite\n[cmd]") | nc [host] 4212 -w0
```

Most important commands:

```
add XYZ  . . . . . . . . . . . . . . . . . . . . add XYZ to playlist
enqueue XYZ  . . . . . . . . . . . . . . . . . queue XYZ to playlist
playlist . . . . . . . . . . . . .  show items currently in playlist
play . . . . . . . . . . . . . . . . . . . . . . . . . . play stream
stop . . . . . . . . . . . . . . . . . . . . . . . . . . stop stream
next . . . . . . . . . . . . . . . . . . . . . .  next playlist item
prev . . . . . . . . . . . . . . . . . . . .  previous playlist item
goto [X], gotoitem [X] . . . . . . . . . . . . .  goto item at index
pause  . . . . . . . . . . . . . . . . . . . . . . . .  toggle pause
volume [X] . . . . . . . . . . . . . . . . . .  set/get audio volume
volup [X]  . . . . . . . . . . . . . . .  raise audio volume X steps
voldown [X]  . . . . . . . . . . . . . .  lower audio volume X steps
help, ? [pattern]  . . . . . . . . . . . . . . . . .  a help message
longhelp [pattern] . . . . . . . . . . . . . . a longer help message
logout . . . . . . . . . . . . . .  exit (if in a socket connection)
```

## Building the installer script

You need `make` and [`makeself`](https://makeself.io/). Once installed, run

```
make
```

## Credits

Developed by Christian Stussak for IMAGINARY gGmbH.

## License

Copyright 2021 IMAGINARY gGmbH

Licensed under the Apache License, Version 2.0.
