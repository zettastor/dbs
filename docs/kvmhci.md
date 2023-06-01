在 KVM 超融合部署方案中，每台服务器既作为计算节点，承担业务负载，同时也作为数据存储节点，对外提供数据存储服务。其优点在于架构更加灵活敏捷，用户只需按一定配置向系统内补充节点，即可实现计算和存储能力的同步横向扩展，同时可实现更高的资源利用率，从而降低总体成本。

<img src="https://zdbs.io/vitualization/media/hci1.png" width="50%" />

在本文中，我们将介绍在物理服务器中安装 KVM，并在其中部署 DBS 块存储软件。

## 0. 环境要求
- 1 台具有局域网连接的 Linux 服务器（下文简称“物理服务器”）  
为了方便说明，假设 IP 为 192.168.142.128

>**注意**  
下列命令假设您已经具有足够权限，关于使用 `su` 或 `sudo` 等提权操作不再赘述。

## 1. 在物理服务器中安装 KVM
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
1. 在开始安装 KVM 之前，请通过 egrep 命令检查物理服务器的 CPU 是否支持硬件虚拟化：
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

### 4.3 虚拟机资源需求

在超融合部署方案中，DBS 节点部署在 KVM 虚拟机中。

- 在默认配置下，每个节点至少需要 **128GB内存**；最小化验证部署配置下，每个节点需要 **8GB内存**。
- 每个存储节点除了操作系统盘之外，还需要配置至少 **1块1TB空白硬盘**。

### 4.4 创建 CentOS 7 虚拟机 

1. 运行 Virtual Machine Manager 创建虚拟机：
```
virt-manager
```
2. 点击 `File - New Virtual Machine`

<img src="https://zdbs.io/vitualization/media/kvm01.png" width="33%" />

3. 在打开的对话框中，选择使用 ISO 镜像安装 VM 的选项。然后点击 `Forward`。

<img src="https://zdbs.io/vitualization/media/kvm02.png" width="33%" />
<img src="https://zdbs.io/vitualization/media/kvm03.png" width="33%" />

4. 输入希望分配给虚拟机的 RAM 数量和 CPU 数量，然后点击 `Forward`。

<img src="https://zdbs.io/vitualization/media/kvm04.png" width="33%" />

5. 输入希望分配给虚拟机的系统盘空间，然后点击 `Forward`。

<img src="https://zdbs.io/vitualization/media/kvm05.png" width="33%" />

6. 将所有物理服务器上的存储用磁盘透传分配给虚拟机。

7. 请为您的虚拟机指定名称，然后单击 `Finish` 以完成设置。

<img src="https://zdbs.io/vitualization/media/kvm09.png" width="33%" />

8. 虚拟机会自动启动，并提示您开始安装 ISO 文件中的操作系统。

<img src="https://zdbs.io/vitualization/media/kvm10.png" width="65%" />

## 2. 在虚拟机中安装 CentOS 7

参考 [安装操作系统](/docs/operatingsystem-zh.md)

## 3. 部署 ZettaStor DBS

在 CentOS 7 中部署 DBS 块存储软件，参考 [快速配置与部署](/INSTALL-zh.md)
