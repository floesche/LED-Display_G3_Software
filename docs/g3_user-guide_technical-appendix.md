---
title: Technical Appendix
grand_parent: Generation 3
parent: User Guide
nav_order: 1
---

# Technical Appendix 1 – Example Arena Experiments and Code

The following instructions assume a standard arena setup with 44 panels
(four rows and eleven columns), MATLAB is installed, and the `controller`
folder from the [Panels Software Repository](https://github.com/floesche/LED-Display_G3_Software)) is included in your MATLAB
path. Going from idea to execution, this section lists the main steps
and is intended as a standalone example. Note that more information on
troubleshooting and setup is [available]({{site.baseurl}}/Generation%203/Software/g3_troubleshooting.html).

## Plan a stimulus

What type of experiment is desired? If there is literature already using
the desired stimulus, then methods sections might be a good place to
start. Even if there is not a history of your stimulus being used, this
might be a good starting point if it is unclear exactly what happens in
flight arena experiments. Once you have a good idea, drawing out the
pattern is helpful for the next step. For the example we have chosen a
simple expansion-contraction pattern, where a grating will expand from
some position and contract 180° from it. For the experiment, we will be
testing how the position of the expansion or contraction impacts the
turning response. Specifically, we will want to test the turning
response at one of three foci of expansion or contraction.

## Design the pattern(s)

Once you know what stimulus you want to use, you must create it using a
MATLAB m-file. Open a new m-file in the MATLAB editor, and name it
something related to your stimulus, to keep things organized name it
starting with `make_` followed by a description. In order to generate a
pattern from this file, a few parameters must be defined. The Software
Overview section has information on the variables being set in this
example; here we will reiterate the points made. For our expansion
example, if thinking of the X and Y channels as axes of a memory buffer,
the X channel will be all the different possible frames of expansion
from one point (say, directly in front of the fly). The Y channel can
then be all of the different shifts, or starting positions for this
expansion. Note that each of the X-Y positions now corresponds to a
state, or frame, of the arena. Here is how this is done.

First we need to populate and save a MATLAB structure called pattern and
save the pattern to a file for later use.

```matlab
pattern.x_num = 96; % x is all frames of one expansion

pattern.y_num = 96; % y is different starting positions of expansion,
one for each position in the arena

pattern.num_panels = 48;

pattern.gs_val = 3; % So we can play around with brightness later

pattern.row_compression = 1; % this can be used here because all of the
columns in our pattern can be easily represented by a single row of the
arena LEDs
```

Now we have set the stage for our pattern. There are 96 positions in
each axis of our memory buffer. Because the pattern could also be
represented by one row of the panels, we can write the pattern for just
one row and by using row compression, our 96 by 32 array of individual
LEDs can be represented as an array of only 96 by 4 LEDs. To start, it
is good form to pre-allocate a block of space for MATLAB to work with.
We will call the 4-D matrix `Pats`.

```matlab
Pats = zeros(4, 96, pattern.x_num, pattern.y_num); % Preallocate space
```

Next, we have to populate our blank pattern. The `repmat` function comes
in handy here, typing help `repmat` in the command window gives a
description of how it works. We start by populating the first frame
(where x and y both = 1).

```matlab
Pats(:,:,1,1) = [repmat([7*ones(4,4), 0*ones(4,4)], 1, 6),...
    repmat([0*ones(4,4), 7*ones(4,4)], 1, 6)];
```

Now we manipulate this first frame to populate the other channels. You
may first want to do the math to convince yourself you have filled up
all 96 LED columns.

```matlab
for i = 1:pattern.y_num
    for j = 2:pattern.x_num
        Pats(:,:,j,i) = simple_expansion(Pats(:,:,j-1,i), 49,96);
    end
end

for g = 1:pattern.y_num;
    for i = 1:pattern.x_num
        Pats(:,:,i,g) = circshift(Pats(:,:,i,1),[0 i]);
    end
end
```

Go through these lines to understand how they populate the pattern file
by first creating a filled pattern with all identical X channel
expansion, and then shift the matrix one column per each Y index. Be
sure to note there need not be 96 Y channels. Because the experiment
only wanted to test three different starting points of expansion, the Y
channels could be individually assigned to three different starting
points any number of ways (one of which is increasing the shift of
ShiftMatrix). Just know this is not the only way to make this pattern,
or any pattern!

```matlab
pattern.Pats = Pats;
new_controller_48_panel_map = [12 8 4 11 7 3 10 6 2 9 5 1;...
                               24 20 16 23 19 15 22 18 14 21 17 13;...
                               36 32 28 35 31 27 34 30 26 33 29 25;...
                               48 44 40 47 43 39 46 42 38 45 41 37];
pattern.Panel_map = new_controller_48_panel_map;
pattern.BitMapIndex = process_panel_map(pattern);
pattern.data = make_pattern_vector(pattern);
directory_name = 'C:\MatlabRoot\Panels\Patterns';

str = [directory_name 'Pattern_expansion']
save(str, 'pattern');
```

The `Panel_map` attribute of the structure is set equal to a map of the
arena panels. The given matrix is an optimized arrangement of panel IDs,
corresponding to the `48Panel_4Bus` arena configuration file (see below,
or in appendix). Be sure that the name you give your pattern begins with
`Pattern_`. Run this m-file in MATLAB to create a `.mat` pattern.

To play this pattern, select from the PControl GUI _configurations_{:.gui-btn} →
_play pattern_{:.gui-btn} → _choose your pattern_{:.gui-btn}, noting the majority of the frames
are black (unpopulated). We will next use a number of functions created
to ease pattern creation, all of which are included in the
`\Panels\Pattern_tools\` folder you already included in the MATLAB path.
Refer to the help, or directly to the functions for better
understanding.

Understand that `Pattern_expansion`, when played with positive gain, will
be expansion, but when played with a negative gain, will be contraction.
Instead of needing to make a second pattern, we can use the one pattern
twice for our desired experiment.

## Making an experiment script

In order to have more control over patterns and settings, an experiment
script will be needed. There are very few necessary settings (the same
ones from the PControl GUI), and they can be changed easily using the
Panel_com command. To run an experiment these are the only lines needed:

```matlab
Panel_com('set_mode',Mode);
Panel_com('send_gain_bias',[Gain_X Bias_X Gain_Y Bias_Y]);
Panel_com('set_pattern_id', Pattern_ID);
Panel_com('set_position', [Ind_X Ind_Y]);
Panel_com('start')
pause(Time) % The pattern will run for this ‘Time’
Panel_com('stop')
```

Each of the `Panel_com` commands are explained in the Software Overview
section and in the appendix. Some example values for each of the
variables passed are as follows:

```matlab
Mode = [0 0]; % Corresponds to closed loop for X and Y
Gain_X = 1;
Bias_X = 0;
Gain_Y = 0;
Bias_Y = 0;
Pattern_ID = 1; % The first pattern on the SD card
Time = 3;
Ind_X = 49; % This should center the pattern
Ind_Y = 1;
```

If this were saved as an experiment script it would run a pretty boring
experiment, doing nothing more than what you can from the PControl GUI.
What makes a more exciting experiment are more conditions, each of which
may have different Gains, Biases, Patterns, X and Y starting positions,
and, importantly, Modes. An easy way to do this is to store all of the
relevant condition attributes in a MATLAB structure called condition.
Here is an example of how this is done where the conditions differ in
three ways: speed of pattern (by changing `Gain_X`), direction of pattern
(by changing the sign of `Gain_X`) and starting orientation of the pattern
(by changing `Ind_Y`). Remember our desired experiment starts expansion or
contraction at one of three spots, and that by changing the direction
(sign) of pattern, we switch from expansion to contraction.


```matlab
speeds = [32 64 96];
time = 3;
condition_num = 1; % The first condition value

for i = 1:length(speeds)
    for k = 1:2 % Positive and Negative Speed
        for g = [28 36 49] % 3 Starting positions corresponding to the 
                           % 3 initial expansions or contraction points
            condition(condition_num).Y_ind = g;
            condition(condition_num).pattern = 1;
            condition(condition_num).X_ind = 1;
            condition(condition_num).X_gain = speeds(i);
            condition(condition_num).Y_gain = 0;
            condition(condition_num).X_bias = 0;
            condition(condition_num).Y_bias = 0;
            condition(condition_num).mode = [0 0];
            condition(condition_num).time = time;
            if k == 1; % Set the value to be pos
                condition(condition_num).X_gain = condition(condition_num).X_gain;
            else
                condition(condition_num).X_gain = -condition(condition_num).X_gain;
            end

            condition_num = condition_num + 1;
        end
    end
end

num_conditions = condition_num - 1;
```

These for loops create the condition structure with 18 different
conditions, the value of the num_conditions variable. Convince yourself
by examining the structure in MATLAB, that all conditions were made and
stored.

Now, by using the length of the num_conditions variable, we can loop
through the different conditions by adding a for loop and some other
details to the first code of this section. There is one critical
component missing, a voltage encoded condition signal. In order for us
to tell each condition apart when recording the experiment, each
condition must be associated with some signal also recorded. Thankfully
the data acquisition toolbox makes this painless, we create an analog
voltage object that can encode the condition signals using a value of
1-4 volts. This example is for a standard NIDAQ board with 32-bit
Windows, see below and MATLAB help for instructions on making analog
output objects with other boards and OS versions.

```matlab
AO = analogoutput('nidaq', 'Dev1');
chans = addchannel(AO, [0]);
```

Each time we switch conditions, the value of condition_num changes to
encode the next condition. The command putsample from the data
acquisition toolbox allows this.

```matlab
putsample(AO, condition_num/(num_conditions/4))
```

That is the last component of the script. Here it is, from start to
finish, with a few simple lines added to make it run nicely.

```matlab
clear all

%% Make the condition structure
speeds = [32 64 96];
time = 3;
condition_num = 1; % The first condition value

for i = 1:length(speeds)
    for k = 1:2 % Positive and Negative Speed
        for g = 1:3 % 3 Starting positions
            condition(condition_num).Y_ind = g;
            condition(condition_num).X_ind = 1;
            condition(condition_num).pattern = 1;
            condition(condition_num).X_gain = speeds(i);
            condition(condition_num).Y_gain = 0;
            condition(condition_num).X_bias = 0;
            condition(condition_num).Y_bias = 0;
            condition(condition_num).mode = [0 0];
            condition(condition_num).time = time;
            if k == 1; % Set the value to be pos
                condition(condition_num).X_gain = condition(condition_num).X_gain;
            else
                condition(condition_num).X_gain = -condition(condition_num).X_gain;
            end

            condition_num = condition_num + 1;
        end
    end
end

num_conditions = condition_num - 1;

%% Create an Analog Output Object (AO)
AO = analogoutput('nidaq', 'Dev1');
chans = addchannel(AO, [0]);

%% Experiment
condition_num = 1; % set this again to 1
num_reps = 3; % define the number of repititions you want
fprintf('Trial beginning')

conds_to_run = randperm(num_conditions);
fprintf(strcat('conds2run =', num2str(conds_to_run), ' \n'));

for i = 1:num_reps
    for j = 1:length(conds_to_run) % for each different speed
        cond_num = conds_to_run(j); % and take the values out of the condition struct
        Gain_X = condition(condition_num).X_gain;
        Gain_Y = condition(condition_num).Y_gain;
        Bias_Y = condition(condition_num).Y_bias;
        Bias_X = condition(condition_num).X_bias;
        Ind_Y = condition(condition_num).Y_ind;
        Ind_X = condition(condition_num).X_ind;
        Pattern_ID = condition(condition_num).pattern;
        Time = condition(condition_num).time;
        Mode = condition(condition_num).mode;
        fprintf('round %d, run %2d, cond num = %2d, speed = %2d, pause = %2d, start Y pos = %2d \n',i, j, cond_num, Gain_X, Time, Ind_Y);
        disp('...next')

        % Open Loop Begins
        % scale condition number to fit in 0-4V range
        putsample(AO, cond_num/(num_conditions/4)) 
        Panel_com('set_mode', [Mode]); % set to open loop with Panel_com
        Panel_com('set_pattern_id', Pattern_ID);
        Panel_com('send_gain_bias',[Gain_X Bias_Y Gain_Y Bias_Y]);
        Panel_com('set_position', [Ind_X Ind_Y]); 
        Panel_com('start')
        pause(Time)
        Panel_com('stop')
    end
end

clear AO
```

> __Important Note__
>
>There are many other possible ways to set up an experiment script, here are a few:
>
> - Alternate closed loop ‘reward’ stimuli with open loop experimental
> stimuli. This helps to keep the fly flying and engaged
>
> - Make the experiment script into a function that can take a number of
> individual stimuli as input. This is useful if some conditions did not
> run properly during the first repetitions.
{: .warning}

## Acquiring the Data

Once the experiment script is working, and flies are ready to be run
(see the Fly Preparation section), experimental data must be acquired.
Data is usually acquired using a standard data acquisition board (DAQ),
and a program such as LabVIEW, from National Instruments, axoscope, if
you are using an Axon board, or the freeware Spikehound, developed by
Gus Lott. The data acquisition toolbox in MATLAB works with all major
DAQ boards, can be used in place of the GUI programs above, and has
extensive help files for getting started. The programs require
configuration to acquire all relevant data from the DAQ board, this is
usually painless, but care should be taken to note the sampling rate
each experiment is conducted at.

A typical experiment will start with some closed loop time to properly
align the fly in the arena, starting the DAQ program, and then running
the experiment script. If the fly stops flying during one condition, it
is typically rerun before ending the DAQ program.

More detailed data acquisition instructions are available on the
bitbucket sites listed in the introduction.

## Preparing the Data

Depending on your DAQ program of choice you will now have a VI file from
LabView, an abf file from axoscope, or a daq file from the Data
Acquisition toolbox (i.e. Spikehound). Each of these formats can be
loaded to the MATLAB environment with varying degrees of difficulty. In
this example we will use a daq file, as this is the data acquisition
native format. To read a daq file use:

```matlab
Data = daqread('Filename.daq');
```

Note that for the proprietary axoscope format (abf files), reading into
MATLAB requires a special function freely available on the [MATLAB central file exchange website](https://www.mathworks.com/matlabcentral/fileexchange/).This will import the
file to a matrix named `Data` in the workspace. You can break the `Data`
matrix into individual DAQ channels with the following strategy (if you
are using the default configuration):

```matlab
Left = Data(:,1); % left wing beat amplitude
Right = Data(:,2); % right wing beat amplitude
% Insert the rest of the channels here
num_conditions = 18;
condition_signal = round((num_conditions/4)* Data(:,6));
```

If your condition signal, or left and right wing beat amplitudes were
recorded from different channels, then change the column extracted for
each attribute. We must now break up the data in terms of condition, we
have already recovered each condition signal in the condition_signal
variable, but there are some steps required to break the data up
effectively. Here is the entire script necessary to do so, note this
will not fit all situations, but is representative of voltage decoding
strategies. The code is heavily commented, and should be relatively
transparent after some thought.

```matlab
Data = daqread('Filename.daq');

Left = Data(:,1); % left wing beat amplitude
Right = Data(:,2); % right wing beat amplitude
% Insert the rest of the channels here, also needs some data from the
% experiment to be explicitly listed for later use

% From the original condition struct
num_conditions = 18;
Time = 3;

% Noted from the DAQ program settings
sample_rate = 1000;

% Recover the condition signal for the entire column of analog readings
condition_signal = round((num_conditions/4)* Data(:,6));

% establish what is a significant difference between condition signals, in
% this case 85% of the difference between two sequentials is good enough
sig_diff = (0.85*(num_condtions/4));

% create a variable with all of the Positions where the condition signal is
% significantly different (defined above)
diff_Pos = (find( abs(diff(condition_signal)) > sig_diff));
max_Pos = length(diff_Pos);

% create two arrays to define blocks of data, and make the current Position
% in the data sweep two
start_Pos = []; end_Pos = []; cur_Pos = 2;

% Search through the positions to establish blocks of data for each
% condition using both the diff_Pos and the time passing between each set
% of conditions to verify a condition change.
while cur_Pos <= max_Pos %when the current Position is less than or equal to the maximum
    Pos_diff = (diff_Pos(cur_Pos) - diff_Pos(cur_Pos - 1));
    range_start = diff_Pos(cur_Pos - 1);
    range_end = diff_Pos(cur_Pos);
    if ((Pos_diff > (Time-.15)*samp_rate)&&(Pos_diff < (Time+.15)*samp_rate))
        start_Pos = [start_Pos diff_Pos(cur_Pos - 1)];
        end_Pos = [end_Pos diff_Pos(cur_Pos)];
    end
    cur_Pos = cur_Pos + 1;
end

OL_data_length = Time*sample_rate;

% Create an empty struct, OL_Data for each condition and data line
for j = 1:num_conditions
    OL_Data(j).Left = [];
    OL_Data(j).Right = [];
end

% Populate the OL_Data struct for each condition, Index will correspond to
% each condition number in the original condition struct
for j = 1:(length(start_Pos))
    curr_Range = start_pos(j):start_pos(j)+OL_data_length-1;
    
    % make a condition number for each data range that corresponds to the
    % actual condition number
    cond(j) = round( mean(condition_signal(curr_Range)));

    Index = cond(j);
    OL_Data(Index).Left = [OL_Data(Index).Left; Left(curr_Range)'];
    OL_Data(Index).Right = [OL_Data(Index).Right; Right(curr_Range)'];
end

% cond and OL_Data from the previous for loop need to be saved together, we
% will use a struct named fly

fly.condition = cond;
fly.OL_Data = OL_Data;

save fly fly
```

You now have your data broken up into several conditions ready to be
graphed! More data, such as the frame index, and the wing beat frequency
could be added to this data set as well.

This brings us to the end of the first sample experiment tutorial.

## Original file

This user guide is an online version of the original appendix (available for [download in the original file format](assets/Flight%20Arena%20Appendix%201%20(exp%20scripts).docx)).
