在分离部署方案中，使用多台物理机设备仅部署 Zettastor DBS，分布式存储对接的应用软件由其它的节点进行部署。这样就做到存储和计算之间的分离。这种方式下计算和存储之间相对独立，各自软件可以单独部署、扩容，技术上也相对简单。Zettastor DBS通过标准的 iSCSI 协议与各类软件进行对接。

<img src="https://zdbs.io/vitualization/media/hci2.png" width="65%" />

在本文中，我们将介绍在 ZettaStor DBS 中设置 iSCSI 服务并在客户机中使用其中的存储空间来部署 VMware ESXi 虚拟机映像。

## 0. 环境要求

### 硬件要求
- 64位 x86 CPU
- 支持硬件虚拟机化 (Intel VT-x 或 AMD-V)
- 至少 8 GB 内存

### 软件要求
- 1 套 ZettaStor DBS 分布式存储设备（下文简称“DBS”）  
为了方便说明，假设 IP 为 192.168.142.128
- 1 台具有局域网连接的 VMware ESXi 客户机（下文简称“客户机”）

>**注意**  
下列命令假设您已经具有管理权限。

## 1. 在客户机上安装 VMware ESXi
### 1.1 安装 VMware ESXi

1. VMware vSphere Hypervisor (ESXi) 是一款商业产品，但是当您 [创建一个账户](https://customerconnect.vmware.com/account-registration) 并开始试用期后，您可以下载一个为期60天的 [试用版本](https://customerconnect.vmware.com/en/evalcenter?p=free-esxi7)。在网站页面点击“License & Download”，然后下载“VMware vSphere Hypervisor (ESXi ISO) image”。

<img src="https://zdbs.io/vitualization/media/esxi_install01.png" />

2. 刻录 ISO 光盘并从光盘启动客户机

<img src="https://zdbs.io/vitualization/media/esxi_install02.png" />

3. 在欢迎页面, 按 `ENTER` 键

<img src="https://zdbs.io/vitualization/media/esxi_install03.png" />

4. 按 `F11` 键接受许可

<img src="https://zdbs.io/vitualization/media/esxi_install04.png" />

5. 按 `ENTER` 键选择单个磁盘作为默认安装驱动器

6. 按 `ENTER` 键选择默认 `US keyboard`

7. 输入root的初始密码

8. 按 `F11` 键确认进行安装，并等待安装完成

9. 在重启客户机前，弹出 CDROM

<img src="https://zdbs.io/vitualization/media/esxi_install09.png" />

10. 重启客户机后，您应该看到下面的屏幕，显示 ESXi 服务器的管理地址

<img src="https://zdbs.io/vitualization/media/esxi_install10.png" />

### 1.2 启用 iSCSI 适配器

1. 打开浏览器输入 ESXi 服务器的管理地址，在登录界面输入root的密码

<img src="https://zdbs.io/vitualization/media/esxi_login.png" />

2. 在右上角点击用户名，在弹出菜单中选择 `Settings - Language` 切换语言

<img src="https://zdbs.io/vitualization/media/esxi_language.png" />

3. 在左侧导航栏点击存储图标，选择 `适配器`，点击 `软件iSCSI`

<img src="https://zdbs.io/vitualization/media/esxi_addiscsi.png" />

4. 点击 `已启用`，记录 `名称和别名`，点击 `保存配置`

<img src="https://zdbs.io/vitualization/media/esxi_iscsiinit1.png" />

5. 点击 `刷新`，确认 `iSCSI Software Adaptor` 状态为 `联机`

<img src="https://zdbs.io/vitualization/media/esxi_iscsiinit2.png" />

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
此步骤需要填写 [1.2 启用 iSCSI 适配器 ](#12-启用-iscsi-适配器) 第 4 步 中客户机的名称和别名(IQN)


## 3. 在 VMware ESXi 上创建虚拟机 

### 3.1 挂载 iSCSI 存储

1. 打开浏览器输入 ESXi 服务器的管理地址，在登录界面输入root的密码

2. 在左侧导航栏点击存储图标，选择 `适配器`，选择 `iSCSI Software Adaptor`，点击 `配置 iSCSI`

<img src="https://zdbs.io/vitualization/media/esxi_iscsiconf1.png" />

3. 点击 `添加动态目标`，填写 DBS IP 地址，然后点击 `保存配置`

<img src="https://zdbs.io/vitualization/media/esxi_iscsiconf2.png" />

4. 选择 `设备`，点击 `重新扫描`，然后点击相应的 iSCSI 磁盘

<img src="https://zdbs.io/vitualization/media/esxi_datastore0.png" />

5. 点击 `新建数据存储`

<img src="https://zdbs.io/vitualization/media/esxi_datastore1.png" />

6. 填写 `名称`，点击 `下一页`

<img src="https://zdbs.io/vitualization/media/esxi_datastore3.png" />

7. 选择 `使用全部磁盘`，点击 `下一页`

<img src="https://zdbs.io/vitualization/media/esxi_datastore4.png" />

8. 点击 `完成`

<img src="https://zdbs.io/vitualization/media/esxi_datastore5.png" />

9. 在警告对话框点击 `是`

<img src="https://zdbs.io/vitualization/media/esxi_datastore6.png" />

### 3.2 创建虚拟机 

1. 在左侧导航栏点击存储图标，选择 `虚拟机`，点击 `创建/注册虚拟机`

<img src="https://zdbs.io/vitualization/media/esxi_vm01.png" />

2. 点击 `下一页`

<img src="https://zdbs.io/vitualization/media/esxi_vm02.png" />

3. 根据实际情况填写虚拟机选项，点击 `下一页`

<img src="https://zdbs.io/vitualization/media/esxi_vm03.png" />

4. 选择已新建的 iSCSI 数据存储，点击 `下一页`

<img src="https://zdbs.io/vitualization/media/esxi_vm04.png" />

5. 选择相应的虚拟机配置以及安装镜像，点击 `下一页`

<img src="https://zdbs.io/vitualization/media/esxi_vm05.png" />

6. 点击 `完成`

<img src="https://zdbs.io/vitualization/media/esxi_vm06.png" />

7. 选择新建的虚拟机，点击 `打开电源`

<img src="https://zdbs.io/vitualization/media/esxi_vm07.png" />

<img src="https://zdbs.io/vitualization/media/esxi_vm08.png" />