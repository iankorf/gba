GBA
===

Messing around with GBA development. I'd like to make chemotaxis on the GBA.
Maybe some kind of simple menu system to run some other simulations.

+ Development
	+ devkitARM
	+ libtonc
	+ devkitPro pacman installs them
+ Emulators
	+ mGBA https://mgba.io/downloads.html

## Install ##

https://devkitpro.org/wiki/devkitPro_pacman

```
wget https://apt.devkitpro.org/install-devkitpro-pacman
chmod +x ./install-devkitpro-pacman
sudo ./install-devkitpro-pacman
```

Also this

```
sudo dkp-pacman -S gba-dev
```

And this

```
export DEVKITPRO=/opt/devkitpro
export DEVKITARM=/opt/devkitpro/devkitARM
export DEVKITPPC=/opt/devkitpro/devkitPPC
PATH=$PATH:$DEVKITARM/bin:$DEVKITPRO/tools/bin
```



## Hardware ##

+ CPU
	+ 32-bit ARM7TDMI
	+ 16.8 MHz
+ Screen
	+ 40.8mm x 61.2mm
	+ ~60Hz
	+ 240x160 pixels
	+ 15-bit RGB color pixel mode
	+ 7-bit color text mode
+ Inputs
	+ 8-way control pad
	+ Buttons: A B L R Start Select
	+ Volume slider
	+ Power switch
	+ Serial I/O link cable
	+ Cartridge I/O
