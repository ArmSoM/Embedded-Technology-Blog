# RK3588 Mass Production Testing:DDR Bandwidth Monitoring of ArmSoM-W3 

# 1. Introduction

- [Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)

- Before mass production, the ArmSoM team will conduct several rounds of professional functional testing and performance stress testing on the products to ensure product quality and stability.

- Excellent products require multiple comprehensive functional and performance stress tests to withstand market validation.

# 2. Environment Introduction

- Hardware environment: 
ArmSoM-W3 RK3588 development board

- Software version:
OS: ArmSoM-W3 Debian11

# **3.** ArmSoM-W3 DDR Bandwidth Testing Scheme

- rk-msch-probe-for-user is an official tool used to monitor and collect statistics on system DDR load and bandwidth usage. It can display real-time DDR load and bandwidth information.

- Use the rk-msch-probe-for-use tool to monitor and collect statistics on system DDR load and bandwidth usage.

# 4. DDR Bandwidth Testing 

- Test principle: Run the official Rockchip DDR bandwidth test tool to monitor and collect statistics on system DDR load and bandwidth usage.

- Test date: October 11, 2023

- Test tool: RK3588 - ArmSoM-W3 development board, power supply, screen, HDMI cable, mouse, serial port

## 4.1 Test Steps:

1. The rk-msch-probe-for-user tool needs to run in frequency lock mode.
   Set DDR frequency lock to the highest frequency 2112MHz

    ```bash
   //Switch to user space
   root@linaro-alip:/# echo userspace > sys/class/devfreq/dmc/governor
    
   //Get supported frequency information  
   root@linaro-alip:/# cat sys/class/devfreq/dmc/available_frequencies
   528000000 1068000000 1560000000 2112000000
    
   //Set DDR frequency lock to the highest frequency 2112MHz
   root@linaro-alip:/# echo 2112000000 > sys/class/devfreq/dmc/userspace/set_freq
    ```

2. Change rk-msch-probe-for-use tool permissions to 777

    ```bash
   chmod 777 ./data/rk-msch-probe-for-user-64bit
   ```

3. Start running

    ```bash
   ./data/rk-msch-probe-for-user-64bit -c rk3588
   ```

   
   ```bash
   root@linaro-alip:/# ./data/rk-msch-probe-for-user-64bit -c rk3588
   V1.44_20230928
    
   2kijec4hi======================================================================================================
   ddr freq: 2112Mhz          cpu      vicap        gpu        vop        isp     others      total
   master bw(MB/s)           0.64       0.00       0.00    1019.79       0.00      24.79    1045.22
   bw prorated(%)            0.06       0.00       0.00      97.57       0.00       2.37     100.00
   utilization(%)            0.00       0.00       0.00       3.02       0.00       0.07       3.09
   ----------------------------------------------ALL-------------------------CH0-------------------------CH1-------------------------CH2-------------------------CH3--------
                recorded LOAD: max 1045.22MB/s(3.09%), min 1045.22MB/s(3.09%), avg 1045.22MB/s(3.09%)
                         LOAD:         1045.22MB/s(3.09%),          261.50MB/s(3.10%),          261.24MB/s(3.09%),          261.18MB/s(3.09%),          261.31MB/s(3.09%)
                           RD:         1045.16MB/s(3.09%),          261.46MB/s(3.09%),          261.23MB/s(3.09%),          261.17MB/s(3.09%),          261.30MB/s(3.09%)
                           WR:            0.07MB/s(0.00%),            0.04MB/s(0.00%),            0.01MB/s(0.00%),            0.01MB/s(0.00%),            0.01MB/s(0.00%)
   -+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-
   =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
   ```
4. Run applications that need to monitor ddr information on the device, and monitor ddr bandwidth usage in real time.

## 4.2 Explanation of Test Statistics Results
According to the test results above: During the 1000ms monitoring time, the average bandwidth of all channels is 1045.22MB/s, and the load is 3.09%.



> ALL: Total bandwidth statistics for all channels

> CHx: Bandwidth statistics for DDR channel x

> LOAD: Bandwidth and load for this channel across all DDR banks

> RD: Read bandwidth and percentage across all DDR banks

> WR: Write bandwidth and percentage across all DDR banks

## ArmSoM Product Introduction: [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)

## ArmSoM Technical Forum: [http://forum.armsom.org/](http://forum.armsom.org/)