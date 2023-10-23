# RK3588 Mass Production Testing：DDR Stress of ArmSoM-W3 

## 1. Introduction

- [ Mastering RK3588 from Beginner to Expert ](https://blog.csdn.net/nb124667390/article/details/130725546)

- Before mass production, the ArmSoM team will conduct several rounds of professional functional testing and performance stress testing on the products to ensure product quality and stability.

- Excellent products require multiple comprehensive functional and performance stress tests to withstand market validation.

## 2. Environmental Introduction

- Hardware Environment: 
ArmSoM-W3 RK3588 development board

- Software Version:
OS: ArmSoM-W3 Debian11

## 3. ArmSoM-W3 DDR Stress Testing Scheme

**Testing Scheme: Perform three stress tests on DDR simultaneously:**

- Use memtester tool to stress test DDR

- Use stressapptest tool to stress test DDR 

- Use Rockchip official test script for DDR frequency scaling test

## 4. DDR Stress Testing 

- Test Principle: Run Rockchip official DDR stress test script, and perform three stress tests on DDR simultaneously, observing development board operation, then see if it can withstand 24 hours of continuously DDR stress testing.

- Test Time: August 31, 2023 9:55AM - September 1,  10:02AM  

- Test Tools: RK3588 - ArmSoM-W3 development board, power supply, screen, HDMI cable, mouse, serial port

- Test Steps:

	> 1. Prepare 5 ArmSoM-W3 boards, and flash all with ArmSoM-W3 Debian firmware.

	> 2. Connect boards to serial port, and save stress test serial port prints to log. Check if boards encounter anomalies during DDR stress test.

	> 3. Run Rockchip official DDR stress test script, and perform three stress tests on DDR simultaneously.

- Test Results: There are 5 boards operated normally after  24 hours of DDR stress testing.


## 5. DDR Stress Testing Process 

- Enter rockchip-test test directory via serial port:

	```
	root@linaro-alip:/# cd /rockchip-test 
	```

- Run rockchip-test test script: Select 1: ddr stress test, then select 5: stressapptest + memtester + ddr auto scaling

	```
	root@linaro-alip:/rockchip-test# ./rockchip_test.sh
	```

	![DDR stress test process](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/ddr-test-process.jpg)

- DDR stress test begins: 

	![DDR stress test begins](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/ddr-test-begins.png)
	![DDR stress test desktop](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/ddr-test-desktop.jpg)
	![DDR stress test product](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/ddr-test-product.jpg)

## ArmSoM Product Introduction: [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)

## ArmSoM Technical Forum: [http://forum.armsom.org/](http://forum.armsom.org/)