本文档主要是列出分布式存储相关产品运行所依赖的操作系统相关信息，并对操作系统的安装要求进行说明。
根据文档指定的步骤，在操作系统安装配置完毕之后，可以更改 IP 地址，支持 root 用户通过 ssh 方式进行登录。

### 使用范围

本文档主要涉及操作系统安装以及部分需要手动配置的部分。

- 部分型号RAID卡的配置指导
- 操作系统安装过程中语言、时区、磁盘分区等方面的选择。（对于某些操作系统可以支持PXE方式安装，PXE方式安装相关的配置也需要遵循此文档中约定的设置）
- 操作系统安装完毕之后，基础环境正常运行所需要的驱动安装方法（比如网卡驱动的安装）
- 计算节点软件的安装（计算节点操作系统的安装不在此文档范围内，对于VMWare、OpenStack等涉及面广的软件由其余专门的文档进行说明）

### 更新记录 

2022-09-19
根据目前开源产品所支持的操作系统类型，对文档进行梳理

2022-09-30
评审后发布

## 操作系统安装

### 通用原则

无论哪种类型的操作系统，在进行安装时都需要遵循以下规则（特殊的操作系统为另外注明）：

- 在设置安装语言的时候，请选择 `English(United States)`
- 在设置日期和时间的时候，请选择 `Asia / Shanghai`
- 在设置安装磁盘和分区情况的时候，请选择需要安装操作系统的磁盘，并在设置分区的时候，除 `/boot` 分区和 `/` 根目录分区之外，对于 `swap` 和 `/home` 等分区不建议保留。也就是说，除了必须的 `/boot` 分区之外，系统磁盘的所有剩余空间分配给根分区，根分区文件系统格式推荐设置为 `ext4`

- 根据网络规划，为安装的节点设置对应的 IP 地址
- 其它选项请根据操作系统类型进行设置
- 某些类型的操作系统需要手动设置 hostname，推荐 hostname 的设置由小写字母、数字组成。
  原因在于我们通常以节点的 hostname 作为复制槽名称的组成部分，postgresql
  cluster 的复制槽的名称只能是小写字母、数字和下划线 组成。

本章节所列出的操作系统对分布式存储以及相关产品可以支持的大部分操作系统，至于每个项目实际选用哪种类型操作系统，需要综合多方面因素进行决定。

注意：

- 本章节列举的操作系统类型，名称按照字母从小到大进行排序，忽略大小写。
- 每一种操作系统的名称需要与`安装部署工具`中规划的名称保证一致

### CentOS 7 (1908)

#### 镜像信息

文件名称：`CentOS-7-x86_64-DVD-1908.iso`

MD5值：`dc5932260da8f26bcdce0c7ebf0f59ca`

#### 安装事项

使用U盘或者虚拟光驱等方式，进入系统安装界面。

首先选择语言为英语

<img src="https://zdbs.io/operatingsystem/media/image2.png" />

将时区设置为东八区。

<img src="https://zdbs.io/operatingsystem/media/image3.png" />

进行本次系统安装的软件选择

<img src="https://zdbs.io/operatingsystem/media/image4.png" />

这里软件列表选择`Compute Node`模式。 右边的软件包不需要选择。

<img src="https://zdbs.io/operatingsystem/media/image5.png" />

确定软件选择之后，开始进行系统分区配置

<img src="https://zdbs.io/operatingsystem/media/image6.png" />

选中系统安装所在的磁盘，然后选择自主配置分区选项。

<img src="https://zdbs.io/operatingsystem/media/image7.png" />

将分区模式由默认的方式，更改为 `Standard Partition`

<img src="https://zdbs.io/operatingsystem/media/image8.png" />

选定标准分区模式之后，点击上方的自动分区，进行分区调整。

<img src="https://zdbs.io/operatingsystem/media/image9.png" />

调整后，只保留 `/boot` 和 `/` 分区。系统磁盘的所有剩余空间分配给根分区，根分区文件系统格式推荐设置为 `ext4` 格式
<img src="https://zdbs.io/operatingsystem/media/image10.png" />

（这里有个方法来设置根分区大小，将不需要的分区删除之后，选择根分区，将根分区的大小，设置为一个大于整个系统盘的容量，然后调整根分区的文件系统格式为 ext4。系统会自动将尽可能所有的剩余空间分配给根分区）

确定分区配置，写入硬盘

<img src="https://zdbs.io/operatingsystem/media/image11.png" />

可以根据需要是否在此处配置节点的 IP 地址。然后可以开始安装操作。

<img src="https://zdbs.io/operatingsystem/media/image12.png" />

在安装过程中，配置 root 用户的密码。不需要创建其它的用户。

<img src="https://zdbs.io/operatingsystem/media/image13.png" />

等系统安装完毕之后，进行重启，检查是否可以正确进入系统即可。

#### 版本及内核信息

>**注意**  
在部署 ZettaStor DBS 之前，**不要**联网对原始操作系统的软件包进行任何升级。

系统安装完毕之后：

版本信息
```bash
cat /etc/*release | grep PRETTY_NAME

PRETTY_NAME="CentOS Linux 7 (Core)"
```

内核信息
```
uname -a

Linux localhost.localdomain 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7
18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

## 常见问题

### 如何从U盘引导系统安装

这里以 CentOS 7 为例来进行介绍。

首先使用 ISO 镜像制作 USB 启动盘（制作方法比较简单，可以自行上网搜索相关资料）。然后将服务器的启动顺序改为首先从 USB 启动。服务器启动后进行如下界面：

<img src="https://zdbs.io/operatingsystem/media/image33.png" />

在进入安装界面时，按 `Tab` 键，将命令修改为如下内容

`vmlinuz initrd=initrd.img linux dd quiet`

这一步的命令是为了确认 USB 启动盘的盘符。比如获取到 USB 的盘符为 `sdc4`
。那么再次通过U盘启动服务器，将 `Tab` 键出来的命令更改为如下内容

`vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdc4 quiet`

如果没有错误的话，就应该进入语言选择界面。开始进行操作系统安装操作。

### 安装引导出现“NMI watchdog: soft lockup CPU stuck”

这个问题发生在使用U盘等介质进行操作系统安装的时候，找不到U盘导致安装程序不能正常引导。

针对这种弄情况，可以尝试的操作就是：在引导之前通过设置 `nomodeset` 选项来禁用英特尔显卡功能

具体操作如下（以 CentOS 系统为例）：

正常插入U盘，从U盘启动，在启动项加载的时候，按 `Tab` 键（其它系统可能需要按 `e` 键）来修改启动项。在 `quiet` 
行添加 `nomodeset` 字段。

比如针对确定U盘盘符的操作，可以修改为：

`vmlinuz initrd=initrd.img linux dd nomodest quiet`

以及**确定盘符之后**

`vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdc4 nomodest quiet`

其余操作正常进行即可。
