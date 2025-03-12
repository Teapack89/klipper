# klipper
# Official klipper v0.12.0 + desuuuu DWIN T5UID1 touchscreen mod

You need modify FW in display:

https://github.com/Desuuuu/DGUS-reloaded-Klipper/wiki/Flashing-the-firmware

https://github.com/Desuuuu/DGUS-reloaded-Klipper/releases

## This procedure below is for the Ender 6!

1) Make SD card with raspberry pi imager:
raspios lite 64bit
set user pi/<yourpassword>
nastavit hostname <hostname>
Enable SSH

2) enable SPI and I2C Interface:
```
sudo raspi-config
```

3) update system, install git and restart raspberry
```
sudo apt update && sudo apt upgrade -y
sudo apt install git -y
sudo apt install mc -y
sudo shutdown -r now
```

5) Install KIAUH
```
git clone https://github.com/dw-0/kiauh.git
```

6) In KIAUH install klipper and then exit kiauh
```
./kiauh/kiauh.sh
```

7) install modified klipper for original display (this repo)
```
rm -rf klipper
git clone https://github.com/Teapack89/klipper
```

7) FW compilation and flash
```
cd ~/klipper
make menuconfig
```

![klipperFW_menuconfig](klipperFW_menuconfig.JPG)

Then rename klipper.bin to something else (like klipper08012025.bin and put on card formated FAT32 4096bytes). Put card to printer, power-on, wait 1min and turn off.

8) Install moonraker and then fluidd
```
cd ..
./kiauh/kiauh.sh
```

9) restart raspberry
```
sudo shutdown -r now
```

10) make mcu raspberry
```
cd ~/klipper/
make clean
make menuconfig
```

Change to linux process, exit and save

```
sudo service klipper stop
make flash
sudo service klipper start
sudo usermod -a -G tty pi
```

11) Restart raspberry
```
sudo shutdown -r now
```

12) Copy default config for Ender6 and add to printer.cfg to make display work:
```
[t5uid1]
firmware: dgus_reloaded
machine_name: Ender 6
volume: 0
brightness: 100
z_min: 0
z_max: 400
```

13) Enable Bltouch, adxl,... in printer.cfg (optional)

Change position_endstop and position_max in stepper_x/stepper_y if you have non original extruder and extruder could collide somewhere.

ADXL:
```
[mcu rpi]
serial: /tmp/klipper_host_mcu

[adxl345]
cs_pin: rpi:None

[resonance_tester]
accel_chip: adxl345
probe_points:
    130, 130, 20

[input_shaper]
shaper_freq_x: 67.2
shaper_type_x: ei
shaper_freq_y: 72.8
shaper_type_y: mzv
```

Screws tilt adjust:
```
[screws_tilt_adjust]
horizontal_move_z: 5
screw1: 63,243
screw1_name: back left
screw2: 253,243
screw2_name: back right
screw3: 64,53
screw3_name: front left
screw4: 254,53
screw4_name: front right
```

If you have BLtouch, on lines with "# disable to use BLTouch" add #, delete # on lines with "# enable to use BLTouch".

Set X and Y Bltouch offsets:
https://www.youtube.com/watch?v=FKvPU2nwdts

Personaly i add also this in BLtouch section (original BLtouch, i thing 3Dtouch need different settings):
```
stow_on_each_sample: false
probe_with_touch_mode = true
```

14) Calibrate:

If use ADXL:
https://www.klipper3d.org/Measuring_Resonances.html#measuring-the-resonances_1

PID heater extruder:
```
PID_CALIBRATE HEATER=extruder TARGET=200
SAVE_CONFIG
```

PID heater bed
```
PID_CALIBRATE HEATER=heater_bed TARGET=60
SAVE_CONFIG
```

Calibrate rotation_distance if you have not original extruder:
https://www.service-uplink.de/esteps_cal/calculator.php

Calibrate Z offset if use BLtouch:
https://www.kevink.org/calibrate-z-offset-with-klipper/

**DONE :)**

---------------------------------------------------------------
From:
https://github.com/Klipper3d/klipper

with update from:
https://github.com/Desuuuu/klipper

Using this script (found in Desuuuu->Issues - user xdadrm):
```
#!/bin/bash
if [ ! -d klipper/.git ]; then
  git clone https://github.com/Klipper3d/klipper.git klipper
else
  echo "Klipper exists .."
fi

pushd klipper 


if ( git branch | grep desuuuu-ender6 -q ); then
  git switch desuuuu-ender6
else
  git branch desuuuu-ender6  
  git checkout desuuuu-ender6
fi

GIT_EDITOR=/bin/true git pull https://github.com/Klipper3d/klipper.git
GIT_EDITOR=/bin/true git pull -q -X ours --no-rebase https://github.com/Desuuuu/klipper.git

popd
```
