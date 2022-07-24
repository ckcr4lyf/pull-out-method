# What is the pull out method?

It is a udev rule to shutdown your computer when a USB device is pulled out.

# Why?

I encrypt my laptop with LUKS, but if I ever use it in public, someone could physically steal it from me while I am using it, and it is in a decrpyted state.

With the pull out method, I can tie a usb key to my wrist (with a certain amount of leeway, say 30 centimeters). Then, if someone snatches my laptop away from me, the USB key will get removed, and the udev rule will force the computer to shutdown immediately, so the other party cannot access my data.

[This is how Ross Ulbricht's laptop was seized by the FBI](https://www.businessinsider.com/the-arrest-of-silk-road-mastermind-ross-ulbricht-2015-1). Unfortunately he didn't have such a system in place, so all the contents of the laptop became availale to the agents.

# How?

First we need a USB key to use for this, one with a hoop to tie a lanyard or keychain. The hoop should be of strong quality, a metallic one part of the drive is best.

Plug the drive into your machine and use `lsblk` to get its SCSI ID.

E.g. in this case, it is `/dev/sdb`.

```sh
$ lsblk
NAME                    MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                       8:0    1     0B  0 disk
sdb                       8:16   1  28.7G  0 disk
|-sdb1                    8:17   1   790M  0 part
`-sdb2                    8:18   1    74M  0 part
```

Then, you can grab the device's serial ID using:

```
udevadm info -a -n /dev/sdb
```

Replace `/dev/sdb` with the SCSI ID as per your system. Among the different blocks, look for the one with `SUBSYSTEMS=="usb"` and `DRIVERS="usb"`.

Example:


```
looking at parent device '/devices/pci0000:00/0000:00:14.0/usb2/2-2':
    KERNELS=="2-2"
    SUBSYSTEMS=="usb"
    DRIVERS=="usb"
    [...ommitted...]
    ATTRS{product}==" SanDisk 3.2Gen1"
    ATTRS{serial}=="0401356[...ommitted...]"
```

Next, copy the shutdown script somewhere, I picked `~/bin/shutdown.sh`. Remember to make it executable:


```
chmod +x ~/bin/shutdown.sh
```

Then, copy the file `80-local.rules` into `/etc/udev/rules.d/` (changing the deviceId and path to shutdown script as required).

Finally, reload the udev rules with:

```
udevadm control --reload
```

Nothing should happen if you unplug other devices, or even _plug in_ the "registered" device. But if you remove it, the system should shut down.


# Tips

TODO (add stuff about physical considerations, e.g. tensile strength and elasticity of cord).

# References

Kenlon, S. (2018) _An introduction to Udev: The Linux subsystem for managing device events_ . Retrieved 24 July 2022 from https://opensource.com/article/18/11/udev.