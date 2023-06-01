---
title: Windows 环境 iSCSI 挂载
description: Docs intro
layout: ~/layouts/DocLayout.astro
---

iSCSI 协议是一种将 SCSI 协议扩展到 TCP/IP 网络的技术，可以实现在 IP 网络上运行块存储设备。

## 0. 环境要求

在本文中，我们将搭建一个 iSCSI 共享存储环境：
- 在 ZettaStor DBS 中安装并配置 iSCSI Target 服务，创建虚拟磁盘并分配给目标
- 在 Windows Server 2016 上安装并配置 iSCSI Initiator，发现并连接到 iSCSI Target 服务，并将虚拟磁盘格式化为本地分区或文件系统

| IP地址        | 系统           | 角色  |
| ------------- |:-------------:| -----:|
| 192.168.142.128 | ZettaStor DBS | iSCSI Target 服务，下文简称“DBS” |
| 192.168.142.130 | Windows Server 2016 | iSCSI initiator，下文简称“客户机” |

>**注意**  
下列命令假设您已经具有足够的管理员权限。

## 1. 在客户机上获取 iSCSI 限定名
1. 在 “控制面板 > Windows 工具” 中启动 “iSCSI 发起程序” 

<img src="/vitualization/media/iscsiwin01_zh.png" />
<img src="/vitualization/media/iscsiwin01a_zh.png" />

2. 在 “iSCSI 发起程序 - 属性” 页面上，点击 “配置”，在 “发起程序名称” 中查看客户机 IQN。这个 IQN 将在 [步骤2.7 创建访问控制并授权](/manual#访问控制管理) 中被使用。

<img src="/vitualization/media/iscsiwin02_zh.png" />

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
1. 在 “控制面板 > Windows 工具” 中启动 “iSCSI 发起程序”
2. 在 “iSCSI 发起程序 - 目标” 页面上，输入 DBS 的 IP 地址，并点击“快速连接”。

<img src="/vitualization/media/iscsiwin04_zh.png" width="66%" />

3. 确认发起程序已连接到 iSCSI 目标磁盘。

<img src="/vitualization/media/iscsiwin04a_zh.png" width="66%" />

### 3.2 挂载 iSCSI 驱动器
我们已经建立了与 iSCSI 目标的连接，挂载的驱动器现在可以在客户机上用作常规文件系统。我们需要对其进行初始化。

1. 在 “控制面板 > Windows 工具” 中启动 “计算机管理”，打开 “计算机管理 - 磁盘管理” 页面

2. 我们可以看到存在一个未分配的分区

<img src="/vitualization/media/iscsiwin10_zh.png" />

3. 创建一个卷。为此，请右键单击该分区，然后单击“新建简单卷”，并在接下来所有操作中全部点击“下一步”

<img src="/vitualization/media/iscsiwin14_zh.png" />

4. 在接下来所有操作中全部点击“下一步”，直到出现 “完成新的简单卷向导” 对话框，检查所有设置并单击 “完成” 以创建新卷

<img src="/vitualization/media/iscsiwin18_zh.png" />

5. 完成创建后可以从 Windows 资源管理器中访问该磁盘