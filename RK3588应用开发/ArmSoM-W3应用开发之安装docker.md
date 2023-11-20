

# 1. 简介

- [RK3588从入门到精通](https://blog.csdn.net/nb124667390/article/details/130725546)系列专题
- 开发板：ArmSoM-W3
- Kernel：5.10.160
- OS：Debian11
- 本⽂介绍ArmSoM-W3在Debian11下如何安装使用docker



# 2.Rockchip 平台系统运行docker

 Docker运行对内核配置有要求，需要 kernel 开启 cgroups、namespace、netfilter、overlayfs 等功能的⽀持，这些配置打开才满足docker运行的要求。

ArmSoM发布的普通固件一般不满足 Docker 的运行要求，如果有需求可以用我们配置过的内核固件，或者按照下文自己配置。



## 2.1 kernel配置

Docker开源团队提供了一个[检测脚本](https://github.com/moby/moby/blob/master/contrib/check-config.sh)，用以检测内核配置是否符合Docker运行的要求，下载脚本到SDK源码kernel目录下。

输入如下命令：

```
chmod 777 check-config.sh

./check-config.sh .config
```

*注意：.config需要在内核配置完后才会生成*

得到如下打印：

```
lhd@ydtx:~/project_code/3588/3588_linux5.10_v1.0.5/kernel$ ./check-config.sh .config
info: reading kernel config from .config ...

Generally Necessary:
- cgroup hierarchy: cgroupv2
  Controllers:
  - cpu: available
  - cpuset: available
  - io: available
  - memory: available
  - pids: available
- apparmor: enabled and tools installed
- CONFIG_NAMESPACES: enabled
- CONFIG_NET_NS: enabled
- CONFIG_PID_NS: enabled
- CONFIG_IPC_NS: enabled
- CONFIG_UTS_NS: enabled
- CONFIG_CGROUPS: enabled
- CONFIG_CGROUP_CPUACCT: enabled
- CONFIG_CGROUP_DEVICE: enabled
- CONFIG_CGROUP_FREEZER: enabled
- CONFIG_CGROUP_SCHED: enabled
- CONFIG_CPUSETS: enabled
- CONFIG_MEMCG: enabled
- CONFIG_KEYS: enabled
- CONFIG_VETH: enabled
- CONFIG_BRIDGE: enabled
- CONFIG_BRIDGE_NETFILTER: enabled
- CONFIG_IP_NF_FILTER: enabled
- CONFIG_IP_NF_TARGET_MASQUERADE: enabled
- CONFIG_NETFILTER_XT_MATCH_ADDRTYPE: enabled
- CONFIG_NETFILTER_XT_MATCH_CONNTRACK: enabled
- CONFIG_NETFILTER_XT_MATCH_IPVS: enabled
- CONFIG_NETFILTER_XT_MARK: enabled
- CONFIG_IP_NF_NAT: enabled
- CONFIG_NF_NAT: enabled
- CONFIG_POSIX_MQUEUE: enabled
- CONFIG_CGROUP_BPF: enabled

Optional Features:
- CONFIG_USER_NS: enabled
- CONFIG_SECCOMP: enabled
- CONFIG_SECCOMP_FILTER: enabled
- CONFIG_CGROUP_PIDS: enabled
- CONFIG_MEMCG_SWAP: enabled
    (cgroup swap accounting is currently enabled)
- CONFIG_BLK_CGROUP: enabled
- CONFIG_BLK_DEV_THROTTLING: missing
- CONFIG_CGROUP_PERF: enabled
- CONFIG_CGROUP_HUGETLB: missing
- CONFIG_NET_CLS_CGROUP: enabled (as module)
- CONFIG_CGROUP_NET_PRIO: missing
- CONFIG_CFS_BANDWIDTH: enabled
- CONFIG_FAIR_GROUP_SCHED: enabled
- CONFIG_RT_GROUP_SCHED: missing
- CONFIG_IP_NF_TARGET_REDIRECT: enabled (as module)
- CONFIG_IP_VS: enabled
- CONFIG_IP_VS_NFCT: enabled
- CONFIG_IP_VS_PROTO_TCP: enabled
- CONFIG_IP_VS_PROTO_UDP: enabled
- CONFIG_IP_VS_RR: enabled (as module)
- CONFIG_SECURITY_SELINUX: missing
- CONFIG_SECURITY_APPARMOR: missing
- CONFIG_EXT4_FS: enabled
- CONFIG_EXT4_FS_POSIX_ACL: enabled
- CONFIG_EXT4_FS_SECURITY: enabled
- Network Drivers:
  - "overlay":
    - CONFIG_VXLAN: enabled (as module)
    - CONFIG_BRIDGE_VLAN_FILTERING: enabled
      Optional (for encrypted networks):
      - CONFIG_CRYPTO: enabled
      - CONFIG_CRYPTO_AEAD: enabled
      - CONFIG_CRYPTO_GCM: enabled
      - CONFIG_CRYPTO_SEQIV: enabled (as module)
      - CONFIG_CRYPTO_GHASH: enabled
      - CONFIG_XFRM: enabled
      - CONFIG_XFRM_USER: enabled
      - CONFIG_XFRM_ALGO: enabled
      - CONFIG_INET_ESP: enabled (as module)
  - "ipvlan":
    - CONFIG_IPVLAN: enabled (as module)
  - "macvlan":
    - CONFIG_MACVLAN: enabled (as module)
    - CONFIG_DUMMY: enabled (as module)
  - "ftp,tftp client in container":
    - CONFIG_NF_NAT_FTP: enabled (as module)
    - CONFIG_NF_CONNTRACK_FTP: enabled (as module)
    - CONFIG_NF_NAT_TFTP: enabled (as module)
    - CONFIG_NF_CONNTRACK_TFTP: enabled (as module)
- Storage Drivers:
  - "aufs":
    - CONFIG_AUFS_FS: missing
  - "btrfs":
    - CONFIG_BTRFS_FS: missing
    - CONFIG_BTRFS_FS_POSIX_ACL: missing
  - "devicemapper":
    - CONFIG_BLK_DEV_DM: enabled (as module)
    - CONFIG_DM_THIN_PROVISIONING: enabled (as module)
  - "overlay":
    - CONFIG_OVERLAY_FS: enabled (as module)
  - "zfs":
    - /dev/zfs: present
    - zfs command: missing
    - zpool command: missing

Limits:
- /proc/sys/kernel/keys/root_maxkeys: 1000000
```

Generally Necessary是内核必须配置项，Optional Features是可选配置项

如果检测Generally Necessary下面的结果是missing或者enabled (as module)，都可以去对应配置那设置为Y。



## 2.2 Debian 配置

 Debian 默认使⽤ iptables-nft，⽽ docker 默认使⽤ iptableslegacy，故需要配置 iptables 使⽤ legacy 版本，可以通过以下命令进⾏切换：

```
# 使⽤ iptables-legacy
update-alternatives --set iptables /usr/sbin/iptables-legacy
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
# 使⽤ iptables-nft
update-alternatives --set iptables /usr/sbin/iptables-nft
update-alternatives --set ip6tables /usr/sbin/ip6tables-nft
```



## 2.3 安装Docker

在RK3588上安装Docker，按照以下步骤进行操作：

1. **更新系统：** 

   在开始安装Docker之前，确保系统是最新的。运行以下命令：

   ```
   sudo apt update
   sudo apt upgrade
   ```

2. **安装依赖项：** 

   安装Docker所需的一些依赖项：

   ```
   sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
   ```

3. **添加Docker官方GPG密钥：** 

   通过添加Docker官方的GPG密钥来信任官方存储库：

   ```
   curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

4. **设置Docker存储库：** 

   添加Docker存储库到APT源列表中：

   ```
   echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. **安装Docker引擎：** 

   更新APT软件包索引并安装Docker引擎：

   ```
   sudo apt update
   sudo apt install docker-ce docker-ce-cli containerd.io
   ```

6. **启动Docker服务：** 

   安装完成后，启动Docker服务：

   ```
   sudo systemctl start docker
   ```

   还可以将Docker设置为在系统启动时自动启动：

   ```
   sudo systemctl enable docker
   ```

7. **验证安装：** 

   运行以下命令以验证Docker是否正确安装：

   ```
   sudo docker pull hello-world
   sudo docker run hello-world
   ```



如果一切顺利，应该能够看到hello-world容器成功运行。

