
cd SDK/kernel/driver
git clone https://github.com/hailo-ai/hailort-drivers.git
# or specify the version to download
git clone --depth 1 -b v4.xx.x https://github.com/hailo-ai/hailort-drivers.git
# Important!! confirm downloaded Hailort-Driver version
git -C hailort-drivers/ log -1  # tag: v4.xx.0 

echo "obj-y       += hailort-drivers/linux/pcie/" >> Makefile
cd ../../
./build.sh kernel

# check if hailo_pci.ko is builtfile kernel/drivers/hailort-drivers/linux/pcie/hailo_pci.ko
# Download the hailo8 firmware
./kernel/drivers/hailort-drivers/download_firmware.sh


sudo mkdir -p /usr/lib/modules/5.10.160-legacy-rk35xx/kernel/drivers/hailo
sudo cp hailo_pci.ko /usr/lib/modules/5.10.160-legacy-rk35xx/kernel/drivers/hailo
sudo cp modules.builtin /usr/lib/modules/5.10.160-legacy-rk35xx/
sudo cp modules.order /usr/lib/modules/5.10.160-legacy-rk35xx/
sudo mkdir -p /lib/firmware/hailo
sudo mv hailo8_fw.4.16.0.bin /lib/firmware/hailo/hailo8_fw.bin 
sudo cp 51-hailo-udev.rules /etc/udev/rules.d/
sudo depmod -a
sudo modprobe hailo_pci
sudo echo hailo_pci >> /etc/modules
sudo reboot


sudo mkdir -p /usr/lib/modules/5.10.160/kernel/drivers/hailo
sudo cp hailo_pci.ko /usr/lib/modules/5.10.160/kernel/drivers/hailo
sudo cp modules.builtin /usr/lib/modules/5.10.160/
sudo cp modules.order /usr/lib/modules/5.10.160/
sudo mkdir -p /lib/firmware/hailo
sudo mv hailo8_fw.4.16.0.bin /lib/firmware/hailo/hailo8_fw.bin 
sudo cp 51-hailo-udev.rules /etc/udev/rules.d/
sudo depmod -a
sudo modprobe hailo_pci
sudo echo hailo_pci >> /etc/modules
sudo reboot


sudo dpkg -i hailort_4.16.0_arm64.deb


hailortcli scan
hailortcli fw-control identify
wget https://hailo-model-zoo.s3.eu-west-2.amazonaws.com/ModelZoo/Compiled/v2.8.0/resnet_v1_18.hef
hailortcli run resnet_v1_18.hef

./compile.sh build BOARD=armsom-w3 BRANCH=legacy BUILD_DESKTOP=yes BUILD_MINIMAL=no DESKTOP_APPGROUPS_SELECTED='3dsupport browsers chat desktop_tools editors email internet multimedia office programming remote_desktop' DESKTOP_ENVIRONMENT=gnome DESKTOP_ENVIRONMENT_CONFIG_NAME=config_base KERNEL_CONFIGURE=no RELEASE=focal


sudo dmesg | grep "hailo"

chmod 777 run.sh
./run.sh

armbian ubuntu报错：
./rk_mpi_vdec_test: error while loading shared libraries: librockchip_mpp.so.1: cannot open shared object file: No such file or directory
./usr/lib/libgraphic_lsf.so 没找到

debian:
root@armsom-w3:/# sudo find -name "*mpp.so*"
./usr/lib/aarch64-linux-gnu/gstreamer-1.0/libgstrockchipmpp.so
./usr/lib/aarch64-linux-gnu/librockchip_mpp.so
./usr/lib/aarch64-linux-gnu/librockchip_mpp.so.0
./usr/lib/aarch64-linux-gnu/librockchip_mpp.so.1
./usr/lib/aarch64-linux-gnu/libv4l/plugins/libv4l-rkmpp.so

ulimit -Sn 4096

cd ak_rockit
#此demo设置的输出分辨率为1920x1080
xrandr –s 1920x1080
chmod 777 ak_rockit_demo
  ./ak_rockit_demo
