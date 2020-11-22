# raspberry pi zfs nas

breadboard build

## background

Pi 4 has a PCIe bus but it takes a soldering iron to get to it: https://hackaday.com/2020/07/01/adding-pcie-to-your-raspberry-pi-4-the-easier-way/

Option 1 for adding SATA ports to the Pi is to do the above and find a SATA controller that works. This will, of course, void your warranty.

Option 2: Radxa: https://wiki.radxa.com makes a Pi clone with PCIe bus (apparently) exposed, and a 5-port SATA HAT for it.

Option 3: Radxa also makes a HAT for Pi 4: https://wiki.radxa.com/Dual_Quad_SATA_HAT -- this one hooks the Pi's 2 USB 3.1 ports to a pair of JMicron port mulripliers turning them into 4 SATA ports. Having to go through port multipliers is the downside (*) but there is no soldering required and how much performance do you really need in a home NAS.

This is the option 3 build.

*) PCIex1 is rated to 2.5Gbps one-way, same as half of the USB 3.1's 5Gbps each disk would be getting over the port multiplier (best case scenario), which makes the whole performance thing is somewhat academic. There are more practical problems with this setup thout, see below.

## lessons learned

Go for option 1 or wait for a Pi with accessible PCIe bus: it's too obviously useful to keep locked down, so it's likely that Raspberry Pi Foundation will soon release one.

  * With USB to SATA HAT your drives show up as USB mass storage devices. Those are detected and spun up fairly late in the boot sequence, ZFS pool can't be imported when OS startup scripts expect. This also means you can't use these for e.g. swap as they're not available when swap is mounted, and so on. None of that should be happening with a proper SATA controller. 
  * Radxa's HAT is not a pure USB device, it needs a few GPIO pins connected to be properly recognized by the OS. It also needs Radxa software that works on both kinds of Unix: debian and ubuntu(*). A supported SATA controller, OTOH, may even work on BSD where ZFS is a first-class citizen.
  * Radxa's HAT is optimised for their own NAS product and not for DIY use. That causes some minor headaches, like not much room for the CPU fan when stacked (nor place to plug it in), or the need for SATA gender benders or M-F data cables (the HAT works as a backplane for 4x2.5" drives in Radxa NAS and so has the "backplane" connectors).

*)https://www.youtube.com/watch?v=vS-zEH8YmiM

## parts

  * Pi 4. I have the 4GB version but ZFS will use all the RAM it can get to, so... more is better.
  * Radxa dual-quad SATA HAT: https://wiki.radxa.com/Dual_Quad_SATA_HAT.
  * PicoPSU-80: https://www.amazon.com/gp/product/B005TWE5E6/ and an 80+W 12V PSU for it. 
  * According to Radxa you can power the whole thing through the HAT but I have the HDDs on the SATA power cable: https://www.amazon.com/gp/product/B0086OGN9E and the HAT and Pi on the 4-pin molex HDD to floppy cable.
  * 4x SATA gender benders: https://www.amazon.com/gp/product/B00S6HTVGI/ or M-F SATA data cables (keep in mind that the ones with sides sticking out will need to be filed down), or the 7+15 M-F cables (and power the disks through the HAT).
  * For the breadboard build I used a 4-disc caddy from my junk pile, for the final version I'm considering the IcyDock MB074SP-1B with hot-swap backplane.
  * I might get the ATX HAT for the final product if it looks like I can get it to play nice with the Radxa HAT: https://www.tindie.com/products/tomtibbetts/mini-atx-psu-cool-kit-for-raspberry-pi/

## software and setup

OS: ubuntu-20.04.1 server. I would much prefer Alpine but as of the time of this writing they have not built ZFS modules for `rpi4` kernel, so... I didn't get to see if Radxa stuff will even work with `clang` and `musl`. It may not. Ubuntu comes with ZFS modules and Radxa install works out of the box -- just follow the instructions and you can have your zpool up and running in no time. It won't survive the reboot, though.

### files

These are the result of a prolonged fight with the idiocy that passes for init in mainstream linux distros these days. I might trim them down at some point, but as-is, they work (for me).

  * `20-usbhd.rules` goes into `/etc/udev/rules.d` and handles "plugging in" the drives. Run `udevadm info --attribute-walk --path=$(udevadm info --query=path --name=/dev/sda)` for each drive and adjust `ATTRS` to taste.
  * `sd?-up.service` are custom "services" started by the above `udev` rules. It doesn't really matter what they do, they provide dependencies for `zfs-import`.
  * `zfs-*.service` go into `/etc/systemd/system` (`systemctl edit --full`) to completely override the stock service files. 
    * The default setup allows for root and boot on ZFS, for which ZFS has to be available early in the boot process, which we can't do because USB. We have to remove a couple `Before` dependencies from `zfs-mount` script to break that.
    * `zfs-import` scripts: add dependencies on the USB drives being up. It should probably work with dependencies on `dev-sd?.device`, I ended up with the `sd?-up` "services" instead (but may revisit that sometime).

### zpool

With 4 SATA ports, the "bang per buck" option is a `raidz1` on 3 spinning rust drives and a smallest SSD you can find for caches. You won't have a 2nd SSD to mirror `SLOG`, but I'm not entirely sure how critical that is nowadays, especially on lightly used home NAS.

### pics or it didn't happen

breadboard build: `bbb1.jpg` and `bbb2.jpg`

