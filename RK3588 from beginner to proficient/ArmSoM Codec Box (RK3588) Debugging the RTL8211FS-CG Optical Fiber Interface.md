# **ArmSoM Codec Box (RK3588)**: Debugging the RTL8211FS-CG Optical Fiber Interface

## 1. Introduction

- [Mastering RK3588 from Beginner to Expert] Series Contents
- This article summarizes the debugging of the RTL8211FS-CG  optical fiber interface based on the RK3588 platform, SDK version :RK3588_ANDROID 12.0.
- Video bridge chip: RTL8211FS-CG
- Driver code: "kernel/drivers/net/phy/realtek.c" 
- Debugging functional scheme : Using the RTL8211FS-CG on RK3588 to convert the GMAC Ethernet interface to an optical fiber interface.

## 2. Hardware

The hardware engineer designs according to the RTL8211FS-CG reference layout.

Here are some screenshots:

![RTL8211FN](https://img-blog.csdnimg.cn/73211b316e054f3391082109e1072c06.png#pic_center)
![SFP](https://img-blog.csdnimg.cn/496168b03a59470ba2aee071a32100d8.png#pic_center)

During the design process, refer to the reference design provided by Realtek and consult the PHY chip vendor for any considerations.

Notes: The 3.3V external power supply voltage for the 8211FS must align with the electrical level of the GMAC1 controller on RK3588.

Use UTP<->RGMII, and configure CFG_MODE2 :0 =010 to support both optical fiber and electrical interfaces.

## 3. Software

It is recommended to debug the RJ45 electrical port first and then connect the optical fiber interface, which may be easier to debug. 

When debugging the electrical port, first plug in a 100Mbps ethernet cable to debug 100Mbps, then switch to a Gigabit ethernet cable after succeeding.

### 3.1 Code

#### 3.1.1 Realtek PHY Kernel Configuration

Enter make menuconfig under kernel:

```
Device Drivers ---> 

[*] Network device support --->

-*- PHY Device support and infrastructure --->

[*] Realtek PHYs
```

This will compile realtek.c into the kernel.

#### 3.1.2 DTS Configuration

```
&gmac1 {

  /* Use rgmii-rxid mode to disable rx delay inside SoC */

  phy-mode = "rgmii-rxid";

  clock_in_out = "output";

  snps,reset-gpio = <&gpio3 RK_PB7 GPIO_ACTIVE_LOW>;

  snps,reset-active-low;

  /* Reset time is 20ms, 100ms for rtl8211f */

  snps,reset-delays-us = <0 20000 100000>;

  

  pinctrl-names = "default";

  pinctrl-0 = <&gmac1_miim

           &gmac1_tx_bus2

           &gmac1_rx_bus2

           &gmac1_rgmii_clk

           &gmac1_rgmii_bus

           &gmac1_clkinout>;



  tx_delay = <0x43>;

  /* rx_delay = <0x4f>; */

  

  phy-handle = <&rgmii_phy1>;

  status = "okay";  

};



&mdio1 {

  rgmii_phy1: phy@1 {

    compatible = "ethernet-phy-ieee802.3-c22";

    reg = <0x1>;

  };

};
```

#### 3.1.3 Code Verification

Plug in a 1Gbps ethernet cable, related prints exist and ping baidu works.

At this point, cannot get IP when plugging in optical fiber interface.

#### 3.1.4 Debugging 

Turn on IO debug commands:

CONFIG_DEVMEM=y

#### 3.1.5 Operating Registers

```
find /sys -name phy_registers //find ethernet register node and enter directory

echo 31 0xdc0 > phy_registers //switch to PHY PAGE 0xdc0

cat phy_registers //Read the current PAGE register values to verify if PHYID1 and PHYID2 are correct to confirm if the registers are accessed properly.

echo 0 0x value > phy_registers //set PAGE0 register 0 to needed value

cat phy_registers //read value to check if modified successfully 

echo 31 0 > phy_registers //switch back to PAGE0! Important to switch back for changes to take effect
```

If modification is invalid, please refer to section 8.5 of PHY datasheet to modify.

#### 3.1.6 Force Fixed Fiber Mode

setup_fiber_mode 

```
#endif
+static int phy_8211fS_fiber_mode_fixup(struct phy_device *phydev)
+{	int i;
+	printk("%s in\n", __func__);
+	phy_write(phydev, 31, 0xdc0 );
+   phy_write(phydev, 16, 0x79ad );
+   //phy_write(phydev, 20, 0x79ad );
+	printk("page 0xdc0 register\n");
+	for(i =0; i<8,i++)
+		printk("%d: %x\n",i,phy_read(phydev,i));
+    phy_write(phydev, 31, 0xdc1 );
+   printk("23: %x\n", phy_read(phydev,23));
+   phy_write(phydev, 31, 0x0 );
+   printk("page 0 register\n");
+	for(i =0; i<32,i++)
+		printk("%d: %x\n",i,phy_read(phydev,i));
+	
+	return 0;
+}
/**
 * stmmac_dvr_probe
 * @device: device pointer
#ifdef CONFIG_DWMAC_RK_AUTO_DELAYLINE
	INIT_DELAYED_WORK(&priv->scan_dwork, stmmac_scan_delayline_dwork);
#endif
+	ret = phy_register_fixup_for_uid(RTL_8211FS_PHY_ID, 0xffffffff, +phy_8211fS_fiber_mode_fixup);
+	if (ret)
+		pr_warn("Cannot register PHY board fixup.\n");
	return ret;
error_netdev_register:
```

After compiling and flashing, the network port light status changes to fiber mode, but still cannot get IP when plugging in optical fiber interface.

#### 3.1.7 Patch

```
--- a/kernel/drivers/net/phy/realtek.c
+++ b/kernel/drivers/net/phy/realtek.c
@@ -46,6 +46,11 @@
 #define RTL8366RB_POWER_SAVE			0x15
 #define RTL8366RB_POWER_SAVE_ON			BIT(12)
 
+#define RTL8211FS_FIBER_ESR			0x0F
+#define RTL8211FS_MODE_MASK			0xC000
+#define RTL8211F_MODE_COPPER		0
+#define RTL8211FS_MODE_FIBER		1
+
 #define RTL_SUPPORTS_5000FULL			BIT(14)
 #define RTL_SUPPORTS_2500FULL			BIT(13)
 #define RTL_SUPPORTS_10000FULL			BIT(0)
@@ -58,6 +63,10 @@
 
 #define RTL_GENERIC_PHYID			0x001cc800
 
+struct rtl8211f_priv {
+	int lastmode;
+};
+
 MODULE_DESCRIPTION("Realtek PHY driver");
 MODULE_AUTHOR("Johnson Leung");
 MODULE_LICENSE("GPL");
@@ -93,7 +102,6 @@ static int rtl821x_ack_interrupt(struct phy_device *phydev)
 static int rtl8211f_ack_interrupt(struct phy_device *phydev)
 {
 	int err;
-
 	err = phy_read_paged(phydev, 0xa43, RTL8211F_INSR);
 
 	return (err < 0) ? err : 0;
@@ -140,7 +148,6 @@ static int rtl8211e_config_intr(struct phy_device *phydev)
 static int rtl8211f_config_intr(struct phy_device *phydev)
 {
 	u16 val;
-
 	if (phydev->interrupts == PHY_INTERRUPT_ENABLED)
 		val = RTL8211F_INER_LINK_STATUS;
 	else
@@ -242,7 +249,7 @@ static int rtl8211f_config_init(struct phy_device *phydev)
 			"2ns RX delay was already %s (by pin-strapping RXD0 or bootloader configuration)\n",
 			val_rxdly ? "enabled" : "disabled");
 	}
-
+	
 	return 0;
 }
 
@@ -560,6 +567,89 @@ static int rtlgen_resume(struct phy_device *phydev)
 	return ret;
 }
 
+static int rtl8211f_probe(struct phy_device *phydev)
+{
+	struct device *dev = &phydev->mdio.dev;
+	struct rtl8211f_priv *priv;
+
+	priv = devm_kzalloc(dev, sizeof(struct rtl8211f_priv), GFP_KERNEL);
+	if (!priv)
+		return -ENOMEM;
+	
+	phydev->priv = priv;
+
+	return 0;
+}
+
+static void rtl8211f_remove(struct phy_device *phydev)
+{
+	struct device *dev = &phydev->mdio.dev;
+	struct rtl8211f_priv *priv = phydev->priv;
+
+	if (priv)
+		devm_kfree(dev, priv);
+}
+
+static int rtl8211f_mode(struct phy_device *phydev)
+{
+	u16 val;
+
+	val = phy_read(phydev, RTL8211FS_FIBER_ESR);
+	val &= RTL8211FS_MODE_MASK;
+
+	if(val)
+		return RTL8211FS_MODE_FIBER;
+	else
+		return RTL8211F_MODE_COPPER;
+}
+
+static int rtl8211f_config_aneg(struct phy_device *phydev)
+{
+	int ret;
+
+	struct rtl8211f_priv *priv = phydev->priv;
+
+	ret = genphy_read_abilities(phydev);
+	if(ret < 0)
+		return ret;
+
+	linkmode_copy(phydev->advertising, phydev->supported);
+
+	if (rtl8211f_mode(phydev) == RTL8211FS_MODE_FIBER) {
+		dev_info(&phydev->mdio.dev, "Fiber Mode");
+		priv->lastmode = RTL8211FS_MODE_FIBER;
+		return genphy_c37_config_aneg(phydev);
+	}
+
+	dev_info(&phydev->mdio.dev, "Copper Mode");
+
+	priv->lastmode = RTL8211F_MODE_COPPER;
+
+	return genphy_config_aneg(phydev);
+}
+
+static int rtl8211f_read_status(struct phy_device *phydev)
+{
+	int ret;
+	struct rtl8211f_priv *priv = phydev->priv;
+
+	if(rtl8211f_mode(phydev) != priv->lastmode) {
+		ret = rtl8211f_config_aneg(phydev);
+		if(ret < 0)
+			return ret;
+
+		ret = genphy_restart_aneg(phydev);
+		if(ret < 0)
+			return ret;
+	}
+
+	if (rtl8211f_mode(phydev) == RTL8211FS_MODE_FIBER)
+		return genphy_c37_read_status(phydev);
+
+	return genphy_read_status(phydev);
+}
+
+
 static struct phy_driver realtek_drvs[] = {
 	{
 		PHY_ID_MATCH_EXACT(0x00008201),
@@ -632,10 +722,15 @@ static struct phy_driver realtek_drvs[] = {
 		.write_page	= rtl821x_write_page,
 	}, {
 		PHY_ID_MATCH_EXACT(0x001cc916),
-		.name		= "RTL8211F Gigabit Ethernet",
+		// .name		= "RTL8211F Gigabit Ethernet",
+		.name		= "RTL8211F(S) Gigabit Ethernet",
+		.probe		= rtl8211f_probe,
+		.remove		= rtl8211f_remove,		
 		.config_init	= &rtl8211f_config_init,
 		.ack_interrupt	= &rtl8211f_ack_interrupt,
 		.config_intr	= &rtl8211f_config_intr,
+		.config_aneg	= rtl8211f_config_aneg,
+		.read_status	= rtl8211f_read_status,		
 		.suspend	= genphy_suspend,
 		.resume		= rtl821x_resume,
 		.read_page	= rtl821x_read_page,
```

With the patch added, resetting the reset pin once more can recognize the optical fiber interface after powering on.

#### 3.1.8 PHY Reset

io -4  0xFEC40000 0x80000000  Pull down

sleep 0.1

io -4  0xFEC40000 0x80008000  Pull up

RTL8211F(S) Gigabit Ethernet stmmac-1:01: Copper Mode

**Summary**

As long as the hardware is configured in RGMII to Fiber mode, RTL8211FS will work. Fiber mode configuration can refer to the register descriptions in the RTL8211FS datasheet. It is recommended to communicate with Realtek to coordinate resolving the issue.

![img](https://img-blog.csdnimg.cn/ae342e2c968a4e0f9740307b8cfa2af8.jpeg)

## ArmSoM Product Introduction： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)

## ArmSoM Technical Forum： http://forum.armsom.org/