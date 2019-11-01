# temp_freq_cpus

[temp_freq_cpus](tools/temp_freq_cpus) is a Raspbian tool monitoring:
* temperature of BCM2835 SoC
* arm clock frequency
* cpu statistics per core
* summary cpu statistics

It uses VT100 escape code to clear screen and then displays the measured information in this layout:


```

11/01/19 13:49:19.154(0.99)  temp=53.0'C  frequency(48)=600169920

%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us, 16.7 sy,  0.0 ni, 83.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

%Cpu(s):  4.0 us,  1.3 sy,  0.0 ni, 94.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

```

The first empty line allows to grep the other lines without the clear scren escape, the other two empty lines separate information. The tool determines all information, and after a specific amount of time is gone, then the screen gets updated.

First line shows date and  time at millisecond precision, the passed delay (in seconds, does not need to be integer) in parenthesis, the soc temperature and arm frequency.

The 2nd block is cpu statistics per core taken from top command.

The 3rd block is summary cpu statistics from top command.


Best use is by utilising "tee" command to store the output to log file and see live on screen. This is command that is useful to get rougly updates every second:

    temp_freq_cpus 0.99 | tee log.txt

The [sample log file](step2.log.txt) was captured that way, while doing dependency installations for OpenCV according "step 2" of [this instruction](https://www.pyimagesearch.com/2019/09/16/install-opencv-4-on-raspberry-pi-4-and-raspbian-buster/). Passing 0.99 seconds resulted in nearly exact 1 second steps between log entries.

Looking at the log file is not simple because of the contained VT100 escapes. You can best view a log file on command line with "more -9 log.txt" command. That ensures that only the 9 lines belonging together are shown. Pressing "z" shows the next 9 lines, pressing "b" shows the previous 9 lines.

There are some tools that allow to extract information from a log file.

[log_freq](tools/log_frew) does a frequency distribution analysis:

    $ tools/log_freq step2.log.txt 
        111 600117184
        166 600169920
         57 1500345728
         40 1500398464
    $

[log_temp](tools/log_temp) does temperature distribution analysis:

    $ tools/log_temp step2.log.txt 
          1 49.0
          8 50.0
         55 51.0
        111 52.0
        132 53.0
         61 54.0
          6 55.0
    $

[log_cpus_us](tools/log_cpus_us) does summary user time analysis (in percent, skipping the fractional part information received from top):

    $ tools/log_cpus_us step2.log.txt 
        110   0
        127   1
         46   2
         10   3
         11   4
          6   5
          7   6
          4   7
          2   8
          5   9
          5  10
          3  11
          3  12
          3  13
          2  15
          5  16
          2  17
          2  18
          2  20
          2  21
          1  23
          2  24
          6  25
          3  26
          1  27
          2  28
          1  29
          1  37
    $

These tools can be used as basis for other log information extraction tools.

Finally [log2csv](tools/log2csv) converts a log file into CSV standard data format for further processing by eg. spreadsheet.

    $ tools/log2csv step2.log.txt
    11/01/19,13:43:06.517,50.0,600169920
    ,,,,%Cpu0  ,0.0,0.0,0.0,100.0,0.0,0.0,0.0,0.0
    ,,,,%Cpu1  ,0.0,0.0,0.0,100.0,0.0,0.0,0.0,0.0
    ,,,,%Cpu2  ,0.0,16.7,0.0,83.3,0.0,0.0,0.0,0.0
    ,,,,%Cpu3  ,0.0,0.0,0.0,100.0,0.0,0.0,0.0,0.0
    ,,,,%Cpu(s),1.5,1.5,0.0,97.0,0.0,0.0,0.0,0.0
    11/01/19,13:43:07.520,49.0,600117184
    ...
    ...
    ,,,,%Cpu(s),0.0,5.3,0.0,94.7,0.0,0.0,0.0,0.0
    11/01/19,13:49:19.154,53.0,600169920
    ,,,,%Cpu0  ,0.0,0.0,0.0,100.0,0.0,0.0,0.0,0.0
    ,,,,%Cpu1  ,0.0,16.7,0.0,83.3,0.0,0.0,0.0,0.0
    ,,,,%Cpu2  ,0.0,0.0,0.0,100.0,0.0,0.0,0.0,0.0
    ,,,,%Cpu3  ,0.0,0.0,0.0,100.0,0.0,0.0,0.0,0.0
    ,,,,%Cpu(s),4.0,1.3,0.0,94.7,0.0,0.0,0.0,0.0
    $

The 1st and 374th(last) timestamps of [sample log file](step2.log.txt) show, that little less than 373 seconds have passed between first and last log entry (372.637 seconds).
