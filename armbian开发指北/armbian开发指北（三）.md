# armbian开发指北（三）
前言：上篇带大家一步一步编译出了已支持板子的armbian固件，假如我们的开发板需要支持应该如何做呢，
带着这个疑问，我们开始看armbian的代码。

假设我们现在需要把armsom团队开发的[armsom-w3](http://wiki.armsom.org/index.php/ArmSoM-w3)

## 1. board.config 板级配置
```
$ armsom/armbian/build/config/boards
```
改目录下存放目前armbian支持的board，首先我们要建立自己的config
armsom-w3.config, show code no bb

```
# Rockchip RK3588 SoC octa core 8-32GB SoC 2.5GBe PoE eMMC USB3 NvME 
// 这行是我们执行编译脚本后，选择哪个boader后面的描述，可自行更改
BOARD_NAME="ArmSoM W3" 
// 板子名称
BOARDFAMILY="rockchip-rk3588" 
// rockchip-rk3588 文件在config/sources/families/rockchip-rk3588.conf，里面有详细的板子kernel，uboot下载链接，各种配置
BOARD_MAINTAINER="armsom-team"
// 板子的维护者名称
BOOTCONFIG="rock-5b-rk3588_defconfig"
// uboot的config
KERNEL_TARGET="legacy,edge,midstream,collabora"
// kernel支持哪些版本
KERNEL_TEST_TARGET="legacy" # in case different then kernel target
FULL_DESKTOP="yes"
BOOT_LOGO="desktop"
BOOT_FDT_FILE="rockchip/rk3588-rock-5b.dtb"
BOOT_SCENARIO="spl-blobs"
BOOT_SUPPORT_SPI="yes"
BOOT_SPI_RKSPI_LOADER="yes"
IMAGE_PARTITION_TABLE="gpt"
SKIP_BOOTSPLASH="yes" # Skip boot splash patch, conflicts with CONFIG_VT=yes
BOOTFS_TYPE="ext4"

function post_family_tweaks__rock5b_naming_audios() {
	display_alert "$BOARD" "Renaming rock5b audios" "info"

	mkdir -p $SDCARD/etc/udev/rules.d/
	echo 'SUBSYSTEM=="sound", ENV{ID_PATH}=="platform-hdmi0-sound", ENV{SOUND_DESCRIPTION}="HDMI0 Audio"' > $SDCARD/etc/udev/rules.d/90-naming-audios.rules
	echo 'SUBSYSTEM=="sound", ENV{ID_PATH}=="platform-hdmi1-sound", ENV{SOUND_DESCRIPTION}="HDMI1 Audio"' >> $SDCARD/etc/udev/rules.d/90-naming-audios.rules
	echo 'SUBSYSTEM=="sound", ENV{ID_PATH}=="platform-hdmiin-sound", ENV{SOUND_DESCRIPTION}="HDMI-In Audio"' >> $SDCARD/etc/udev/rules.d/90-naming-audios.rules
	echo 'SUBSYSTEM=="sound", ENV{ID_PATH}=="platform-dp0-sound", ENV{SOUND_DESCRIPTION}="DP0 Audio"' >> $SDCARD/etc/udev/rules.d/90-naming-audios.rules
	echo 'SUBSYSTEM=="sound", ENV{ID_PATH}=="platform-es8316-sound", ENV{SOUND_DESCRIPTION}="ES8316 Audio"' >> $SDCARD/etc/udev/rules.d/90-naming-audios.rules

	return 0
}
```

以上boards配置均可在config/boards/README.md文件查看详细解释



lib/functions/artifacts/artifacts-obtain.sh
去掉413行的else，确保每次都会拉kernel uboot到 cache目录，方便开发
artifact_exists_in_remote_cache=no


查看单独编译kernel命令，output里面的log，

把sd卡固件烧录到emmc
nand-sata-install