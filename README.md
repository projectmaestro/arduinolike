# arduinolike
Reducing the pain in migrating from Arduino to nrf Connect for nrf52840.

This is a library of 'Arduino like' functions that provide a starting point to migrate from Arduino to nrf Connect, such as digitalRead, digitalWrite, analogRead, etc.

* Target - nrf52840 on an Adafruit Itsy Bitsy board. This is a fairly bare board with two buttons, a colour changing LED and a 4Mbit flash memory.*
* Using Segger Embedded Studio (SES)
* Segger J-Link

I last coded microcontrollers on ARM M0 cores using the LPC-Expresso toolkit. I needed to migrate from Arduino to Nordic nrf Connect but have found the process incredibly confusing. This public repo is to share code & advice for anyone similarly suffering.

## Why did I need to migrate? ##

I'm making a product for the general public (rather than technical users) and it needs to be sealed in a box. There will be regular firmware updates so it must be possible to do updates.

Arduino / Adafruit / Bluefruit has a mobile all called Bluefruit Connect. There claims to be a way to do over the air updates (OTA DFU - Over The Air Device Firmware Updates); however this doesn't work for the nrf52840 (as of January 2022). I put messages on forums and don't have a solution.

Before I realised this limitation I'd already created most of my firmware on Arduino. This was working well and could do everything I needed.

The only solution was to take a deep breath and open up the nrf Connect SDK - it's been a much bigger task than I expected.

## Problem 1 - Which SDK? ##

Nordic currently have two SDKs running in parallel. One called something like NRF51 SDK, and another called nrf Connect.
* NRF51 uses a binary 'softdevice' that's like a black box providing functions for the application code. I think it's based on FreeRTOS (I have no more knowledge than this). The Arduino nrf seems to be based on this SDK.
* nrf Connect doesn't have a soft device; various projects are built and linked together by the SDK. nrf Connect uses an RTOS called Zephyr.

I opted for nrf Connect as this supports nrf53 devices. NRF51/52 won't support nrf53 so seemed like a dead end to start any new projects with it.

## Problem 2 - Making any sense of what's going on

These are hard won lessons!

### RTOS / Zephyr

Zephyr is a RTOS (Real Time Operating System) that's a set of modules that provide a wrapper and services to your application, including services such as booting. The Zephyr modules will be compiled and linked with your nrf Connect project.

You choose the Zephyr (and other) modules using the root/prj.conf file . You can also opt to include nrfx and nrf modules using the prj.conf file - more on this later.

Once you've changed the prj.conf file, go to Project -> Run CMake... . This remakes the project and includes all the modules you've selected with prj.conf.

When searching for help (you'll do a lot of this) you'll see reference to sdk_config.h . This is a red herring; it seems to be for an earlier or different SDK. prj.conf is the new way to configure projects; sdk_config.h is no longer used.

Zephyr provides a number of services, such as Bluetooth and a bootloader (mcumgr) that can handle over the air updates. Zephyr also provides a number of low level services such as PWM - you may not wish to use these though.


### nrfx / nrf

nrfx is a driver layer provided by Nordic. nrf (HAL) is a hardware abstraction layer provided by Nordic. Zephyr uses the nrfx drivers to manipulate the hardware.

Zephyr -> nrfx -> nrf -> Device registers

However, Zephyr has limitations. For example, I want to run a 32kHz PWM output with 8 bit resolution. Off-the-shelf Zephyr makes it simpler to start a PWM instance but runs into problems if you want to push the hardware beyond what was intended in Zephyr.

There are various places where Zephyr doesn't have modules that control the nrf drivers, and various other clashes that mean you can't use certain Zephyr modules.

You will likely find that Zephyr doesn't let you do what you need to do with the hardware in some circumstances. For me, this includes PWM & ADC. In that case, you will need to deliberately exclude the relevant Zephyr module from the build and in the nrfx and/or nrf module instead. 

## Troubleshooting

### Segger Embedded Studio RTT Unreliable / Flakey when mcuboot enabled

I lost (a day) two days to this one. The RTT just wouldn't connect reliably. Turns out there's a define called _SEGGER_RTT (_) in zephyr/zephyr.map and this is not passed reliably to the J-Link Viewer which presumably sits behind the emStudio Debug Terminal.

I went on a wild goose chase via changing zephyr/merged.hex as the active project, the debug terminal didn't work, but if the zephyr/zephyr.elf project was active, the debug terminal worked. My assumption is that the RTT define doesn't get outside the main solution so when the debug terminal starts from the zepyyr.hex project, the define isn't defined - so the viewer struggles to find the control block.

It eventually turned out that the problem at the storage partition being in different places between the DTS system & overlays, and the Partition Manager that mcuboot uses put it in a different place. Why there are two different ways to define the partition in the same build system, I have no idea.

The eventual solution was to create a file called pm_static.yml in the project root and set the storage partition to the same location as in the zephyr.dts.

settings_storage:
  address: 0xf8000
  size: 0x8000



