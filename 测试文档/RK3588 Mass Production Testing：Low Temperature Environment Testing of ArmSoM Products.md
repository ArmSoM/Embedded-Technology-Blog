# RK3588 Mass Production Testing：Low Temperature Environment Testing of ArmSoM Products 

## 1. Introduction

- [ Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)

- Before mass production, the ArmSoM team will conduct several rounds of professional functional testing and performance stress testing on the products to ensure product quality and stability.

- Excellent products require multiple comprehensive functional and performance stress tests to withstand market validation.
- This article outlines low temperature environment testing of the ArmSoM-W3 for RK3588 mass production testing.

## 2. Environmental Introduction

- Hardware Environment: 
ArmSoM-W3 RK3588 Development Board

- Software Version:
OS: ArmSoM-W3 Debian11 

- Temperature:
-20°C low temperature environment

## 3. ArmSoM-W3 Low Temperature Test Scheme

- 2000 software reboots in -20°C low temperature environment

- 2000 power cycle hardware reboots in -20°C   low temperature environment 

- 24 hours high load running in -20°C  low temperature environment


![Low temperature test setup](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/Low-temperature-test.jpeg#pic_left=500x)

## 4. -20°C Software Reboot Testing

- Test Principle: Place ArmSoM-W3 boards in -20°C environment,and perform 2000 software reboots, then observing if boards boot normally.

- Test Time: May 7, 2023 9:55 AM- May 8, 2023 13:50PM

- Test Tools: 6 serial ports, 6 ArmSoM-W3 boards, power supply, computer, HDMI cables and displays

- Test Steps:

	> 1. Prepare 5 ArmSoM-W3 boards, connecting serial port and power supply, and place all in chamber.
	> 2. Open MobaXterm to record logs during test.
	> 3. Lower chamber to -20°C, holding for 4 hours, and check if boards boot normally. Power cycle every 15 minutes, and repeat 1000 times. Check if boards boot normally.
	> 4. Check logs print for actual number of board reboots , and check for anomalies during the power cycle.
	
- Test Results: There are  5 boards operated normally after  2000 software reboots completed.

![Low temp software reboot test](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/Low-temperature-test1.jpeg#pic_left=600x)

## 5. -20°C Hardware Power Cycle Reboot Test 

- Test Principle: Place ArmSoM-W3 boards in -20°C environment, and perform 2000 power cycle reboots, observing if board boot operation normally.

- Test Time: May 8, 2023 17:52 PM - May 10, 2023 9:02 AM  

- Test Tools: Serial ports, 6 ArmSoM-W3 boards, timer for power cycling(set to automatically power off/on) .

- Test Software: MobaXterm to record log prints during test and save logs.

- Test Steps:

	> 1. Prepare 5 ArmSoM-W3 boards, connecting power supply of timers and connecting serial ports to computer. Place all in chamber. 
	> 2. Open MobaXterm to record logs during test.
	> 3. Set timer to 25s boot and 10s shutdown for boards.
	> 4. Lower chamber to -20°C, holding for 4 hours, and check if can normally boot. Power cycle every 15 minutes, and repeat 1000 times.See if it can boot normally.
	> 5. Check logs print for actual number of board reboots , and check for anomalies during the power cycle.
	
- Test Results: There is no anomalies detected in hardware reboot test after 2000 power cycle reboots completed and reboot counts differed by 1-2 due to margin of error.

## 6. 24 Hours High Load at -20°C 

- Test Principle: Run ArmSoM-W3 boards at high load for 24 hrs in -20°C environment, observing for anomalies.

- Test Time: May 7, 2023 9:55AM - May 8, 2023 13:50PM  

- Test Tools: 6 serial ports, 6 ArmSoM-W3 boards, power supply, computer, HDMI cables and displays

- Test Steps:

	> 1. Prepare 5 ArmSoM-W3 boards, connecting serial ports and power supply. Place all in chamber.
	> 2. Connecting to the serial ports ,save serial output logs during reboot.
	> 3. Lower chamber to -20°C, holding for 4 hrs. Start high load software, and monitor boards for anomalies.
	> 4. Check logs for anomalies during 24 hrs run.
	
- Test Results: There are  5 boards operated normally after ran for 24 hrs at high load in -20°C .

![24 hour low temp load test](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/Low-temperature-test2.jpeg#pic_left=600x)