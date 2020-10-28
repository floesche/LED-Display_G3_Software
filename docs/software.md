---
title: Software
parent: Generation 3
has_children: true
has_toc: false
nav_order: 2
---

# Overview

This project documents the hardware and software of an updated version of the Panels, a modular display system typically used for insect vision research. This is an update of [the old panel system](g2-panels.md), which we call the Panel Display Controller (PDC) v2. The new system, PDC v3, uses four parallel I2C channels to transfer pattern data from the panel display controller to the arena, making these system nearly 4 times faster as compared to PDC v2. Several other improvements have been added to the system to make it more flexible and user-friendly, these are detailed below.


This repository includes the code for several related projects:

- C code for the Xmega panel controller
  - bootloader for controller
  - application code which merged two working modes: controller mode and PC dumping mode
- C code for the panels
  - bootloader for each panel
  - application code for panels
- PC-side Matlab code

# New Features

## Four I2C bus communication between controller and arena

The most important of the new feature of the Xmega controller is the the number of I2C communication channel increases from 1 to 4. Since the bottleneck to increase the frame rate is on the I2C communication, the frame per second increase almost 4 times for the new system.

In order to take advantage of the four I2C channels, we arrange the panel map as follows in order to transfer frame data on four I2C at the same time.

|                | J1 | J2 | J3 | J4 | J5 | J6 | J7 | J8 | J9 | J10 | J11 | J12 |
| -------------- | --:| --:| --:| --:| --:| --:| --:| --:| --:| ---:| ---:| ---:|
| **Bus number** |  1 |  1 |  1 |  2 |  2 |  2 |  3 |  3 |  3 |   4 |   4 |   4 |
| **top**        |  1 |  5 |  9 |  2 |  6 | 10 |  3 |  7 | 11 |   4 |   8 |  12 |
|                | 13 | 17 | 21 | 14 | 18 | 22 | 15 | 19 | 23 |  16 |  20 |  24 |
|                | 25 | 29 | 33 | 26 | 30 | 34 | 27 | 31 | 35 |  28 |  32 |  36 |
| **bottom**     | 37 | 41 | 45 | 38 | 42 | 46 | 39 | 43 | 47 |  40 |  44 |  48 |


## Function generator upgrade 

The new system read function data from the SD card instead of loading them from PC at the beginning. In this way, we can have functions with any length. The size of function data increase to 2 bytes from 1 bytes, so the range of the data value is from -32768 to 32768. The function update rate can be adjusted for X and Y channel separately. The upper limit for the rate is hard-coded to 500 in Panel_com.m.

## Synchronize SD information

With this feature, the controller can copy the SD.mat from the SD card to PC side Matlab controller path defined in panel_control_paths.m. This feature is useful when the user want to use a SD card among different systems. After the user insert the sd card from another xmega controller, he just needs to synchronize the SD information with new PC, the new PC gets the information of the patterns and functions in the SD card.

## Configure arena

With this feature, we can get the  customized the arena configuration, such as how many panels in the arena and the channel number of each panel. If your arena con figuration is not the default one shown on the arena configuration table, you should change the table and load the cfg file to the sd card. in this way, the controller won't send data to non-existing panels and the performance is better.

## Program panel

After flashing the boot-loader to the panel atmel168 chip, we can send new panel code and eeprom code to the panels via I2C bus from controller. First step is to copy the latest panel.hex and  panel.eep to the SD card, and the next step is to click the panel flash and panel eeprom button in the PControl menu.

## Status

Report the current status of the system, including current pattern, current X function, current X function update rate, current Y function, and current Y function update rate.


## Quiet mode

quiet mode is for the experimental script mode only. When this mode is on, the controller stops send information, including warning and error information, to PC in order to save time. The default mode is off.

Turn on the quiet mode: `Panel_com('quiet_mode_on')`

turn off the quiet mode:  `Panel_com('quiet_mode_off')`

## Dump frame data

With this feature, the user can dump frame data from PC to controller via serial port directly.The current frame rates are close to the benchmark value. ie, for gs = 1, the maximum frame rate is 190 and when gs = 3, the maximum is 70. 

Ran the command in Matlab directly

```matlab
Panel_com('dump_frame', ...
    [frame_data_length, ...
    X_dac_val, Y_dac_val, ...
    numofPanel, grayScale, ...
    rowCompression, data] );

% first argument is the 'dump_frame' 
% argument 2 is the length of the data
% argument 3 is x_AO,  0 <= x_AO < 2048
% argument 4 is y_AO, 0 < y_AO < 2048
% argument 5 is the number of the panels
% argument 6 is the gray scale level
% argument 7 is the flag for the row compression
% argument 8 to the end is the frame data

% for example:
Panel_com('dump_frame', [384, 1, 0, 48, 1, 0, round(rand(1,8*48)*255)]);
```


## Set laser pattern

The user can set a new laser pattern. The laser patter should be a numerical array with 96 elements whose value is 0 or 1.
```matlab
Panel_com('send_laser_pattern',pattern);

% argument pattern is a numerical array with 96 elements whose value is either 0 or 1
% for example
Panel_com('send_laser_pattern' [ones(1,24),zeros(1,24),ones(1,24),zeros(1,24)]); 
```

## Set analog output

The user can set an output value from 0 to 2047, corresponding to 0-5V, to a DAC channel 1-4.

```matlab
Panel_com('set_ao',[chan, val]);

% argument chan ranges from 1 to 4, val ranges from 0 to 2047
% for example
Panel_com('set_ao',[1, 1000]);
```

Note:  The output range for the DAC 1-4 is from -5V to +5V. We only use 0-5V for this feature currently.

## Get analog input value

Since ADC input ranges from 0 to 10 V,  users can get an analog input value from 0 to 4095, from one of the ADC channel 1-4.

```matlab
Panel_com('get_adc_value', chan);

% argument chan ranges from 1 to 4
% for example
Panel_com('get_adc_value',1);
```