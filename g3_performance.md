---
title: Performance
parent: Software
grand_parent: Generation 3
nav_order: 3
---


# Benchmark testing

- 48 panels in a 48 panel arena, grayscale = 1, with (463 fps) and w/o row compression (191 fps)
- 48 panels in a 48 panel arena, grayscale = 2, w/o row compression (108 fps)
- 48 panels in a 48 panel arena, grayscale = 3, with (332 fps) and w/o row compression (72 fps)
- 48 panels in a 48 panel arena, grayscale = 4, with (290 fps)

# Frame dumping rate test

When we use the frame dumping mode to stream frame data from PC to controller directly (serial port baud rate 921600), we found the frame rate can reach the benchmark value. ie, for gs = 1, the maximum frame rate is 190 and when gs = 3, the maximum is 70. When we set the rate higher than the maximum, using timer function in matlab, the panel will freeze.

We increased the serial port baud rate from 115200 to 460800, the maximum frame rate increased linearly. When we set the baud rate to 921600, the system froze if we let the system ran as fast as it can. The reason is the I2C set a bottleneck to the maximum frame rate at which the system can run. That is why we don't need to set a higher serial port baud rate although the maximum baud rate the XMega controller can reach is 1843200.

We also did the jitter testing for the frame dumping mode and the results are list as the following tables.

## Test 1

Grayscale = 1, no row compression,  sampling rate 10K

| Frame Rate| 10| 20| 30| 40| 50| 60| 70| 80| 90| 100| 110| 120|
| ------------------ | -------- | -------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | ------------ | ------- | -------- |
| Mean IFI           | 0.1000   | 0.0500   | 0.0330    | 0.0250    | 0.0200    | 0.0170    | 0.0140    | 0.0130    | 0.0110    | 0.0100      | 0.0090  | 0.0080   |
| Standard deviation | 0.2129e-3| 0.2098-3 | 0.1894e-3 | 0.2049e-3 | 0.3416e-3 | 0.2750e-3 | 0.2622e-3 | 0.3126e-3 | 0.1770e-3 | 0.2711e-3 | 0.2312e-3 | 0.3226e-3|
| Min IFI            | 0.0985   | 0.0487   | 0.0321| 0.0237| 0.0150| 0.0136| 0.0120| 0.0077| 0.0100| 0.0055| 0.0066| 0.0064|
| Max IFI            | 0.1008   | 0.0513| 0.0338| 0.0265| 0.0253| 0.0204| 0.0155| 0.0182| 0.0116| 0.0145| 0.0108| 0.0090|

## Test 2

Grayscale = 3, no row compression,  sampling rate 10K

| Frame Rate         | 10        | 20        | 30       | 40        | 50        | 60        |
| ------------------ | --------- | --------- | -------- | --------- | --------- | --------- |
| Mean IFI           | 0.1000    | 0.0500    |0.0330    | 0.0250    | 0.0200    | 0.0170    |
| Standard deviation | 0.3335e-3 | 0.2650e-3 |0.3247e-3 | 0.4414e-3 | 0.3226e-3 | 0.2213e-3 |
| Min IFI            | 0.0974    | 0.0475    | 0.0286   | 0.0167    | 0.0166    | 0.0164    |
| Max IFI            | 0.1028    | 0.0524    |0.0368    | 0.0332    | 0.0247    | 0.0178    |

# Jitter Testing

We found the jitter testing results differs when we format the SD card with different allocation unit size. Allocation units are the granularity with which disk space will be allocated to files, so if you have a 4k allocation unit, a 1k file will be allocated 4k, and a 10k file will be allocated 12k. A smaller allocation size will give less "wasted" space, but will also be slower to access (actual data) due to increased MFT/FAT access. When we set the allocation units to 32K for FAT32 system, we had an acceptable jitter rate although we don't find what the optimal allocation units size should be.

To format a SD card, we have two ways (supposing the SD disk drive is E)

1. use dos command `format E: /Q/FS:FAT32/A:32K`
2. use matlab command `dos('format E: /Q/FS:FAT32/A:32K');`

__Note__: In the case 'Format' is not recognized as a internal or external command, you need to add the path `C:\Windows\System32` at the end of the windows environment variable PATH.
{:.info}

The jitter test results using a formatted SD using the above methods are list as follows:

## Test 3

Grayscale = 1, no row compression, 4x4 check pattern, sampling rate 10K

Method one: continuously save pattern data frame after frame. Benchmark test result: 201.

| Frame Rate         | 10        | 20        | 30        | 40        | 50        | 60        | 70        | 80        | 90        | 100       | 110       | 120     |
| ------------------ | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | ------- |
| Mean IFI           | 0.1013    | 0.0506    | 0.0337    | 0.0253    | 0.0203    | 0.0169    | 0.0145    | 0.0126    | 0.0112    | 0.0101    | 0.0092    | 0.0084    |
| Standard deviation | 0.7098e-3 | 0.7046e-3 | 0.7189e-3 | 0.7135e-3 | 0.7120e-3 | 0.7056e-3 | 0.7065e-3 | 0.7351e-3 | 0.7121e-3 | 0.7336e-3 | 0.7093e-3 | 0.7065e-3 |
| Min IFI            | 0.1002    | 0.0495    | 0.0325    | 0.0242    | 0.0191    | 0.0157    | 0.0133    | 0.0115    | 0.0101    | 0.0090    | 0.0081    | 0.0073    |
| Max IFI            | 0.1024    | 0.0517    | 0.0349    | 0.0264    | 0.0214    | 0.0179    | 0.0156    | 0.0138    | 0.0124    | 0.0112    | 0.0104    | 0.0095    |

Method two: save each frame data starting from a new sector (512 byte each sector). Benchmark test result: 193.

| Frame Rate         | 10        | 20        | 30        | 40        | 50        | 60        | 70        | 80        | 90        | 100       | 110       | 120     |
| ------------------ | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | ------- |
| Mean IFI           | 0.1013    | 0.0506    | 0.0337    | 0.0253    | 0.0203    | 0.0169    | 0.0145    | 0.0126    | 0.0112    | 0.0101    | 0.0092    | 0.0084    |
| Standard deviation | 0.5588e-4 | 0.5272e-4 | 0.6415e-4 | 0.4680e-4 | 0.5350e-4 | 0.6145e-4 | 0.5832e-4 | 0.5062e-4 | 0.6256e-4 | 0.4816e-4 | 0.6019e-4 | 0.5934e-4 |
| Min IFI            | 0.1011     | 0.0505   | 0.0336    | 0.0252    | 0.0201    | 0.0167    | 0.0143    | 0.0125    | 0.0111    | 0.0100    | 0.0090    | 0.0083    |
| Max IFI            | 0.1014     | 0.0508   | 0.0339    | 0.0255    | 0.0204    | 0.0170    | 0.0146    | 0.0128    | 0.0114    | 0.0102    | 0.0094    | 0.0086    |

## Test 4

Grayscale = 3, no row compression,  pattern_lift_gs3.mat, sampling rate 10K

Method one: continuously save pattern data frame after frame. Benchmark test result: 78.

| Frame Rate         | 10     | 20     | 30     | 40     | 50     | 60     |
| ------------------ | ------ | ------ | ------ | ------ | ------ | ------ |
| Mean IFI           | 0.1013 | 0.0506 | 0.0337 | 0.0253 | 0.0203 | 0.0169 |
| Standard deviation | 0.0008 | 0.0008 | 0.0008 | 0.0008 | 0.0008 | 0.0008 |
| Min IFI            | 0.0992 | 0.0487 | 0.0318 | 0.0234 | 0.0182 | 0.0148 |
| Max IFI            | 0.1039 | 0.0532 | 0.0362 | 0.0278 | 0.0227 | 0.0194 |

Method two: save each frame data starting from a new sector (512 byte each sector). Benchmark test result: 75.

| Frame Rate         | 10     | 20     | 30     | 40     | 50     | 60     |
| ------------------ | ------ | ------ | ------ | ------ | ------ | ------ |
| Mean IFI           | 0.1013 | 0.0506 | 0.0337 | 0.0253 | 0.0203 | 0.0169 |
| Standard deviation | 0.0003 | 0.0003 | 0.0003 | 0.0003 | 0.0003 | 0.0004 |
| Min IFI            | 0.1000 | 0.0492 | 0.0323 | 0.0237 | 0.0188 | 0.0155 |
| Max IFI            | 0.1026 | 0.0522 | 0.0353 | 0.0270 | 0.0219 | 0.018  |

# Pattern making

When making large patterns and previewing them with the PControl GUI, the data field of the pattern struct is not necessary. Commenting out the following line will allow you to view the pattern in the pattern player before taking the time to make them for writing to the SD card (which requires the data field): `pattern.data = make_pattern_vector(pattern);`

# Gains over 127

For making the gain faster. This does not allow reaching the maximum speed, which could be reached having nonzero values for both gain and bias in a given channel, but just lets values greater than 127 be piped into a normal workflow. The conversion is `gain = bias * 2.5`.

```matlab
% Deal with values over 127. Gains is the four
% element array with [gain_X bias_X gain_Y bias_Y]
% sent to Panel_com with 'send_gain_bias'
if abs(Gains(1))>127 % then set X bias
    Gains(2) = Gains(1)/2.5;
    Gains(1) = 0;  
end              
if abs(Gains(3))>127 % then set Y bias 
    Gains(4) = Gains(2)/2.5;
    Gains(3) = 0;  
end
Panel_com('send_gain_bias',Gains);
```
