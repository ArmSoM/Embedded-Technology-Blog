# RK3588 Mass Production Testing：High Temperature Environment Testing of ArmSoM Products  

## 1. Introduction

- [ Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)

- Before mass production, the ArmSoM team will conduct several rounds of professional functional testing and performance stress testing on the products to ensure product quality and stability.

- Excellent products require multiple comprehensive functional and performance stress tests to withstand market validation.
- This article outlines high temperature testing of the ArmSoM-W3 for RK3588 mass production testing.

## 2. ArmSoM-W3 High Temperature Test Scheme 

- 2000 software reboots in 70°C high temperature environment

- 2000 power cycle hardware reboots in 70°C environment

- 24 hours high load running in 70°C environment

![High temperature test setup](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/High-temperature-test.jpeg#pic_left=500x)

## 3. 70°C High Temperature Software Reboot Test

- Test Principle: Place ArmSoM-W3 boards in 70°C environment,and perform 2000 software reboots, observing if boards boot normally.

- Test Time: May 9, 2023 9:55AM - May 10, 2023 13:50PM  

- Test Tools: 6 serial ports, 6 boards (1 green core board, 5 blue core boards), power supply, computer, HDMI cables and displays

- Test Steps:

	> 1. Prepare 5 ArmSoM-W3 boards, connecting serial ports and power supply, and place all in chamber.
	> 2. Open MobaXterm to record logs during test.
	> 3. Raise chamber to 70°C, holding for 4 hrs, and check if it can normally boot. Power cycle every 15 mins, repeating 2000 times. Check if it can normally boot.
	> 4. Check logs print for actual number of board reboots , and check for anomalies during the power cycle.
	
- Test Results: There are  5 boards operated normally after 1000 software reboots completed.

## 4. 70°C Hardware Power Cycle Reboot Test

- Test Principle: Place ArmSoM-W3 boards in 70°C environment, and perform 2000 power cycle reboots, observing if boot is abnormal.

- Test Time: May 10, 2023 17:52PM - May 12, 2023 9:02AM

- Test Tools: Serial ports, 6 boards(1 green core board, 5 blue core boards), power cycling timers(Automatically power off, power on)

- Test Software: MobaXterm to record logs and save.

- Test Steps:

  > 1. Prepare 5 ArmSoM-W3 boards, connecting power supply of timers and serial ports connected to computer. Place all in chamber.
  > 2. Open MobaXterm to record test logs.
  > 3. Set timers to 25s boot and 10s shutdown cycles.
  > 4. Raise chamber to 70°C, holding for 4 hrs, and check if it can normally boot. Power cycle every 15 mins, repeating 2000 times. Check if it can normally boot.
  > 5. Check logs print for actual number of board reboots , and check for anomalies during the power cycle.

- Test Results:  There is no anomalies detected in hardware reboot test after 1000 power cycle reboots completed and reboot counts differed by 1-2 due to margin of error.

## 5. 70°C High Temperature Operational Test

- Test Principle: Run ArmSoM-W3 boards at high load for 24 hrs in 70°C environment, observing for anomalies.

- Test Time: May 13, 2023 9:55AM - May 14, 2023 13:50 PM 

- Test Tools: 6 serial ports, 6 boards, power supply, computer, HDMI cables and displays

- Test Steps:

	> 1. Prepare 5 ArmSoM-W3 boards, connecting serial ports and power supply, and place all in chamber.
	> 2. Connect to serial port,saving serial port logs during test.
	> 3. Raise chamber to 70°C, holding for 4 hrs. Start high load software, and monitor boards for anomalies. 
	> 4. Check logs for anomalies during 24 hrs run.

- Test Results: There are  5 boards operated normally after ran for 24 hrs at high load in 70°C environment