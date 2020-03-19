# ESP8266_RTOS_SDK #

----------

ESP8266 SDK based on FreeRTOS.

## Note ##

APIs of "ESP8266_RTOS_SDK" are same as "ESP8266_NONOS_SDK"

More details in "Wiki" !

## Requrements ##

You can use both xcc and gcc to compile your project, gcc is recommended.
For gcc, please refer to [esp-open-sdk](https://github.com/pfalcon/esp-open-sdk).


## Compile ##

Clone ESP8266_RTOS_SDK, e.g., to ~/ESP8266_RTOS_SDK.

    $git clone https://github.com/espressif/ESP8266_RTOS_SDK.git

Modify gen_misc.sh or gen_misc.bat:
For Linux:

    $export SDK_PATH=~/ESP8266_RTOS_SDK
    $export BIN_PATH=~/ESP8266_BIN

For Windows:

    set SDK_PATH=/c/ESP8266_RTOS_SDK
    set BIN_PATH=/c/ESP8266_BIN

ESP8266_RTOS_SDK/examples/project_template is a project template, you can copy this to anywhere, e.g., to ~/workspace/project_template.

Generate bin:
For Linux:

    ./gen_misc.sh

For Windows:

    gen_misc.bat

Just follow the tips and steps.

## Download ##

eagle.app.v6.flash.bin, downloads to flash 0x00000

eagle.app.v6.irom0text.bin, downloads to flash 0x40000

blank.bin, downloads to flash 0x7E000

# Amateur archaeology (i.e. setting up Espressif RTOS SDK 1.5.0)

Instructions for getting the RTOS SDK running with Ubik projects. Tested on Linux (Debian 11 bullseye), please adapt to your environment.

## Getting toolchain 4.8.5

We need version 4.8.5 of espressif's gcc toolchain. The "newer" version 5.2 doesn't compile Espressifs v1.x SDK.

### Custom built Espressif toolchain

Some people have kindly automated the building of Espressif toolchain and SDK. This will build NONOS SDK v2.1 which we won't be using. The toolchain is what we need.

We need to patch crosstools configure.ac because it doesn't support bash v5 (braindead version checking). Abbreviated instructions:

```
$ sudo apt install make unrar-free autoconf automake libtool gcc g++ gperf flex bison texinfo gawk ncurses-dev libexpat-dev python-dev python sed git unzip bash help2man wget bzip2 libtool-bin
$ mkdir -p ~/local/lib/esp8266
$ cd ~/local/lib/esp8266
$ git clone --recursive https://github.com/pfalcon/esp-open-sdk.git
$ cd esp-open-sdk
$ sed -i -r 's/(GNU bash, version.*\|4)/\1\|5/' crosstool-NG/configure.ac
$ make STANDALONE=n
$ export PATH="$HOME/local/lib/esp8266/esp-open-sdk/xtensa-lx106-elf/bin:$PATH"
```

Add last command to .bash_profile or your tmuxinator's `pre_window` command to make it persistant.

### Pre-built toolchain from Espressif

NB! The pre-built toolchain is missing some libraries required by 1.x SDK, specifically `libhal.a`. So avoid this and build our own (see previous section).

Pre-built binaries are linked here: https://github.com/espressif/ESP8266_RTOS_SDK

Download and installation instructions:

```
$ mkdir -p ~/local/lib/esp8266 && cd ~/local/lib/esp8266
$ wget https://dl.espressif.com/dl/xtensa-lx106-elf-linux64-1.22.0-88-gde0bdc1-4.8.5.tar.gz
$ tar -zxf xtensa-lx106-elf-linux64-1.22.0-88-gde0bdc1-4.8.5.tar.gz
$ mv xtensa-lx106-elf xtensa-lx106-elf-linux64-1.22.0-88-gde0bdc1-4.8.5
$ export PATH="$HOME/local/lib/esp8266/xtensa-lx106-elf-linux64-1.22.0-88-gde0bdc1-4.8.5/bin:$PATH"
```

Add last command to .bash_profile or your tmuxinator's `pre_window` command to make it persistant.

## Getting RTOS SDK 1.5.0

A fork with required modifications is available in Ubik github repo https://github.com/UBIKsolutionsOU/ESP8266_RTOS_SDK

```
$ mkdir -p ~/projects/ubik/acov/lib/src/ && cd ~/projects/ubik/acov/lib/src/
$ git clone https://github.com/UBIKsolutionsOU/ESP8266_RTOS_SDK
$ cd ESP8266_RTOS_SDK
$ git checkout -b 1.5.x origin/1.5.x
$ git submodule update --init
$ export SDK_PATH="$HOME/projects/ubik/acov/lib/src/ESP8266_RTOS_SDK"
```

Add last command to .bash_profile or your tmuxinator's `pre_window` command to make it persistant.

### Building RTOS SDK 1.5.0 ###

Building the SDK itself I haven't tried. There are pre-built binaries in `lib` in SDK root.

If you need to build third party libraries, e.g. lwip:

```
$ sudo apt install make unrar-free autoconf automake libtool gcc g++ gperf flex bison texinfo gawk ncurses-dev libexpat-dev python-dev python sed git unzip bash help2man wget bzip2 libtool-bin
$ cd ~/projects/ubik/acov/lib/src/ESP8266_RTOS_SDK/third_party
$ ./make_lib.sh lwip
```

Resulting library is copied to `lib` in SDK root.

## Getting esptool from apt

This is the main tool to upload/download/investigate the ESP chip connected to (virtual) serial port. On Debian, installing is easy:

```
$ sudo apt install esptool
```

## Getting esptool from pip

Previous section should cover your esptool needs. If the Debian package doesn't work or you need to run the deprecated esptool.py (which requires python-serial), then read on.

Debian 11 (bullseye) has ditched pyserial packages for 2.x (`python-serial`) and only pyserial for 3.x is easily available (`python3-serial`). This doesn't help :)

The solution is to create a python virtualenv and install the outdated 2.x libraries with pip.

```
$ sudo apt install python-pip
$ mkdir -p ~/projects/ubik/acov/lib/ && cd ~/projects/ubik/acov/lib
$ virtualenv venv
$ pip install pyserial esptool
```

Whenever working on projects with this hacked esptool, activate the virtualenv before proceeding

```
$ . ~/projects/ubik/acov/venv/bin/activate
```

Add this command to your tmuxinator's `pre_window` command to make it persistant.
