# RK3588 Mass Production Testing: ArmSoM Software 

## 1. Introduction

- [[Mastering RK3588 from Beginner to Expert] Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)
- Excellent products need to undergo rigorous product testing before they can withstand market validation.
- The production testing software developed by the ArmSoM team is used to quickly identify good and bad product functions and devices during mass production, i.e. key FCT (Functional Test) testing, thereby improving production efficiency and detection accuracy.  
- ArmSoM professional production testing software is used to ensure the quality and stability of each mass produced product.
- ArmSoM Production Testing Software Preview:

![ArmSoM Production Testing Software](https://github.com/ArmSoM/Embedded-Technology-Blog/blob/main/image/armsom-test/ArmSoM_Production_Testing_Software.png#pic_left=500x)

## 2. Environmental Introduction

- Hardware Environment: ArmSoM-W3 RK3588 Development Board

- Software Version: OS: ArmSoM-W3 Debian11

## 3. ArmSom Mass Production Testing Software Introduction
- QT developed ARM platform graphical production testing software, one click to start foolproof testing.

- ArmSoM production testing software works on Linux and Android platforms: Ubuntu, Debian, and Android systems.

- ArmSoM production testing software is installed directly on the RK3588 dev board which just connect a display to open the software and start testing.

- ArmSoM production testing software is developed in-house and now applied for commercial mass production testing.

## 4. Technical Details:
- Multi-threading with thread pool, all interfaces tested in parallel for greatly improved test efficiency.

- qt + opencv implemented camera , hdmi-in for real-time video display.

- Precisely written interface test code with high accuracy in identifying good/bad interfaces.

- Supports testing multiple dev boards. Users can manually select boards to test.

- Supports multiple platforms. Support test various architecture boards, not limited to ARM.

- Highly extensible, customize interfaces to test and expand full function testing.

## 5. Currently Supported Test Interfaces
- Includes auto and manual test items.

- Auto tests report results and pass/fail without manual intervention.

- Manual tests require judging if test completed correctly, and providing pass/fail verdict.

- Currently supports: WIFI, Bluetooth, USB, LED, Speaker, Mic, Camera, Hdmi-in, 40PIN, Ethernet, M2, RTC, etc.

- Highly extensible, customize interfaces to test and expand full function testing.

## 6. Mass Production Testing Video:

[ArmSoM Mass Production Testing Demo Video](https://live.csdn.net/v/embed/300962)

## ArmSoM Product Introduction: [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)

## ArmSoM Technical Forum: [http://forum.armsom.org/](http://forum.armsom.org/)