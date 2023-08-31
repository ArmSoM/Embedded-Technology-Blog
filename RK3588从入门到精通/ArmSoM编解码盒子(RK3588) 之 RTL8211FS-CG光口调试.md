# 1. 简介
- [[RK3588从入门到精通] 专栏总目录](https://blog.csdn.net/nb124667390/article/details/130725546)
- 本文是基于RK3588平台，SDK版本：RK3588_ANDROID12.0  RTL8211FS-CG光口调试总结。
- 视频桥接芯片：RTL8211FS-CG
- 驱动代码："kernel/drivers/net/phy/realtek.c"
- 本次调试的方案功能：RK3588 调试RTL8211FS-CG 转接出光口

# 2. 硬件部分


硬件工程师参考RTL8211FS-CG发布的设计图设计

以下为部分截图
![RTL8211FN](https://img-blog.csdnimg.cn/73211b316e054f3391082109e1072c06.png#pic_center)
![SFP](https://img-blog.csdnimg.cn/496168b03a59470ba2aee071a32100d8.png#pic_center)





在设计过程中参考realtek发过的参考设计，建议咨询一下phy厂家，看有哪些注意地方

注意： 8211FS使用外部3.3V，3.3V电平要与主控GMAC1接口电平相匹配；
使用UTP<->RGMII的接法，且CFG_MODE2：0＝010兼容光口和电口；

# 3. 软件部分
建议先调电口RJ45，调通后再接光口，可能更容易；调电口时先插百兆网线调百兆，成功后再换千兆网线

## 3.1 代码
### 3.1.1 Realtek phy的内核配置
在kernel下输入make menuconfig
```bash
Device Drivers  --->

[*] Network device support  --->

-*-   PHY Device support and infrastructure  --->

 -*-   Realtek PHYs
```
这样realtek.c就可以编译到kernel了

### 3.1.2 dts配置
```bash
&gmac1 {
    /* Use rgmii-rxid mode to disable rx delay inside Soc */
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
### 3.1.3 代码验证

插千兆网线有相关打印且可以ping通百度

此时插光口没有分配IP地址

### 3.1.4 调试
打开IO调试命令

CONFIG_DEVMEM=y

### 3.1.5 操作寄存器
```
find /sys -name phy_registers //先找到以太网寄存器的配置节点并进入所在目录

echo 31 0xdc0 >phy_registers //切换到PHY的PAGE 0xdc0

cat phy_registers //读取当前PAGE寄存器的值,核对PHYID1,PHYID2是否正确来确认寄存器是否正确

echo 0 0x value >phy_registers //改写PAGE0第0个寄存器的值为需要的value

cat phy_registers //读取值检查是否修改成功

echo 31 0 > phy_registers //切回到PAGE0寄存器！！！！重要，一定写完要切回来才会生效
```
如果修改无效，参阅PHY规格书的8.5章节修改

### 3.1.6 将芯片强制固定光口模式 
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
编译烧写之后网口灯状态已经变为光口模式了，此时插入光口还是无法分配IP地址

### 3.1.7 补丁
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
将补丁加进去

上电之后再复位reset脚一次可以识别到光口了

### 3.1.7 phy reset

io -4  0xFEC40000 0x80000000  拉低

sleep 0.1

io -4  0xFEC40000 0x80008000  拉高

RTL8211F(S) Gigabit Ethernet stmmac-1:01: Copper Mode

**总结**
RTL8211FS，只要硬件线路配置为RGMII to Fiber 等涉及到Fiber的模式，即可工作。Fiber相关的模式设定可参考RTL8211FS数据手册中寄存器描述，公司推动 realtek 跟进解决。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ae342e2c968a4e0f9740307b8cfa2af8.jpeg)
## ArmSoM 产品介绍： [http://wiki.armsom.org/index.php/ArmSoM-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)
## ArmSoM 技术论坛： [http://forum.armsom.org/](http://forum.armsom.org/)