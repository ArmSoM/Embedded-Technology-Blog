# RK3588 Mass Production Testing：Software and Hardware Reboot Testing of ArmSoM-W3 

## 1. Introduction

- [Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)

- Before mass production, the ArmSoM team will conduct several rounds of professional functional testing and performance stress testing on the products to ensure product quality and stability.

- Excellent products require multiple comprehensive functional and performance stress tests to withstand market validation.

## 2. ArmSoM-W3 Software and Hardware Reboot Testing Scheme

- Software reboot system 3000 times test
- Hardware power cycle reboot 3000 times test

## 3. Software Reboot 3000 Times Test

- Test Principle: Perform 3000 software system reboots on the target board, and observe board operation, see if it can withstand 3000 continuously reboots.

- Test Time: May 7, 2023 9:55AM - May 8, 2023 13:50PM 

- Test Tools: RK3588 - ArmSoM-W3 dev board, power supply, screen, HDMI cable, mouse

- Test Steps:

	> 1. Prepare 5 ArmSoM-W3 boards, and flash all with ArmSoM-W3-Box firmware.
	> 2. Connect boards to serial port, and save serial port prints during reboot to log. Record actual number of board reboots. Check for anomalies during reboot.
	> 3. Use specialized reboot test software, and set reboot count to 3000. Click start to begin test.
	
- Test Results: There are 5 boards operated normally after 3000 times software reboots completed.

![reboot-testing-software](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/reboot-testing-software.jpeg#pic_left=600x)

## 4. Hardware Reboot 3000 Times Test

- Test Principle: Perform 3000 power cycle reboots on target board, and observe board operation, then see if it can withstand 3000 continuously hardware reboots.

- Test Time: May 8, 2023 17:52PM - May 10, 2023 9:02AM

- Test Tools: Serial port, computer, RK3588 - ArmSoM-W3 dev board, two timers (set to automatically power off/on) 

- Test Software: MobaXterm to record log prints during test and save logs.

- Test Steps:

	> 1. Prepare 5 ArmSoM-W3 boards, connecting timers power supply, and connecting serial port to computer.
	> 2. Open MobaXterm to record log prints.
	> 3. Set board boot time to 25s and shutdown time to 10s. 
	> 4. Check logs print for actual number of board reboots , and check for anomalies during the power cycle.
	
- Test Results: There is no anomalies detected in hardware reboot test after 3000 power cycle reboots completed and reboot counts differed by 1-2 due to margin of error.

![reboot-testing-hardware](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/reboot-testing-hardware.jpeg#pic_left=600x)