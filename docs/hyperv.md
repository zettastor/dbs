在分离部署方案中，使用多台物理机设备仅部署 Zettastor DBS，分布式存储对接的应用软件由其它的节点进行部署。这样就做到存储和计算之间的分离。这种方式下计算和存储之间相对独立，各自软件可以单独部署、扩容，技术上也相对简单。Zettastor DBS通过标准的 iSCSI 协议与各类软件进行对接。

<img src="https://zdbs.io/vitualization/media/hci2.png" width="50%" />

在本文中，我们将介绍在 ZettaStor DBS 中设置 iSCSI 服务并在客户机中使用其中的存储空间来部署 Hyper-V 虚拟机映像。

## 0. 环境要求

### 硬件要求
- 支持二级地址转换（SLAT）的64位 CPU
- 支持虚拟机监视器模式扩展（Intel CPU 上的VT-c）
- 至少 4 GB 内存

### 软件要求
- 1 套 ZettaStor DBS 分布式存储设备（下文简称“DBS”）  
为了方便说明，假设 IP 为 192.168.142.128
- 1 台具有局域网连接的 Windows 客户机（下文简称“客户机”）

>**注意**  
下列命令假设您已经具有管理权限。

## 1. 在客户机上挂载 iSCSI
在客户机上，完成 [Windows 环境 iSCSI 挂载](/docs/iscsiwin.md) 中描述的所有操作。

## 2. 在客户机上安装 Hyper-V
### 2.1 安装 Hyper-V 及相关软件包

1. 角色功能安装，打开“服务器管理器”，点击“添加角色和功能”，运行“添加角色和功能向导”，点击“下一步”

<img src="https://zdbs.io/vitualization/media/hyperv_inst01_zh.png" />

2. 安装类型选择“基于角色或基于功能的安装”，点击“下一步”

<img src="https://zdbs.io/vitualization/media/hyperv_inst02_zh.png" />

3. 服务器选择“从服务器池中选择服务器”，选中本地服务器的计算机名，点击“下一步”

<img src="https://zdbs.io/vitualization/media/hyperv_inst03_zh.png" />

4. 服务器角色选择“Hyper-V”，弹出“添加Hyper-V所需的功能”，点击“添加功能”

<img src="https://zdbs.io/vitualization/media/hyperv_inst04_zh.png" />

5. 选中“Hyper-V”，接下来步骤均点击“下一步”

<img src="https://zdbs.io/vitualization/media/hyperv_inst05_zh.png" />

<img src="https://zdbs.io/vitualization/media/hyperv_inst06_zh.png" />

<img src="https://zdbs.io/vitualization/media/hyperv_inst07_zh.png" />

<img src="https://zdbs.io/vitualization/media/hyperv_inst08_zh.png" />

<img src="https://zdbs.io/vitualization/media/hyperv_inst09_zh.png" />

<img src="https://zdbs.io/vitualization/media/hyperv_inst10_zh.png" />

6. 点击 “安装”，过程中需要重新启动服务器

<img src="https://zdbs.io/vitualization/media/hyperv_inst11_zh.png" />

7. 安装完成，点击“关闭”

<img src="https://zdbs.io/vitualization/media/hyperv_inst12_zh.png" />

### 2.2 创建虚拟机 
1. 打开 Hyper-V 管理器
2. 点击“新建”，然后点击“虚拟机”
3. 在“新建虚拟机向导”中，勾选“将虚拟机存储在其他位置”，并将位置更改为步骤1中通过 iSCSI 挂载的磁盘，点击“下一页”

<img src="https://zdbs.io/vitualization/media/hyperv_setup03_zh.png" width="65%" />

4. 选择“第一代”，点击“下一页”

<img src="https://zdbs.io/vitualization/media/hyperv_setup04_zh.png" width="65%" />

5. 分配虚拟机内存，然后点击“下一页”

<img src="https://zdbs.io/vitualization/media/hyperv_setup05_zh.png" width="65%" />

6. 选择要用于局域网的虚拟交换机，然后点击“下一页”

<img src="https://zdbs.io/vitualization/media/hyperv_setup06_zh.png" width="65%" />

7. 创建虚拟硬盘，并将位置更改为步骤1中通过 iSCSI 挂载的磁盘，点击“下一页”

<img src="https://zdbs.io/vitualization/media/hyperv_setup07_zh.png" width="65%" />

8. 选择虚拟机的安装介质，选择“映像文件(.iso)”

<img src="https://zdbs.io/vitualization/media/hyperv_setup08_zh.png" width="65%" />

9. 在摘要页面确认配置后，点击“完成”

10. 在 Hyper-V 管理器中选择新创建的虚拟机，在右侧菜单中点击“连接”，然后点击“启动”

<img src="https://zdbs.io/vitualization/media/hyperv_setup10_zh.png" width="65%" />
