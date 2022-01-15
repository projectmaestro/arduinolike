# arduinolike
Reducing the pain in migrating from Arduino to nrf Connect for nrf52840

I last coded microcontrollers on ARM M0 cores using the LPC-Expresso toolkit. I needed to migrate from Arduino to Nordic nrf Connect but have found the process incredibly confusing. This public repo is to share code & advice for anyone similarly suffering.

## Why did I need to migrate? ##

I'm making a product for the general public (rather than technical users) and it needs to be sealed in a box. There will be regular firmware updates so it must be possible to do updates.

Arduino / Adafruit / Bluefruit has a mobile all called Bluefruit Connect. There claims to be a way to do over the air updates (OTA DFU - Over The Air Device Firmware Updates); however this doesn't work for the nrf52840 (as of January 2022). I put messages on forums and don't have a solution.

Before I realised this limitation I'd already created most of my firmware on Arduino. This was working well and could do everything I needed.

The only solution was to take a deep breath and open up the nrf Connect SDK - it's been a much bigger task than I expected.

## Problem 1 - Which SDK? ##




