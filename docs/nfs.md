网络文件系统（NFS）是一种分布式文件系统协议。它允许客户端计算机上的用户像访问本地存储一样访问计算机网络上的文件。它在 Unix 和 Linux 环境中被广泛用于文件共享，并提供了访问控制和身份验证等安全功能。NFS 的当前版本是 NFSv4。

## 1. 环境要求
- 1 套 ZettaStor DBS 分布式存储设备（下文简称“DBS”）  
为了方便说明，假设 IP 为 192.168.142.128
- 1 台具有局域网的 Linux 客户机 （下文简称“客户机”）  
为了方便说明，假设 IP 为 192.168.142.130
- 假设在客户机上，[Linux 环境 iSCSI 挂载](/docs/iscsiadm.md) 中描述的所有操作已经完成

本文仅包含设置一个可用的 NFS 服务器所需的最小步骤，如安装软件包、设置权限和导出共享等，此配置允许所有系统访问，这可能不适用于生产环境。根据实际要求，可能需要配置更高级的选项，例如用户身份验证和访问控制等。  

>**注意**  
下列命令假设您已经具有足够权限，关于使用 `su` 或 `sudo` 等提权操作不再赘述。  

## 2. 配置 NFS 服务
以下是设置 NFS 服务器的最小设置步骤：

### 2.1 安装 NFS 服务器
#### CentOS/RHEL/Fedora
```bash
yum install nfs-utils
```
#### Ubuntu/Debian
```bash
apt-get install nfs-common
```

### 2.2 为 NFS 共享目录设置权限
将从 DBS 挂载的目录配置为允许完全访问。
```bash
chmod -R 777 /mnt/iscsi
```

### 2.4 导出NFS共享
编辑 `/etc/exports` 文件并添加以下行以允许其他系统访问共享目录。
```
/mnt/iscsi *(rw,sync)
exportfs -a
```
这将把 `/mnt/iscsi` 目录导出到所有具有读写访问和同步权限的系统。

### 2.5 启动并启用 NFS 服务
启动并启用 NFS 服务以确保它们在开机时自动启动：
#### CentOS/RHEL/Fedora
```bash
systemctl start nfs-server
systemctl enable nfs-server
```
接下来，在防火墙打开 SSH 和 NFS 端口，以确保可以从 NFS 客户端访问 NFS 服务。
```
firewall-cmd --permanent --zone=public --add-service=nfs
firewall-cmd --reload
```

#### Ubuntu/Debian
```bash
systemctl restart nfs-kernel-server
```

## 3. 配置 NFS 客户端

#### CentOS/RHEL/Fedora
```bash
yum install nfs-utils
```

#### Ubuntu/Debian
```bash
apt-get install nfs-common
```

挂载远程 NFS 共享：
```
mkdir -p /mnt/nfs/home
mount 192.168.142.131:/home /mnt/nfs/home
```

现在在输出中应当可以看到挂载的 NFS 共享。
```
df -h
```