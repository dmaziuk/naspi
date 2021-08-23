# raspberry pi zfs nas

breadboard build

## Note

Version 1 was built on Radxa SATA HAT. I fried that and while I was waiting for the new batch of HATs from Radxa, CM4 came out. This is CM4 build, switch to radxa branch for details on v.1.

## parts

### zpool

ZFS is very flexible when it comes to disk configuration, I opted for 3x spinnning rust HDDs in `raidz1` (similar to RAID-5) set and an SSD for caches 
(and other things like system swap). This doesn't have a 2nd SSD to mirror `SLOG`, but I'm not entirely sure how critical that is nowadays, especially 
on lightly used home NAS. Hence 4-port SATA controller and 4-port disk cage.

  * CM4 with i/o board. I have the no-WiFi, no eMMC version. ZFS will use all the RAM it can get to, so more is better.
  * PCIe x1 SATA controller w/ Marvel chip (as that's well supported by Linux): https://www.amazon.com/gp/product/B00AZ9T3OU/
  * IcyDock MB074SP-1B: https://www.amazon.com/gp/product/B00LICRE8A/ -- it's a 4-disk cage with hot-swap backplane and a nice big fan for plenty of cooling w/o too much noise.
  * SSD caddy for the above.
  * 80+W 12V PSU w/ barrel jack, like htis one: https://www.amazon.com/gp/product/B07PWZQ33N/. 
  * IcyDock's backplane has 2x SATA power connectors, you'll need to feed it from CM4 i/o board's 4-pin "floppy" connector. These: https://www.amazon.com/gp/product/B076Q19PZS/ and https://www.amazon.com/gp/product/B0086OGN9E/ work.
  * SATA cables.
  * PCIe cable: https://www.amazon.com/gp/product/B086V3JNSG/

## software and setup

OS: ubuntu-20.04 LTS server (64-bit ARM). I plan to switch to Alpine someday but I'll have to figure out their kernel-building setup (or wait till they build one w/ all the required modules),
in the meantime Ubuntu kernel recognizes the Marvel chip out of the box and may have the SATA modules in one of the "universe" packages. I built my own anyway, 
use e.g. https://askubuntu.com/questions/1238261/customizing-the-kernel-arm64-using-ubuntu-20-04-lts-on-a-raspberry-pi-4#1242267 as a reference if not familiar 
with debian/rules.

Before build, edit `debian.raspi/config/config.common.ubuntu` and set `CONFIG_SATA_AHCI=m`. This will turn on some other option that's hiding in the maze
of config files, just start the build and wait for it to pop up and hit enter.

After installing the kernel, `sudo apt install zfsutils-linux` and reboot.

### pics or it didn't happen

breadboard build: `bbb3.jpg`

