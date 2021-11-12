---
title: Troubleshooting
parent: Software
grand_parent: Generation 3
nav_order: 4
---


# Troubleshooting

The Panels system is somewhat involved, and at present is under-documented. Therefore it is not surprising that as people start running this new system they will run in to various troubles. We will try to list the most common problems here and address them as they come up.

## Trouble using multiple I2C busses

Note that it will only be possible to take advantage of the 4 I2C busses with an updated arena board--otherwise the display can only use bus 1. If you have an updated board and you find that only BUS0 is active, and if you have reprogrammed all of the panels w/ the updated firmware according to the [[Quick Start]], and the panels have been addressed properly, and you are having the issue that only Bus 0 (ID = '01') is active, then you should check to see that the jumpers on the arena base are set correctly.  The correct configuration of the jumpers are as follows:

If you have version 1 of the arena base (upper right corner reads "12 panel ring with 4 bus v1.0"): You should use eight jumpers to connect pin 1 and 2 of W0CLl, W1CL, W2CL, W3CL, W0DA, W1DA, W2DA, W3DA.

If you have the "Rev B" arena base (upper right corner reads "JF-MR-FY0001-RevB"): You should use 6 jumpers to connect pin SDAxx with pin SDAx (X = 1, 2, 3) in JP16-18 and pin SCLxx with pin SCLx (X=1,2,3) in JP13-15, etc.

## Error when trying to show bus number

This error could be due to several problems, but the most likely is that the panels must be reprogrammed with a newer version of the firmware, that contains the instructions to respond to this command. If you are already going to the trouble or re-programming panels, make sure to install the bootloader as well, this will save time when updating later.

## Matlab error Open PControl without serial port connection after reboot/killing matlab

This error can be fixed by simply turning off the controller, unplugging the controller USB and then replugging it and switching the power on. If this does not work in a system that was previously working, check the device manager for the COM port that disappears when unplugging the controller, be sure that is the one entered in the window prompt when starting PControl. If this still does not work, try disabling and enabling the device or reinstalling the FTDI usb serial drivers.

## Strange behavior of patterns on a nonstandard panel setup

This error has been noticed when trying to use a subset of panels in a normal setup, or another such modification to using a 'full' set of panels in an arena. The issue lies with a panel map with nonconsecutive numbered panel addresses. Currently a limitation of the pattern data generation is data should be provided to all panels from 1 to the maximum panel ID in the map no matter whether each panel exists or not. For example, the following map will give odd behavior:

```matlab
1     12     8     4    11     7     3
13    24    20    16    23    19    15
25    36    32    28    35    31    27
```

due to skips in numbering from 1 to 36. This problem can be overcome by using a map of the same dimensions, with consecutive numbering:

```matlab
1      7     5     3     6     4     2
8     14    12    10    13    11     9
15    21    19    17    20    18    16
```

This issue will be corrected in future firmware releases.

# Known Issues

1. If the address of a panel in an arena has been set to zero, we cannot change the panel address to an nonzero address. That is why we set a nonzero address, 127, as a default address in panel.eep.
1. If you have trouble running `get_SD_drive.m`, you need to check: 1) make sure the hard-coded SD drive letter is the correct drive; 2) path of `fsutil.exe` is included in the environment variable PATH.
1. When writing experimental script, `Panel_com('set_pattern_id', i)` should be called before `Panel_com('set_position', [m n])`. If not, the system will freeze.
1. From the PControl UI it is possible to get the controller to hang if you send many updates of pattern position or gain/bias in a short period of time. This is also possible in a script. The simple remedy is to place stop/start commands before and after the gain/bias or position change. We have done this for the PControl GUI (updated in code v1.1), but without being aware of this you might crash the controller from a script. A more elegant solution would be to handle this on the controller side, and we might implement this in the future.
1. If using an SD card that has not been formatted after many writes, the onset time of position functions may be affected. If you notice there is a large delay to the start of a position or velocity function, try formatting the SD card (fat32) for better performance. Alternatively, adding a several millisecond pause after the set position function command can also fix this issue (for minor offsets).

For more hardware oriented troubleshooting, see the [panels-hardware troubleshooting guide]({{site.baseurl}}/Generation%203/Hardware/docs/troubleshooting.html).
