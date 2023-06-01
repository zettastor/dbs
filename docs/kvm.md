在分离部署方案中，使用多台物理机设备仅部署 Zettastor DBS，分布式存储对接的应用软件由其它的节点进行部署。这样就做到存储和计算之间的分离。这种方式下计算和存储之间相对独立，各自软件可以单独部署、扩容，技术上也相对简单。Zettastor DBS通过标准的 iSCSI 协议与各类软件进行对接。

<img src="https://zdbs.io/vitualization/media/hci2.png" width="65%" />

在本文中，我们将介绍在 ZettaStor DBS 中设置 iSCSI 服务并在客户机中使用其中的存储空间来部署 KVM 映像。

## 0. 环境要求
- 1 套 ZettaStor DBS 分布式存储设备（下文简称“DBS”）  
为了方便说明，假设 IP 为 192.168.142.128
- 1 台具有局域网连接的 Linux 客户机（下文简称“客户机”）

>**注意**  
下列命令假设您已经具有足够权限，关于使用 `su` 或 `sudo` 等提权操作不再赘述。

## 1. 在客户机上安装 iSCSI Initiator
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
这个 IQN 将在 [步骤2.7 创建访问控制并授权](/manual#访问控制管理) 中被使用。

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
此步骤需要填写 [步骤1 在客户机上安装 iSCSI Initiator](#1-在客户机上安装-iscsi-initiator) 中客户机的 IQN

## 3. 在客户机上使用 iSCSI 服务
### 3.1 发现 iSCSI 服务
现在，我们需要在客户机上发现 iSCSI 服务：
```
$ iscsiadm -m discovery -t st -p 192.168.142.128
192.168.142.128:3260,1 iqn.2017-08.zettastor.iqn:1227055989086196745-0
```
该命令将在 DBS 上发现可用的 iSCSI 服务端并显示其 IQN。

## 4. 在客户机上安装 KVM
### 4.1 安装KVM及相关软件包

#### CentOS/RHEL/Fedora
```
yum install qemu-kvm libvirt libvirt-python libguestfs-tools virt-install
```

#### Ubuntu/Debian 系统
```
apt install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager
```

启动并检查 libvirtd 服务：
```
systemctl enable libvirtd
systemctl start libvirtd
systemctl status libvirtd
```
如果一切正常运行，输出将返回 `active (running)` 状态。

### 4.2 检查硬件要求
1. 在开始安装 KVM 之前，请通过 egrep 命令检查您的 CPU 是否支持硬件虚拟化：
```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```
如果该命令返回值为0，则表示您的处理器不支持运行 KVM，其他大于 0 的数字都意味着您可以继续安装。如果你确定你的 CPU 支持虚拟化，请确保在服务器 BIOS 中未禁用此选项，一般位于 Intel Virtualization Technology 或 SVM MODE 选项。

2. 接下来，输入以下命令检查您的系统是否支持使用 KVM 加速：
```
$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
```

### 4.3 创建虚拟机 
1. 运行 Virtual Machine Manager 创建一个虚拟机：
```
virt-manager
```
2. 点击 `File - New Virtual Machine`

<img src="https://zdbs.io/vitualization/media/kvm01.png" width="50%" />

3. 在打开的对话框中，选择使用 ISO 镜像安装 VM 的选项，此处使用 CentOS 7 安装盘。然后点击 `Forward`。

<img src="https://zdbs.io/vitualization/media/kvm02.png" width="50%" />
<img src="https://zdbs.io/vitualization/media/kvm03.png" width="50%" />

4. 输入希望分配给虚拟机的 RAM 数量和 CPU 数量，然后点击 `Forward`。

<img src="https://zdbs.io/vitualization/media/kvm04.png" width="50%" />

5. 选择 `Select or create custom storage`，然后点击 `Manage`。

<img src="https://zdbs.io/vitualization/media/kvm06.png" width="50%" />

6. 点击左下角 `Add pool` 按钮，打开向导。

<img src="https://zdbs.io/vitualization/media/iscsi02.png" width="75%" />

7. 选择一个存储池的名称，此处需要来完成此菜单中的字段：

    - 将类型更改为 `iscsi`
    - 默认目标路径值 `/dev/disk/by-path/`，不建议编辑目标路径。
    - 输入 iSCSI 目标的主机名或IP地址，本示例使用 DBS 的地址 `192.168.142.128`
    - 在源 IQN 字段中，输入 DBS 的 IQN，本示例使用 [步骤3.1 发现 iSCSI 服务](#31-发现-iscsi-服务) 步骤中获得的 IQN，即 `iqn.2017-08.zettastor.iqn:1227055989086196745-0`

然后点击 `Finish`。

<img src="https://zdbs.io/vitualization/media/iscsi03.png" width="60%" />

8. 现在可以选择刚才创建的 iSCSI 目标，点击 `Choose Volume`。

<img src="https://zdbs.io/vitualization/media/kvm07.png" width="75%" />

9. 确认已挂载的 iSCSI 存储，然后点击 `Forward`。

<img src="https://zdbs.io/vitualization/media/kvm08.png" width="50%" />

10. 请为您的虚拟机指定名称，然后单击 `Finish` 以完成设置。

<img src="https://zdbs.io/vitualization/media/kvm09.png" width="50%" />

11. 虚拟机会自动启动，并提示您开始安装 ISO 文件中的操作系统。

<img src="https://zdbs.io/vitualization/media/kvm10.png" />
