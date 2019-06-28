# cmake-linux-pi
Example for a cmake based cross compilation environment running gcc on linux and producing code for the raspberry pi.

It contains content copied from a number of places. Many thanks to the authors. This would not be possible without them.
## Overview
This repository contains instructions and examples to enable you to set up a gcc based cross compilation running on an x86_64 Linux system and producing executables targeted at the Raspberry Pi bcm2708 platform. It uses CMake as the build environment.
## Stage 1 - Preparing the sysroot and tools
Firstly prepare a directory on the host system containing a sysroot from the target system and a toolchain provided by the raspberrypi/tools project on github.
1. Choose a location. I use a directory called raspi in my home directory.

 ```shell
 mkdir ~/raspi
 ```
2. Clone the raspberrypi/tools repository into the directory.

 ```shell
 git clone https://github.com/raspberrypi/tools.git ~/raspi/tools
 ```
3. Make the sysroot.

 ```shell
 # Substitute your username on the pi and its hostname or ip address in the commands below.
 mkdir ~/raspi/sysroot ~/raspi/sysroot/usr ~/raspi/sysroot/opt
 rsync -avz pi@raspberrypi_ip:/lib sysroot
 rsync -avz pi@raspberrypi_ip:/usr/include sysroot/usr
 rsync -avz pi@raspberrypi_ip:/usr/lib sysroot/usr
 rsync -avz pi@raspberrypi_ip:/opt/vc sysroot/opt
 ```
4. Make all the symbolic links in the sysroot relative. Again I pull a script from github to do this.

  ```shell
  wget -P ~/raspi https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
  chmod +x ~/raspi/sysroot-relativelinks.py
  ~/raspi/sysroot-relativelinks.py ~/raspi/sysroot
  ```
5. The sysroot and toolchain are now ready for use. The compiler can be tested as follows.

 ```shell
 echo "int main() {}" | /<path to your tools>/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf-gcc --sysroot=/<path to your sysroot>/sysroot -xc -
 file a.out
 ```
 The output of the file should show the file as for an arm system.


## Stage 2 - Adding cross compilation to a CMake source tree.
CMake provides an excellent environment for cross compilation. All the information required can be placed in a single file in the root of the source tree.
1. Copy the example-cross-compile.cmake file provided, into the root of your CMake source tree.
2. Edit the local file to point at your sysroot and tools directories.
3. I strongly suggest you make a clean out of source tree build directory and run your preferred cmake command from there. See below if you want to use cmake-gui/
4. You need to tell CMake where the toolchain file is. This is done by passing it the command line parameter "-DCMAKE_TOOLCHAIN_FILE=example-cross-compile.cmake". For example from your new build directory.

 ```shell
 ccmake -DCMAKE_TOOLCHAIN_FILE=example-cross-compile.cmake <path to your source directory>
 ```
 You can ignore any warnings about unused manual parameters.
 5. If you want to use cmake-gui. Just run the command from your new build directory with no parameters. When the gui launches you need to select your source tree. When you start configuring you will be asked to specify a generator. Select your preferred generator and then in the options select "Specify toolchain file for cross-compiling". When asked for the file name browse to example-cross-compile.cmake in your source tree. Thats it!

## Trouble shooting
The most common problem is not setting up the sysroot properly. Have a look at the sysroot-contents file in this directory to see the contents and structure of the sysroot.
