# RK3588 MIPI Panel Debugging: RK3588-MIPI-DSI LCD Power up Initialization Sequence

# 1. Introduction

- [[Mastering RK3588 from Beginner to Expert] Series Contents](https://blog.csdn.net/nb124667390/article/details/130725546)
- For MIPI panels, the power on init sequence (panel-init-sequence) and power off sequence (panel-exit-sequence) are usually configured in the dts file on Rockchip platforms. This article explains how to configure these initialization sequences for the panel.

# 2. Environment Introduction

- Hardware:
   ArmSoM-W3 RK3588 dev board, MIPI-DSI panel (ArmSoM official accessory)

- Software Version:
   OS: ArmSoM-W3 Debian11

# 3.Data Types
# 3.1 Common data type1 ：DCS Write

Note: The parameter here does not refer to the number of data bytes.
```c
0x05  Command type: Single byte data (DCS Short Write, no parameters) 

0x15  Command type: Two byte data (DCS Short Write, 1 parameter)

0x39  Command type: Multi byte data (DCS Long Write, n parameters n > 2)
```

**0x05  Command type：(DCS Short Write, no parameters)**
	

```bash
05 95 01 11
05 95 01 29
```
**0x15  Command type：(DCS Short Write, 1 parameter)**
	

```bash
15 00 02 80 77
|  |  | |  |
|  |  | |  data
|  |  | | Register Address
|  |  Payload Length
|  Delay
Command type（0x05: Single byte data  0x15:Two byte data  0x39:Multi byte data）
	
Analysis：
Data Type: 0x15 (0x15 data type - DCS Short Write, 1 parameter)

Delay: 0x00 (Delay in ms after completing sending current packet before starting next command)

Payload Length: 0x02 (Payload length is 2 bytes)

Payload: 0x80 0x77 (Payload data)
```

**0x39 command type is with multiple parameters, more than two parameters (DCS Long Write / write_LUT Command Packet)**
	

```bash
39 00 06 FF 77 01 00 00 10
39 00 03 C0 63 00
39 00 03 C1 11 02	
```
# 3.2 Common data type 2 ：Generic Write

Note: The parameter here does not refer to the number of data bytes.
```xml
0x03 Command type: Single byte data  (Generic Short Write, no parameters)
	
0x13 Command type: Two byte data (Generic Short Write, 1 parameter)
	
0x23 Command type: Three byte data  (Generic Short Write, 2 parameters)
	
0x29 Command type: Multi byte data  (Generic Long Write, n parameters n > 2)
```

# 4.  Rockchip Platform Power on Initialization Sequence Configuration

There are panel manufacturer's MIPI panel initialization code as follows

```bash
params->dsi.vertical_sync_active=2
params->dsi.vertical_backporch=10
params->dsi.vertical_frontporch=14
params->dsi.horizontal_sync_active=24
params->dsi.horizontal_backporch=80
params->dsi.horizontal_frontporch=60
params->dsi.PLL_CLOCK=478
 
LCD_nReset=1;
Delayms(5);
LCD_nReset=0;
Delayms(20);//10
LCD_nReset=1;
Delayms(200);//120

Generic_Short_Write_1P(0xB0,0x01);	
Generic_Short_Write_1P(0xC0,0x26);	
Generic_Short_Write_1P(0xC1,0x10);	
Generic_Short_Write_1P(0xC2,0x0E);	
Generic_Short_Write_1P(0xC3,0x00);	
Generic_Short_Write_1P(0xC4,0x00);	
Generic_Short_Write_1P(0xC5,0x23);	
Generic_Short_Write_1P(0xC6,0x11);	
Generic_Short_Write_1P(0xC7,0x22);	
Generic_Short_Write_1P(0xC8,0x20);	
Generic_Short_Write_1P(0xC9,0x1E);	
Generic_Short_Write_1P(0xCA,0x1C);	
Generic_Short_Write_1P(0xCB,0x0C);	
Generic_Short_Write_1P(0xCC,0x0A);	
Generic_Short_Write_1P(0xCD,0x08);	
Generic_Short_Write_1P(0xCE,0x06);	
Generic_Short_Write_1P(0xCF,0x18);	
Generic_Short_Write_1P(0xD0,0x02);	
Generic_Short_Write_1P(0xD1,0x00);	
Generic_Short_Write_1P(0xD2,0x00);	
Generic_Short_Write_1P(0xD3,0x00);	
Generic_Short_Write_1P(0xD4,0x26);	
Generic_Short_Write_1P(0xD5,0x0F);	
Generic_Short_Write_1P(0xD6,0x0D);	
Generic_Short_Write_1P(0xD7,0x00);	
Generic_Short_Write_1P(0xD8,0x00);	
Generic_Short_Write_1P(0xD9,0x23);	
Generic_Short_Write_1P(0xDA,0x11);	
Generic_Short_Write_1P(0xDB,0x21);	
Generic_Short_Write_1P(0xDC,0x1F);	
Generic_Short_Write_1P(0xDD,0x1D);	
Generic_Short_Write_1P(0xDE,0x1B);	
Generic_Short_Write_1P(0xDF,0x0B);	
Generic_Short_Write_1P(0xE0,0x09);	
Generic_Short_Write_1P(0xE1,0x07);	
Generic_Short_Write_1P(0xE2,0x05);	
Generic_Short_Write_1P(0xE3,0x17);	
Generic_Short_Write_1P(0xE4,0x01);	
Generic_Short_Write_1P(0xE5,0x00);	
Generic_Short_Write_1P(0xE6,0x00);	
Generic_Short_Write_1P(0xE7,0x00);	
Generic_Short_Write_1P(0xB0,0x03);	
Generic_Short_Write_1P(0xBE,0x04);	
Generic_Short_Write_1P(0xB9,0x40);	
Generic_Short_Write_1P(0xCC,0x88);	
Generic_Short_Write_1P(0xC8,0x0C);	
Generic_Short_Write_1P(0xC9,0x07);	
Generic_Short_Write_1P(0xCD,0x01);	
Generic_Short_Write_1P(0xCA,0x40);	
Generic_Short_Write_1P(0xCE,0x1A);	
Generic_Short_Write_1P(0xCF,0x60);	
Generic_Short_Write_1P(0xD2,0x08);	
Generic_Short_Write_1P(0xD3,0x08);	
Generic_Short_Write_1P(0xDB,0x01);	
Generic_Short_Write_1P(0xD9,0x06);	
Generic_Short_Write_1P(0xD4,0x00);	
Generic_Short_Write_1P(0xD5,0x01);	
Generic_Short_Write_1P(0xD6,0x04);	
Generic_Short_Write_1P(0xD7,0x03);	
Generic_Short_Write_1P(0xC2,0x00);	
Generic_Short_Write_1P(0xC3,0x0E);	
Generic_Short_Write_1P(0xC4,0x00);	
Generic_Short_Write_1P(0xC5,0x0E);	
Generic_Short_Write_1P(0xDD,0x00);	
Generic_Short_Write_1P(0xDE,0x0E);	
Generic_Short_Write_1P(0xE6,0x00);	
Generic_Short_Write_1P(0xE7,0x0E);	
Generic_Short_Write_1P(0xC2,0x00);	
Generic_Short_Write_1P(0xC3,0x0E);	
Generic_Short_Write_1P(0xC4,0x00);	
Generic_Short_Write_1P(0xC5,0x0E);	
Generic_Short_Write_1P(0xDD,0x00);	
Generic_Short_Write_1P(0xDE,0x0E);	
Generic_Short_Write_1P(0xE6,0x00);	
Generic_Short_Write_1P(0xE7,0x0E);	
Generic_Short_Write_1P(0xB0,0x06);	
Generic_Short_Write_1P(0xC0,0xA5);	
Generic_Short_Write_1P(0xD5,0x1C);	
Generic_Short_Write_1P(0xC0,0x00);	
Generic_Short_Write_1P(0xB0,0x00);
Generic_Short_Write_1P(0xBD,0x30);//VCOM	  37
	
Generic_Short_Write_1P(0xF9,0x5C);	
Generic_Short_Write_1P(0xC2,0x14);	
Generic_Short_Write_1P(0xC4,0x14);	
Generic_Short_Write_1P(0xBF,0x15);	
Generic_Short_Write_1P(0xC0,0x0C);	


Generic_Short_Write_1P(0xB0,0x00);
Generic_Short_Write_1P(0xB1,0x79);
Generic_Short_Write_1P(0xBA,0x8F);//

     DCS_Short_Write_NP(0x11);	
     Delay(200);
     DCS_Short_Write_NP(0x29);
     Delay(50);
```

## 4.1 Analyze the MIPI Panel Initialization Code Provided by the Customer:

```bash
Generic_Short_Write_1P(0xBA,0x8F);
Generic_Short_Write_1P indicates: Send Generic command with 1 parameter, data byte length is 2 (Generic Short Write, 1 parameter)
0xBA: Register address 
0x8F: Data 1 is 0x8F
```

- **Convert to MIPI panel initialization format on Rockchip DTS:******

```bash
13 00 02 BA 8F

Analysis:
13 indicates 0x13 data type command
00 indicates no delay
02 indicates two data bytes: 0xBA, 0x8F
BA is register address 0xBA
8F is data 0x8F
```
- **Analyze customer's MIPI panel initialization code:**

```bash
 DCS_Short_Write_NP(0x11);	
 Delay(200);
 DCS_Short_Write_NP(0x29);
 Delay(50);
 
Analysis:
DCS_Short_Write_NP means: Only send DCS command, no parameters, data byte length is 1 (DCS Short Write, no parameters)
0x11: Data 1 is 0x11
Delay(200): Delay 200 ms
```

- **Convert to MIPI panel initialization format on RK DTS:**

```bash
05 C8 01 11
05 32 01 29

Analysis:
05 indicates 0x05 data type command
C8 indicates delay of 200 in hex is 0xC8
01 indicates 1 data byte: 0x11
11 is the data: 0x11
```
## 4.2 **Convert the Panel Vendor's Initialization Code to Rockchip Platform's Power up Sequence DTS Configuration:**
```xml
panel-init-sequence = [
			13 00 02 B0 01
			13 00 02 C0 26
			13 00 02 C1 10
			13 00 02 C2 0E
			13 00 02 C3 00
			13 00 02 C4 00
			13 00 02 C5 23
			13 00 02 C6 11
			13 00 02 C7 22
			13 00 02 C8 20
			13 00 02 C9 1E
			13 00 02 CA 1C
			13 00 02 CB 0C
			13 00 02 CC 0A
			13 00 02 CD 08
			13 00 02 CE 06
			13 00 02 CF 18
			13 00 02 D0 02
			13 00 02 D1 00
			13 00 02 D2 00
			13 00 02 D3 00
			13 00 02 D4 26
			13 00 02 D5 0F
			13 00 02 D6 0D
			13 00 02 D7 00
			13 00 02 D8 00
			13 00 02 D9 23
			13 00 02 DA 11
			13 00 02 DB 21
			13 00 02 DC 1F
			13 00 02 DD 1D
			13 00 02 DE 1B
			13 00 02 DF 0B
			13 00 02 E0 09
			13 00 02 E1 07
			13 00 02 E2 05
			13 00 02 E3 17
			13 00 02 E4 01
			13 00 02 E5 00
			13 00 02 E6 00
			13 00 02 E7 00
			13 00 02 B0 03
			13 00 02 BE 04
			13 00 02 B9 40
			13 00 02 CC 88
			13 00 02 C8 0C
			13 00 02 C9 07
			13 00 02 CD 01
			13 00 02 CA 40
			13 00 02 CE 1A
			13 00 02 CF 60
			13 00 02 D2 08
			13 00 02 D3 08
			13 00 02 DB 01
			13 00 02 D9 06
			13 00 02 D4 00
			13 00 02 D5 01
			13 00 02 D6 04
			13 00 02 D7 03
			13 00 02 C2 00
			13 00 02 C3 0E
			13 00 02 C4 00
			13 00 02 C5 0E
			13 00 02 DD 00
			13 00 02 DE 0E
			13 00 02 E6 00
			13 00 02 E7 0E
			13 00 02 C2 00
			13 00 02 C3 0E
			13 00 02 C4 00
			13 00 02 C5 0E
			13 00 02 DD 00
			13 00 02 DE 0E
			13 00 02 E6 00
			13 00 02 E7 0E
			13 00 02 B0 06
			13 00 02 C0 A5
			13 00 02 D5 1C
			13 00 02 C0 00
			13 00 02 B0 00
			13 00 02 BD 30

			13 00 02 F9 5C
			13 00 02 C2 14
			13 00 02 C4 14
			13 00 02 BF 15
			13 00 02 C0 0C


			13 00 02 B0 00
			13 00 02 B1 79
			13 00 02 BA 8F

			05 C8 01 11
			05 32 01 29
		];

		panel-exit-sequence = [
			05 00 01 28
			05 00 01 10
		];
```

## ArmSoM Product Introduction： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM Technical Forum： [http://forum.armsom.org/](http://forum.armsom.org/)