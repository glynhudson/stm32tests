# stm32tests

this repo is just some notes gathered whilst starting out with an STM32 MCU, in particular a nucleo-f303re board.

For notes about any particular project check for a README.md in that project's folder.

## Getting started

## Using Make

To compile and flash the firmware to a USB connected device and then view the serial output on a Linux box all that is required is the arm toolchain and a serial console like Minicom, both can be installed via apt-get

```
sudo apt-get update
sudo apt-get install -y gcc-arm-none-eabi minicom
```

once this repo is cloned or downloaded/extracted a project can be compiled by just running `make` from the chosen project folder

```
git clone https://github.com/stm32oem/stm32tests.git
cd stm32tests/emonTxshield
make
```

to flash the firmware to a USB connected nucleo board using the on-board STLink v2.1 programmer/debugger, the `.bin` file found it the `build` folder that was generated by `make` can be copied to the "NODE_F303RE" USB storage device. The STLink v2.1 makes flashing very easy by presenting the nucleo board as a USB mass storage device (512Kb in the case of the F303RE) and any `.bin` file dropped there will be flashed to the target STM32 MCU by the STLink v2.1 automatiaclly.

```
cp build/emonTxshield.bin /media/pi/NODE_F303RE
```

The path to the "NODE_F303RE" may vary depending on the host OS in use, here Raspbian Stretch is being used an a Raspberry Pi 3 and it was autodetected and mounted by the OS.

To then view the serial output Minicom can be used

```
minicom -F -b115200 -D/dev/ttyACM0
```
if the serial output scrolls off the right hand side of the page, the "Add carriage return" setting in Minicom can be set with `ctrl-a` then `u`. to exit Minicom use `ctrl-a` then `x` and `enter` or to bring up a menu of options `ctrl-a` then `z`.

to make life a little easier an alias can be set up with
```
printf "alias nucleo='minicom  -F -b115200 -D/dev/ttyACM0'\n" >> ~/.bash_aliases
source ~/.bashrc
```
so that just typing `nucleo` will open Minicom with the correct path and baud eg
```
pi@raspberrypi:~ $ nucleo

Welcome to minicom 2.7

OPTIONS: I18n
Compiled on Apr 22 2017, 09:14:19.
Port /dev/ttyACM0, 15:04:04

Press CTRL-A Z for help on special keys

0: Vrms: 1150.38, Irms: 39.15, Papp: 45038.84, Preal: 290.31, PF: 0.006
1: Vrms: 1150.38, Irms: 2047.88, Papp: 2355847.50, Preal: 3066.12, PF: 0.001
2: Vrms: 1150.38, Irms: 2047.88, Papp: 2355848.25, Preal: 3066.12, PF: 0.001
3: Vrms: 1150.37, Irms: 2047.88, Papp: 2355817.00, Preal: 1267.52, PF: 0.001
1: Vrms: 1150.94, Irms: 2047.88, Papp: 2356991.25, Preal: 1878.74, PF: 0.001
2: Vrms: 1150.94, Irms: 2047.88, Papp: 2356991.75, Preal: 1878.95, PF: 0.001
3: Vrms: 1150.94, Irms: 2047.88, Papp: 2356979.75, Preal: 2147.80, PF: 0.001
0: Vrms: 1150.94, Irms: 39.14, Papp: 45043.89, Preal: 240.61, PF: 0.005
```
Likewise we can add a bash function to help with uploading too with
```
printf 'function flash() { cp "build/${PWD##*/}.bin" "/media/${USER}/NODE_F303RE" ; }\n' >> ~/.bash_aliases
source ~/.bashrc
```
The line may need editing for some setups, it currently uses the current working directory to workout the `.bin` filename and assumes the nucleo is mounted in the current users media folder.

Now just typing `flash` will copy the `.bin` file to the nucleo board. So to compile, upload and open minicom in one move.
```
make && flash && nucleo
```
this has an added advantage that it actually catches the resetting of the nucleo so you see the full startup info of the newly flashed FW
```
3: Vrms: 1150.20, Irms: 87.93, Papp: 101134.68, Preal: 2293.79, PF: 0.023
0: Vrms: 1150.22, Irms: 8.00, Papp: 9203.00, Preal: -14.08, PF: -0.002
1: Vrms: 1150.22, Irms: 87.93, Papp: 101135.70
emonTxshield Demo 1.2
Patch PA0 through to PB14 for V!!!
CPU temp: 30C, Vdda: 3306mV
CPU temp: 30C, Vdda: 3304mV
1: Vrms: 1147.66, Irms: 87.92, Papp: 100899.40, Preal: 1183.72, PF: 0.012
2: Vrms: 1147.66, Irms: 87.93, Papp: 100910.83, Preal: 1183.76, PF: 0.012
3: Vrms: 1147.65, Irms: 87.92, Papp: 100906.77, Preal: 1373.21, PF: 0.014
0: Vrms: 1147.66, Irms: 8.91, Papp: 10231.19, Preal: -10.75, PF: -0.001
```


### For the less patient (including me) here is an all in one block of code

```
sudo apt-get update
sudo apt-get install -y gcc-arm-none-eabi minicom
printf "alias nucleo='minicom -F -b115200 -D/dev/ttyACM0'\n" >> ~/.bash_aliases
printf 'function flash() { cp "build/${PWD##*/}.bin" "/media/${USER}/NODE_F303RE" ; }\n' >> ~/.bash_aliases
source ~/.bashrc
git clone https://github.com/stm32oem/stm32tests.git
cd stm32tests/emonTxshield
make
flash
nucleo
```
[this works on a Pi running Raspbian Stretch, you may get different results if (for example) the `git` and `make` packages are not already installed. Omit the first lines if tools already installed or the last lines if nucleo device not yet connected]



## Alternatively to use platformIO

[Note - not currently working on a RPi see the [STM32 PlatformIO](https://community.openenergymonitor.org/t/stm32-platformio/7015?u=pb66) thread on OEM forum.

Install platformio http://docs.platformio.org/en/latest/installation.html#local-download-mac-linux-windows

To download this repo, compile and upload with platformIO:

```
git clone https://github.com/stm32oem/stm32tests.git
cd stm32tests/emonTxshield
pio run -t upload
```

To view serial output:

`pio device monitor`


