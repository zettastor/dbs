iSCSI 协议是一种将 SCSI 协议扩展到 TCP/IP 网络的技术，可以实现在 IP 网络上运行块存储设备。

## 0. 环境要求

在本文中，我们将搭建一个 iSCSI 共享存储环境：
- 在 ZettaStor DBS 中安装并配置 iSCSI Target 服务，创建虚拟磁盘并分配给目标
- 在 Linux 客户机上安装并配置 iSCSI Initiator，发现并连接到 iSCSI Target 服务，并将虚拟磁盘格式化为本地分区或文件系统

| IP地址        | 系统           | 角色  |
| ------------- |:-------------:| -----:|
| 192.168.142.128 | ZettaStor DBS | iSCSI Target 服务，下文简称“DBS” |
| 192.168.142.130 | Linux | iSCSI initiator，下文简称“客户机” |

>**注意**  
下列命令假设您已经具有足够权限，关于使用 `su` 或 `sudo` 等提权操作不再赘述。

## 1. 在客户机上获取 iSCSI 限定名
### CentOS/RHEL/Fedora
```bash
yum install iscsi-initiator-utils
```

### Ubuntu/Debian
```bash
apt-get install open-iscsi
```

启用并检查 `iscsid` 服务
```bash
systemctl start iscsid
systemctl enable iscsid
systemctl status iscsid
```

接下来，查看客户机 IQN(iSCSI Qualified Name, iSCSI 限定名)
```
$ cat /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.1994-05.com.redhat:c341717a8db
```

这个 IQN 将在 [步骤2.7 创建访问控制并授权](#27-创建访问控制并授权) 中被使用。

## 2. 在 DBS 上配置 iSCSI 服务
以下业务开通流程是在 DBS 上配置 iSCSI 服务所需的最小设置步骤，列表中提供了该操作在用户手册中对应的链接。有关高级配置选项的详细信息，建议您查阅 [用户手册](/manual) 并进行相应配置以获得最佳性能和安全性。

### 2.1 [创建存储域](/manual#创建域)
### 2.2 [添加存储节点](/manual#添加和移除存储节点)
### 2.3 [创建存储池](/manual#创建存储池)
### 2.4 [添加存储磁盘](/manual#存储池磁盘扩容和减容)
### 2.5 [创建存储卷](/manual#创建卷)
### 2.6 [创建存储驱动](/manual#挂载驱动)
其中，“驱动类型”选择 iSCSI 驱动
### 2.7 [创建访问控制并授权](/manual#访问控制管理)
此步骤需要填写 [步骤1 在客户机上安装 iSCSI Initiator](#1-在客户机上获取-iscsi-限定名) 中客户机的 IQN

## 3. 在客户机上使用 iSCSI 服务
### 3.1 连接 iSCSI 服务
现在，我们需要在客户机上发现 iSCSI 服务：
```
$ iscsiadm -m discovery -t st -p 192.168.142.128
192.168.142.128:3260,1 iqn.2017-08.zettastor.iqn:1227055989086196745-0
```
该命令将在 DBS 上发现可用的 iSCSI 服务端并显示其 IQN。发现服务后，我们需要连接它。在客户端运行以下命令：
```
$ iscsiadm -m node --login
Logging in to [iface: default, target: iqn.2017-08.zettastor.iqn:1227055989086196745-0, portal: 192.168.142.128,3260] (multiple)
Login to [iface: default, target: iqn.2017-08.zettastor.iqn:1227055989086196745-0, portal: 192.168.142.128,3260] successful.
```

可以在客户机查看挂载的网络设备对应的盘符
```
$ ls -l /dev/disk/by-path/ip-*
```

### 3.2 挂载 iSCSI 驱动器
我们已经建立了与 iSCSI 目标的连接，挂载的驱动器现在可以在客户机上用作常规文件系统。我们需要对其进行初始化。例如，假设 iSCSI 对应的盘符为 `/dev/sdb`，需要对存储空间进行分区：
```
$ fdisk /dev/sdb
n⏎
p⏎
1⏎
⏎
⏎
w⏎
```

如果需要格式化该分区：
```
mkfs.ext4 /dev/sdb1
```

我们可以将其作为文件系统挂载到客户机上的`/mnt/iscsi`目录：
```bash
mkdir /mnt/iscsi
mount /dev/sdb1 /mnt/iscsi
```

如果要在系统启动时自动挂载，需要将挂载项添加到 `/etc/fstab`。添加 `_netdev` 选项以避免在网络初始化之前进行挂载。
```bash
uuid=$(blkid | grep "/dev/sdb1" | sed -e 's/.* UUID="\([^"]*\)".*/\1/g')
cat <<EOF | tee -a /etc/fstab
UUID="${uuid}" /mnt/iscsi ext4 _netdev 0 0
EOF
```