## 一、文档介绍

### 文档简介

本文档为南京鹏云网络科技有限公司（后面简称本公司）分布式块存储产品的使用说明文档。分布式存储技术是目前广泛流行和应用的一种新兴的存储技术。本文档主要介绍本公司提供的分布式块存储产品的使用。

### 适用对象

本手册主要适用于如下工程师：

- 存储规划人员
- 现场技术支持与维护人员
- 负责集群配置和维护的管理员

### 更新记录

2022-09-14
根据最新的开源版本更新文档

2022-09-30
评审后发布

## 二、概述

ZettaStor分布式块设备存储系统（简称ZettaStor
DBS）是一款软件定义的分布式存储。它运用分布式技术把大量标准架构服务器的存储介质进行聚合，将这些存储资源整合成为既具备传统SAN/NAS的企业级功能和特性，又具有高弹性、高扩展性、高可靠性的存储系统，可称做Server
SAN。ZettaStor分布式块设备存储系统可以与OpenStack、VMware和FusionCompute等云计算平台进行无缝对接。

<img src="https://zdbs.io/manual/media/image2.png" />

ZettaStor DBS直接管理磁盘裸设备，无需由文件层转化，效率更高、性能更好。

ZettaStor DBS通过PYD和ISCSI为客户机提供块存储服务。

ZettaStor
DBS可以为客户机提供QoS保障,并通过数据负载均衡（Rebalance）、数据重构（Rebuild）等功能，为系统提供高可靠、高性能保证。

ZettaStor
DBS包括InfoCenter、DriverContainer、DataNode、DIH、Console、Coordinator、deployment_daemon等软件模块：

- deployment_daemon主要用于部署、启动其它软件模块；
- DIH监控管理本节点所有服务的服务状态；
- InfoCenter是系统的信息中心，管理系统的配置信息（元数据信息）、性能和告警数据。元数据信息包括系统中所有的Volume信息、DataNode信息、账户信息。
- DriverContainer为系统的网络驱动容器，管理系统的所有网络驱动。当DriverContainer从外部接收挂载Volume的请求时，从容器中选择一个驱动同Volume挂载起来供客户机使用。目前支持标准的ISCSI（互联网小型计算机系统接口）协议和自研PYD协议；
- DataNode为存储模块，每个存储节点上都部署DataNode服务。DataNode管理存储节点上的所有用于存储的未分区的磁盘，并接收网络驱动发过来的读写请求，进行硬盘读写操作。DataNode集群采用P2P协议，各个DataNode上保存各自的元数据信息，并向其它的DataNode模块通报，因此无需元数据中央节点；一个Volume由多个Segment（分片）组成，Segment分布在不同的DataNode上，DataNode负责控制信息在哪一个Segment。Volume信息包括Volume所有的Segment信息和Volume的挂载信息。其中Segment信息中包含Segment到DataNode的映射关系。DataNode信息包含DataNode的存储资源，具体为每个DataNode总共有多少空间、已使用多少空间、还有多少空间可以使用；
- Console是系统的操作管理主要入口，提供Web界面供用户管理整个系统，包括账户管理、保护域管理、存储池、卷管理、服务管理等操作。

## 三、业务开通流程

用户在系统安装部署完成后，遵循以下业务开通流程，使用ZettaStor分布式块设备存储系统，具体如下：

<img src="https://zdbs.io/manual/media/image3.png" />

1. 创建存储域，添加存储节点
2. 创建存储池，添加存储磁盘
3. 创建存储卷
4. 创建存储驱动
5. 创建访问控制并授权
6. 客户端挂载驱动
7. 使用存储卷

以上步骤，详见 [存储管理](#六存储管理)。


## 四、系统登录与退出

### 系统登录

<!-- -->

1. 访问ZettaStor DBS Web界面，访问地址为http://*\[Console_IP\]*:8080。

<img src="https://zdbs.io/manual/media/image4.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

*Console_IP*即系统安装时指定的Console.deploy.host.list地址中的任何一个节点IP地址。

2. 登录ZettaStor DBS系统，默认用户名/密码：admin/admin。

<img src="https://zdbs.io/manual/media/image6.png" />

<img src="https://zdbs.io/manual/media/image7.png" />

admin用户具有系统最高权限，请及时在账户管理界面修改密码，并谨慎使用。

### 系统退出

1. 在页面右上角点击当前登录用户名，选择“用户登出”，也可点击图标<img src="https://zdbs.io/manual/media/image8.png" />安全退出登录账号。

<img src="https://zdbs.io/manual/media/image9.png" />

## 五、系统主页

系统主页实时动态展现系统级容量参数和状态参数。系统主页显示包括系统容量、服务、存储池、磁盘、卷、客户机等概览信息。

<img src="https://zdbs.io/manual/media/image6.png" />

1. 通过下图直观展现系统容量的总容量与使用情况，红色为已使用容量，黄色为已分配但未使用容量，蓝色为未分配未使用容量。

<img src="https://zdbs.io/manual/media/image10.png" />

2. 当前系统的服务数量，<img src="https://zdbs.io/manual/media/image11.png" />为健康状态的服务数量，<img src="https://zdbs.io/manual/media/image12.png" />为错误状态的服务数量。

<img src="https://zdbs.io/manual/media/image13.png" />

3. 当前系统中存储池数量，<img src="https://zdbs.io/manual/media/image11.png" />为健康状态的存储池数量，<img src="https://zdbs.io/manual/media/image12.png" />为亚健康状态的存储池数量。

<img src="https://zdbs.io/manual/media/image14.png" />

4. 当前系统中磁盘数量，<img src="https://zdbs.io/manual/media/image11.png" />为健康状态的磁盘数量，<img src="https://zdbs.io/manual/media/image12.png" />为错误状态的磁盘数量。

<img src="https://zdbs.io/manual/media/image15.png" />

5. 当前系统中卷数量，<img src="https://zdbs.io/manual/media/image11.png" />为健康状态的卷数量，<img src="https://zdbs.io/manual/media/image12.png" />为错误状态的卷数量。

<img src="https://zdbs.io/manual/media/image16.png" />

6. 当前系统中驱动的客户机的总数量，<img src="https://zdbs.io/manual/media/image17.png" />为客户机的连接数。

<img src="https://zdbs.io/manual/media/image18.png" />

7. 当前登陆的用户：admin，此处可以修改当前用户密码，也可以登出系统。

<img src="https://zdbs.io/manual/media/image19.png" /><img src="https://zdbs.io/manual/media/image9.png" />

## 六、存储管理

ZettaStor
DBS以存储块为设备的形式向外提供存储服务。用户通过Console创建好Volume（卷）以后，在界面上将Volume通过挂载驱动提供访问接口，系统提供两种网络驱动：ISCSI和PYD。用户可以根据系统返回的挂载信息，在某个客户机主机通过网络驱动连上挂载的网络驱动，之后便可以在客户机主机对挂载的Volume进行读写操作了。

日常使用ZettaStor
DBS，通常的操作顺序是新建域--\>管理域-增减DataNode节点--\>新建存储池--\>管理存储池--\>存储池扩容、减容--\>新建卷--\>挂载驱动--\>读写卷--\>卸载驱动、扩展卷等诸多功能。

### 域管理

域管理将存储系统以物理节点（部署DataNode服务的节点）为粒度分为不同的范围，此范围可以根据节点的硬件配置进行划分，也可以按不同的地域，不同的管理权限进行划分。每个DataNode节点最多只能属于一个域。创建域后，在创建Volume时选择创建的域，该卷的Segment
Unit将均衡地分配到每个域包含的DataNode节点之中，不会扩散到当前域之外的DataNode节点。

域管理支持域的创建、删除、修改、域信息查看、添加或者删除存储节点等操作。

#### 创建域

1. 在左侧导航栏选择“存储”--\>“域&存储池”。

<img src="https://zdbs.io/manual/media/image20.png" />

2. 选择“创建”，输入“域名称”和“描述”，选择“创建”，选择“完成”。

<img src="https://zdbs.io/manual/media/image21.png" /><img src="https://zdbs.io/manual/media/image22.png" />

1. “域名称”只能输入中英文、数字、“\_”，长度2-64，“域名称”不能重复。

2. “描述”为可选项，输入长度最多为250位字符。

<!-- -->

3. 选择域名称，选择“操作”--\>“存储节点”，开始添加节点操作，也可以根据上一步提示页直接选择“添加节点”。

<img src="https://zdbs.io/manual/media/image23.png" />

4. 勾选相应DataNode节点，选择“添加节点”，选择“完成”。

<img src="https://zdbs.io/manual/media/image24.png" />

5. 查看域信息。

<img src="https://zdbs.io/manual/media/image25.png" />

域信息包含：

1. 状态分为可用和不可使用。

2. 总容量代表该域中包含的所有DataNode节点的总容量。

3. 剩余容量代表该域中包含的所有DataNode节点的剩余容量。

4. 已分配容量代表该域中包含的所有DataNode节点的已分配容量。

#### 修改域

1. 在“域&存储池”导航栏下，选择域名称，选择“操作”，选择“修改”，开始修改域操作。

<img src="https://zdbs.io/manual/media/image26.png" />

<img src="https://zdbs.io/manual/media/image27.png" />

>**说明**  
域ID无法修改。

#### 添加和移除存储节点

1. 在“域&存储池”导航栏下，选择域名称，选择“操作”，选择“存储节点”，开始添加和移除存储节点操作。

<img src="https://zdbs.io/manual/media/image26.png" />

- **添加存储节点**

1. 选择相应DataNode节点，选择“添加节点”，选择“完成”。

<img src="https://zdbs.io/manual/media/image28.png" />

- **移除存储节点**

1. 选择相应DataNode节点，选择“移除存储节点”，选择“确认”。

<img src="https://zdbs.io/manual/media/image29.png" />

>**说明**  
添加存储节点：此节点纳入存储系统，为存储节点提供存储服务，往域中添加节点后，域的容量发生相应改变。  
移除存储节点：此节点不再为当前域提供存储服务，节点移除后，其上的数据会在当前域的其它节点发起重构，重构与否要根据域的实际情况综合判断，域的容量发生相应改变。

#### 域详情

1. 在“域&存储池”导航栏下，选择域名称。

<img src="https://zdbs.io/manual/media/image30.png" />

<img src="https://zdbs.io/manual/media/image31.png" />

域详情包括已使用节点信息和基本信息，基本信息包含域名称、描述、状态、总容量、剩余容量、已分配容量。

#### 查询域

1. 在“域&存储池”导航栏下，在搜索栏输入关键字查询。

<img src="https://zdbs.io/manual/media/image32.png" />

>**说明**  
查询域支持模糊查询。

#### 删除域

1. 在“域&存储池”导航栏下，勾选域名称。

<img src="https://zdbs.io/manual/media/image33.png" />

2. 选择“删除”，进行删除操作。

<img src="https://zdbs.io/manual/media/image34.png" />

>**注意**  
域中存在存储池时，必须先删除存储池才能删除域，数据无价，请谨慎使用。

### 存储池管理

ZettaStor分布式块设备存储系统通过在存储节点上部署DataNode模块，把各个节点上的各类存储介质进行聚合，形成可统一管理的存储池，直接对外提供高性能的块设备服务。存储池是基于磁盘粒度，对存储空间的再一次划分。每一个数据盘最多只能属于一个池，缓存盘不属于任何存储池。

存储池管理支持存储池的创建、删除、修改、存储池中磁盘信息查看、添加或者移除磁盘等操作。

#### 创建存储池

1. 在“域&存储池”导航栏下，选择域名称，选择“存储池”，开始创建存储池。

<img src="https://zdbs.io/manual/media/image35.png" />

2. 选择“创建”，输入“存储池名称”和“描述”，选择“创建”，选择“完成”。

<img src="https://zdbs.io/manual/media/image36.png" />

<img src="https://zdbs.io/manual/media/image37.png" /><img src="https://zdbs.io/manual/media/image38.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “存储池名称”只能输入中英文、数字、“\_”，长度2-64，“存储池名称”不能重复。

2. “描述”为可选项，输入长度最多为250位字符。

<!-- -->

3. 选择存储池名称，选择“操作”--\>“磁盘”，进行添加磁盘操作，也可以根据上一步提示页直接选择“添加”。

<img src="https://zdbs.io/manual/media/image39.png" />

4. 选择相应节点磁盘，选择“扩容”--\>“完成”。

<img src="https://zdbs.io/manual/media/image40.png" />

5. 查看存储池信息。

<img src="https://zdbs.io/manual/media/image41.png" />

存储池信息包含存储池名称、描述、状态、存储池等级、QoS策略、重构进度等内容。

#### 修改存储池

1. 在“域&存储池”导航栏下，选择域名称，选择“操作”，选择“存储池”，选择存储池，选择“操作”，选择“修改”。

<img src="https://zdbs.io/manual/media/image42.png" />

<img src="https://zdbs.io/manual/media/image43.png" />

>**说明**  
存储池ID无法修改。

#### 存储池磁盘扩容和减容

1. 在“域&存储池”导航栏下，选择域名称，选择“操作”，选择“存储池”，选择存储池，选择“操作”，选择“磁盘”。

<img src="https://zdbs.io/manual/media/image42.png" />

- **存储池磁盘扩容**

1. 选择相应节点磁盘，选择“扩容”--\>“完成”。

<img src="https://zdbs.io/manual/media/image40.png" />

- **存储池磁盘减容**

1. 选择相应节点磁盘，选择“减容”，选择“确认”。

<img src="https://zdbs.io/manual/media/image44.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. 存储池磁盘扩容：将磁盘添加至存储池，为存储池提供存储服务，往存储池中添加磁盘后，存储池的容量发生相应改变。

2. 存储池磁盘缩容：磁盘不再用于该存储池的存储服务，磁盘移除后，其上的数据会在其它磁盘重构，重构与否要根据存储池的实际情况综合判断，存储池的容量发生相应改变。

#### 存储池详情

1. 在“域&存储池”导航栏下，选择域名称，选择“操作”，选择“存储池”，选择存储池名称。

<img src="https://zdbs.io/manual/media/image42.png" />

2. 基本信息包含存储池名称、描述、总容量、类型、状态、QoS策略、存储池等级、重构进度。

<img src="https://zdbs.io/manual/media/image45.png" />

3. 性能数据页面包含存储池使用状况、两副本与三副本剩余有效容量。

<img src="https://zdbs.io/manual/media/image46.png" />

4. 磁盘信息页面包含各个磁盘的实例ID、主机IP地址、所在组编号、磁盘名、存储类型等。

<img src="https://zdbs.io/manual/media/image47.png" />

#### 查询储存池

1. 在“域&存储池”导航栏下，选择域名称，选择“操作”，选择“存储池”，在搜索栏输入关键字查询。

<img src="https://zdbs.io/manual/media/image48.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

查询存储池支持模糊查询。

#### 删除存储池

1. 在“域&存储池”导航栏下，选择域名称，选择“操作”--\>“存储池”，勾选存储池名称。

<img src="https://zdbs.io/manual/media/image48.png" />

2. 选择“删除”，选择“删除”。

<img src="https://zdbs.io/manual/media/image49.png" />

>**注意**  
存储池中存在卷时，必须先删除卷才能删除存储池，数据无价，请谨慎使用。

### 卷管理

创建域和存储池后，才能进行卷的创建操作。卷管理是客户应用的重要组成部分，提供对逻辑卷的全面操作和状态呈现。功能包括创建卷、查询卷、卷详情、删除卷、扩展卷。

#### 卷界面

1. 在左侧导航栏选择“存储”--\>“卷”，包含“卷列表”和“卷回收站”子页面。

<img src="https://zdbs.io/manual/media/image50.png" />

2. 卷列表包含卷名称、描述、状态、所属域、所属存储池、创建时间、重构、Rebalance进度、创建类型、操作等。

<img src="https://zdbs.io/manual/media/image51.png" />

3. 卷回收站包含卷名称、描述、状态、所属域、所属存储池、放入时长、重构、Rebalance进度、创建类型、操作等。

<img src="https://zdbs.io/manual/media/image52.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “状态”分为创建中、可用、稳定、不可用、删除中、已删除、克隆中、迁移中等。

2. “所属域”指此卷分配、迁移Segment Unit的DataNode范围。

3. “所属存储池”指此卷分配、迁移Segment Unit的存储磁盘范围。

4. “重构进度”指卷重构时的进度百分比和重构速度。

#### 创建卷

1. 在左侧导航栏选择“存储”--\>“卷”。

<img src="https://zdbs.io/manual/media/image53.png" />

2. 选择“创建”，输入“卷名”、“描述”、“卷容量”、“副本数量”、“所在域”、“存储池”，选择“创建”，选择“完成”。正常情况下卷的最终状态为稳定。

<img src="https://zdbs.io/manual/media/image54.png" /><img src="https://zdbs.io/manual/media/image55.png" />

<img src="https://zdbs.io/manual/media/image56.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “卷名”只能输入中英文、数字、“\_”，长度2-64，“卷名”不能重复。

2. “卷容量”只能输入正整数，最终创建出来的卷大小为Segment
    Unit大小的整数倍。

3. “副本数量”分为2副本、3副本和3副本（高可靠），其中3副本（高可靠）至少需要5个节点。

4. “所在域”指卷存放的物理节点范围。

5. “存储池”指卷存放的物理磁盘范围。

#### 查询卷

1. 在左侧导航栏选择“存储”--\>“卷”，在搜索栏输入关键字查询。

<img src="https://zdbs.io/manual/media/image57.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

查询卷支持模糊查询。

#### 卷详情

1. 在左侧导航栏选择“存储”--\>“卷”，选择卷名称。

<img src="https://zdbs.io/manual/media/image57.png" />

2. 基本信息包含卷名称、描述、占用物理空间、总容量、剩余容量、已用容量、卷状态、所属域、所属存储池、副本数量、创建时间、重构进度、Rebalance进度、创建类型。

<img src="https://zdbs.io/manual/media/image58.png" />

3. 映射关系页面包含映射驱动列表，驱动名称、快照ID、驱动状态、驱动容器IP、驱动地址、驱动端口、驱动用户数量、chap认证、驱动用户信息。

<img src="https://zdbs.io/manual/media/image59.png" />

#### 删除卷

1. 在左侧导航栏选择“存储”--\>“卷”，勾选卷名称。

<img src="https://zdbs.io/manual/media/image60.png" />

2. 选择“删除”，选择“删除”。

<img src="https://zdbs.io/manual/media/image61.png" />

<img src="https://zdbs.io/manual/media/image7.png" />

卷挂载驱动时，必须先删除驱动才能删除卷，数据无价，请谨慎使用。

#### 扩展卷

创建卷完成后，如果需要增加卷的大小，可以通过扩展卷界面进行扩展，支持卷的批量扩展。

1. 在左侧导航栏选择“存储”--\>“卷”，勾选卷名称。

<img src="https://zdbs.io/manual/media/image62.png" />

2. 选择“扩展”，输入“扩展大小”，选择“扩展”。

<img src="https://zdbs.io/manual/media/image63.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

扩展大小表示源卷增加的空间，即扩展后卷大小 = 扩展前卷大小 + 扩展大小。

### 驱动管理

ZettaStor分布式块设备存储系统支持ISCSI和PYD两种类型驱动。

ISCSI：现有SCSI接口与以太网络（Ethernet）技术结合，使服务器可与使用IP网络的储存装置互相交换资料。

PYD：将远程主机的磁盘空间，当作一个块设备来使用，就像一块硬盘一样使用，可以很方便地将另一台服务器的硬盘空间增加到本地服务器上。

驱动状态包含挂载中、已挂载、卸载中、未知和错误等类型。

#### 挂载驱动

1. 在左侧导航栏选择“存储”--\>“驱动”。

<img src="https://zdbs.io/manual/media/image64.png" />

2. 选择“挂载”，输入“驱动名称”、“目标卷”、“驱动类型”、“驱动数量”，选择“高级筛选”，需要选择“所在域”、“存储池”来筛选“目标卷”，选择“挂载”。

<img src="https://zdbs.io/manual/media/image65.png" /><img src="https://zdbs.io/manual/media/image66.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “驱动名称”只能输入中英文、数字、“\_”，长度2-64。

2. “驱动类型”分为ISCSI和PYD两种类型。

3. “驱动数量”只能输入正整数，不能大于DriverContainer节点数量，如果“驱动数量”为1则为单链路，可以手动或者由系统自动指定“驱动容器”。

#### PYD单链路

1. 挂载PYD单链路驱动。

<img src="https://zdbs.io/manual/media/image67.png" />

<img src="https://zdbs.io/manual/media/image68.png" />

2. 客户机创建目录/opt/pyd/（默认位置），将pyd-client和pyd.ko文件复制此目录下，pyd.ko文件与操作系统内核版本相关，使用前需要确认。

<img src="https://zdbs.io/manual/media/image69.png" />

3. 加载并检查内核。

insmod pyd.ko

lsmod \| grep pyd

ll /dev/pyd\*

<img src="https://zdbs.io/manual/media/image70.png" />

4. 将PYD设备与驱动地址连接。

命令格式为：/opt/pyd/pyd-client 卷ID 快照ID 驱动地址 /dev/pydx，例如：

/opt/pyd/pyd-client 3223620355005862222 0 10.0.3.174 /dev/pyd0

<img src="https://zdbs.io/manual/media/image71.png" />

<img src="https://zdbs.io/manual/media/image72.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “卷ID”、“驱动地址”可在“存储”--\>“驱动”界面查看。“快照ID”默认为0

2. PYD与驱动地址连接前，需要配置客户机pyd驱动授权对应的卷，详见“访问控制”章节。

<!-- -->

5. 将本地挂载设备进行分区、格式化、创建文件系统和读写等操作，略。

6. 断开PYD设备与驱动地址连接。

命令格式为/opt/pyd/pyd-client -f /dev/pydx，例如：

/opt/pyd/pyd-client -f /dev/pyd0

<img src="https://zdbs.io/manual/media/image73.png" />

<img src="https://zdbs.io/manual/media/image74.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

断开PYD设备与驱动地址连接前，需要卸载文件系统。

#### PYD多链路

1. 挂载PYD多链路驱动。

<img src="https://zdbs.io/manual/media/image75.png" />

<img src="https://zdbs.io/manual/media/image76.png" />

2. 客户机创建目录/opt/pyd/，将pyd-client和pyd.ko文件放入此目录下。

<img src="https://zdbs.io/manual/media/image69.png" />

3. 加载并检查内核。

insmod pyd.ko

lsmod \| grep pyd

ll /dev/pyd\*

<img src="https://zdbs.io/manual/media/image70.png" />

4. 客户机创建目录/opt/pyd/config/，新增配置文件multipaths，内容如下：

\[pyd0\]

10.0.3.174

10.0.3.176

<img src="https://zdbs.io/manual/media/image5.png" />

multipaths文件内的地址与“存储”--\>“驱动”界面“驱动地址”一致。

5. 将PYD设备与驱动地址连接。

命令格式为：/opt/pyd/pyd-client 卷ID 快照ID /dev/pydx-M，例如：

/opt/pyd/pyd-client 3223620355005862222 0 /dev/pyd0 -M

<img src="https://zdbs.io/manual/media/image77.png" />

<img src="https://zdbs.io/manual/media/image78.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “卷ID”可在“存储”--\>“驱动”界面查看，“快照ID”默认为0。

2. PYD与驱动地址连接前，需要配置客户机pyd驱动授权对应的卷，详见“访问控制”章节。

<!-- -->

6. 将本地挂载设备进行分区、格式化、创建文件系统和读写等操作，略。

7. 断开PYD设备与驱动地址连接。

命令格式为：/opt/pyd/pyd-client -f /dev/pydx，例如：

/opt/pyd/pyd-client -f /dev/pyd0

<img src="https://zdbs.io/manual/media/image79.png" />

<img src="https://zdbs.io/manual/media/image80.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

断开PYD设备与驱动地址连接前，需要卸载文件系统。

#### ISCSI单链路

1. 挂载ISCSI单链路驱动。

<img src="https://zdbs.io/manual/media/image81.png" />

<img src="https://zdbs.io/manual/media/image82.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

“chap认证”为可选配置，客户机相关配置需与之同步。

2. 客户机安装iscsi-initiator-utils，以CentOS为例，命令如下：

yum -y install iscsi-initiator-utils

3. 客户机查看initiatorname，与“访问控制”相关，以CentOS为例，命令如下：

cat /etc/iscsi/initiatorname.iscsi

<img src="https://zdbs.io/manual/media/image83.png" />

4. 客户机配置chap认证，可选，以CentOS为例，配置如下：

cat /etc/iscsi/iscsid.conf

node.session.auth.authmethod = CHAP

node.session.auth.username = inadmin \#incoming用户

node.session.auth.password = inadmin \#incoming密码

node.session.auth.username_in = outadmin \#outgoing用户

node.session.auth.password_in = outadmin \#outgoing密码

<img src="https://zdbs.io/manual/media/image5.png" />

chap认证为可选配置，驱动相关配置需与之同步。

5. 客户机启动iscsid，以CentOS为例，命令如下：

systemctl start iscsid

systemctl enable iscsid

6. 客户机发现target。

以CentOS为例，命令格式为iscsiadm -m discovery -t st -p IP:PORT，例如：

iscsiadm -m discovery -t st -p 10.0.3.176

<img src="https://zdbs.io/manual/media/image84.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. *IP:PORT*可在“存储”--\>“驱动”界面查看，PORT默认为3260。

2. 客户机发现target前，需要配置客户机ISCSI驱动授权对应的卷，详见“访问控制”章节。

<!-- -->

7. 客户机登录节点，以CentOS为例，命令如下：

iscsiadm -m node --login

<img src="https://zdbs.io/manual/media/image85.png" />

<img src="https://zdbs.io/manual/media/image86.png" />

8. 客户机查看挂载设备，以CentOS为例，命令如下：

ls -l /dev/disk/by-path/ip-\*

<img src="https://zdbs.io/manual/media/image87.png" />

9. 将本地挂载设备进行分区、格式化、创建文件系统和读写等操作，略。

10. 断开ISCSI设备与驱动地址连接。

以CentOS为例，命令格式为iscsiadm -m node -T IQN_Number -p IP
--logout，例如：

iscsiadm -m node --logout

<img src="https://zdbs.io/manual/media/image88.png" />

<img src="https://zdbs.io/manual/media/image89.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. 断开ISCSI设备与驱动地址连接前，需要卸载文件系统。

2. *IQN_Number*和*IP*如未指定，指所有ISCSI会话。

#### ISCSI多链路

1. 挂载ISCSI多链路驱动，即驱动数量设置为大于1的值，通常为2或者3。

<img src="https://zdbs.io/manual/media/image90.png" />

<img src="https://zdbs.io/manual/media/image91.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

“chap认证”为可选配置，客户机相关配置需与之同步。

2. 客户机安装iscsi-initiator-utils，以CentOS为例，命令如下：

yum -y install iscsi-initiator-utils

3. 客户机查看initiatorname，与“访问控制”相关，以CentOS为例，命令如下：

cat /etc/iscsi/initiatorname.iscsi

<img src="https://zdbs.io/manual/media/image92.png" />

4. 客户机配置chap认证，可选，以CentOS为例，配置如下：

cat /etc/iscsi/iscsid.conf

node.session.auth.authmethod = CHAP

node.session.auth.username = inadmin \#incoming用户

node.session.auth.password = inadmin \#incoming密码

node.session.auth.username_in = outadmin \#outgoing用户

node.session.auth.password_in = outadmin \#outgoing密码

<img src="https://zdbs.io/manual/media/image5.png" />

chap认证为可选配置，驱动相关配置需与之同步。

5. 客户机启动iscsid，以CentOS为例，命令如下：

systemctl start iscsid（启动iscsid服务）

systemctl enable iscsid（配置iscsid服务为开机自启动）

6. 客户机发现target。

以CentOS为例，命令格式为iscsiadm -m discovery -t st -p IP:PORT，例如：

iscsiadm -m discovery -t st -p 10.0.3.174

iscsiadm -m discovery -t st -p 10.0.3.176

<img src="https://zdbs.io/manual/media/image93.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. *IP:PORT*可在“存储”--\>“驱动”界面查看，PORT默认为3260。

2. 客户机发现target前，需要配置客户机ISCSI驱动授权对应的卷，详见“访问控制”章节。

<!-- -->

7. 客户机登录节点，以CentOS为例，命令如下：

iscsiadm -m node --login

<img src="https://zdbs.io/manual/media/image94.png" />

<img src="https://zdbs.io/manual/media/image95.png" />

8. 客户机查看挂载设备，以CentOS为例，命令如下：

ls -l /dev/disk/by-path/ip-\*

<img src="https://zdbs.io/manual/media/image96.png" />

9. 客户机安装多路径软件device-mapper-multipath，以CentOS为例，命令如下：

yum -y install device-mapper-multipath

10. 创建并修改多路径配置文件/etc/multipath.conf

cp /usr/share/doc/device-mapper-multipath-0.4.9/multipath.conf
/etc/multipath.conf

修改配置文件/etc/multipath.conf

defaults {

user_friendly_names yes

find_multipaths no

polling_interval 1

path_selector "queue-length 0"

path_grouping_policy multibus

uid_attribute ID_SERIAL

prio const

path_checker tur

rr_min_io_rq 1

max_fds 64

rr_weight priorities

failback immediate

no_path_retry fail

checker_timeout 2

}

blacklist_exceptions {

device {

vendor "LIO-ORG"

}

}

blacklist {

devnode "^(ram\|raw\|loop\|fd\|md\|dm-\|sr\|scd\|st)\[0-9\]\*"

devnode "^hd\[a-z\]"

device {

vendor ".\*"

product ".\*"

}

}

<img src="https://zdbs.io/manual/media/image5.png" />

1. failback设置为immediate，表示当前链路出现问题立刻切换到下一条链路，此项配置同时还需调整open-iscsi配置文件中的相关参数才能生效，具体设置/etc/iscsi/iscsid.conf中参数node.session.timeo.replacement_timeout
    = 0。

2. blacklist配置需添加本地磁盘。如果客户机同时也是部署DataNode服务的存储节点的话，本地磁盘一旦被管理，会导致DataNode无法识别磁盘。

<!-- -->

11. 客户机启动multipathd，以CentOS为例，命令如下：

systemctl start multipathd

systemctl enable multipathd

12. 客户机查看多路径拓扑信息，以CentOS为例，命令如下：

multipath -ll

<img src="https://zdbs.io/manual/media/image97.png" />

13. 将本地挂载设备（/dev/mapper/mpatha或者/dev/dm-0）进行分区、格式化、创建文件系统和读写等操作，略。

14. 断开ISCSI设备与驱动地址连接。

以CentOS为例，命令格式为iscsiadm -m node -T IQN_Number -p IP
--logout，例如：

iscsiadm -m node --logout

<img src="https://zdbs.io/manual/media/image98.png" />

<img src="https://zdbs.io/manual/media/image99.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. 断开ISCSI设备与驱动地址连接前，需要在客户机上卸载文件系统。

2. *IQN_Number*和*IP*如未指定，指所有ISCSI会话。

#### 卸载驱动

1. 在左侧导航栏选择“存储”--\>“驱动”。

<img src="https://zdbs.io/manual/media/image100.png" />

2. 选择驱动名称，选择“操作”，选择“卸载”。

<img src="https://zdbs.io/manual/media/image101.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

卸载驱动前，“驱动用户数量”需为0。

### 访问控制管理

ZettaStor系统支持ISCSI和PYD两种类型驱动，对应需要创建ISCSI和PYD两种访问控制。

#### ISCSI客户机

- **创建客户机**

1. 在左侧导航栏选择“存储”--\>“访问控制”。

<img src="https://zdbs.io/manual/media/image102.png" />

2. 选择“ISCSI驱动”，选择“创建”，输入“规则名称”、“Initiator名称”、“incoming用户”、“incoming密码”、“outcoming用户”、“outcoming密码”，选择“创建”。

<img src="https://zdbs.io/manual/media/image103.png" /><img src="https://zdbs.io/manual/media/image104.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “规则名称”只能输入中英文、数字、“\_”，长度2-64。

2. “Initiator名称”需从客户机获取，获取方式见下文。

3. “incoming用户”、“incoming密码”、“outcoming用户”、“outcoming密码”为可选项，客户端配置、驱动配置需与之同步，客户端配置见下文。

<!-- -->

3. 客户机查看Initiator名称，以CentOS为例，命令如下：

cat /etc/iscsi/initiatorname.iscsi

<img src="https://zdbs.io/manual/media/image105.png" />

4. 客户机配置chap认证，可选。以CentOS为例，如果不需要CHAP认证的话，将下列内容注释或者删除；如果需要CHAP认证的话，配置如下：

cat /etc/iscsi/iscsid.conf

node.session.auth.authmethod = CHAP

node.session.auth.username = inadmin \#incoming用户

node.session.auth.password = inadmin \#incoming密码

node.session.auth.username_in = outadmin \#outgoing用户

node.session.auth.password_in = outadmin \#outgoing密码

- **授权客户机**

1. 在左侧导航栏选择“存储”--\>“访问控制”，选择“ISCSI驱动”。

<img src="https://zdbs.io/manual/media/image106.png" />

2. 选择规则名称，选择“操作”，选择“授权”，选择驱动名称，选择“应用”或“撤销”，授权驱动或撤销授权，选择“完成”。

<img src="https://zdbs.io/manual/media/image107.png" />

<img src="https://zdbs.io/manual/media/image108.png" />

- **删除客户机**

1. 在左侧导航栏选择“存储”--\>“访问控制”，选择“ISCSI驱动”，勾选规则名称。

<img src="https://zdbs.io/manual/media/image109.png" />

2. 选择“删除”，选择“删除”，删除客户机。

<img src="https://zdbs.io/manual/media/image110.png" />

#### PYD客户机

- **创建客户机**

1. 在左侧导航栏选择“存储”--\>“访问控制”。

<img src="https://zdbs.io/manual/media/image111.png" />

2. 选择“PYD驱动”，选择“创建”,输入“客户机IP地址”。

<img src="https://zdbs.io/manual/media/image112.png" />

- **授权客户机**

1. 在左侧导航栏选择“存储”--\>“访问控制”，选择“PYD驱动”。

<img src="https://zdbs.io/manual/media/image113.png" />

2. 选择客户机IP，选择“操作”，选择“授权”，选择“卷名称”，选择“应用”或“撤销”，授权卷或撤销授权，选择“完成”。

<img src="https://zdbs.io/manual/media/image114.png" />

<img src="https://zdbs.io/manual/media/image115.png" />

- **删除客户机**

1. 在左侧导航栏选择“存储”--\>“访问控制”，选择“PYD驱动”，勾选客户机IP。

<img src="https://zdbs.io/manual/media/image116.png" />

2. 选择“删除”，选择“删除”，删除客户机。

<img src="https://zdbs.io/manual/media/image117.png" />

### QoS策略管理

#### 数据访问QoS策略

数据访问QoS策略可限制卷某个时间段的数据写入和读取操作的IOPS和吞吐量，包括新建QoS、修改QoS、删除QoS、应用QoS和撤销应用的操作，以保证关键应用的正常。

- **创建数据访问QoS策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“数据访问”。

<img src="https://zdbs.io/manual/media/image118.png" />

2. 选择“创建”，输入“策略名称”、“类型”、“IOPS限制”、“吞吐量”，“类型”为“动态”时，需添加规则，输入“时间跨度”，选择“添加”。

<img src="https://zdbs.io/manual/media/image119.png" /><img src="https://zdbs.io/manual/media/image120.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “策略名称”只能输入中英文、数字、“\_”，长度2-64，“策略名称”不能重复。

2. “类型”分为静态策略和动态策略，静态策略永久生效，动态策略运行指定时间区间生效。

3. 静态类型的限制项优先其他的限制项执行。

4. 在IOPS和吞吐量同时设置的情况下，以先满足限制条件的条目为准。

- **查看数据访问QoS策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“数据访问”，点击策略名称左侧“+”，查看QoS策略详细信息。

<img src="https://zdbs.io/manual/media/image121.png" />

- **修改数据访问QoS策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“数据访问”，选择策略名称，选择“操作”，选择“修改”，修改相关参数，选择“修改”。

<img src="https://zdbs.io/manual/media/image122.png" />

<img src="https://zdbs.io/manual/media/image123.png" /><img src="https://zdbs.io/manual/media/image124.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. 静态策略支持修改“IOPS限制”和“吞吐量”。

2. 动态策略支持修改“时间跨度”、“IOPS限制”和“吞吐量”。

3. 策略修改之后即时生效。

- **关联数据访问QoS策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“数据访问”，选择策略名称“操作”，选择“关联设置”。

<img src="https://zdbs.io/manual/media/image125.png" />

2. 选择“驱动名称”，选择“应用”或“撤销”，关联驱动或撤销关联，选择“完成”。

<img src="https://zdbs.io/manual/media/image126.png" />

- **删除数据访问QoS策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“数据访问”，勾选策略名称，选择“删除”，选择“删除”，删除QoS策略。

<img src="https://zdbs.io/manual/media/image127.png" />

<img src="https://zdbs.io/manual/media/image128.png" />

#### 数据重构QoS策略

当存储单元（节点或者磁盘）发生故障或者被删除后，被删除存储单元上的数据会在其他健康存储单元上进行自动重构（Rebuild），遵循负载轻优先的原则选择重构的存储单元，确保数据的负载均衡。根据重构策略或者重构数据量的不同，重构可能会对逻辑卷的数据写入或者读取速率产生影响。重构期间，系统会根据一定的算法，自动寻找可用的、空闲的磁盘空间，进行数据的重新构建。

数据重构的速率除了QoS策略的影响，还受配置文件中最大允许的重构卷的数据和Segment
Unit数量的限制。

- **查看数据重构QoS策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“数据重构”，查看QoS策略详细信息。

<img src="https://zdbs.io/manual/media/image129.png" />

- **关联数据重构QoS策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“数据重构”，选择策略名称“操作”，选择“关联设置”。

<img src="https://zdbs.io/manual/media/image130.png" />

2. 选择“存储池名”，选择“应用”或“撤销”，关联存储池或撤销关联，选择“完成”。

<img src="https://zdbs.io/manual/media/image131.png" />

#### 负载均衡策略

当新的存储单元（磁盘或者节点）加入后，系统会自动识别和接纳。在系统负载均衡开关开启或者满足负载均衡策略限制的情况下，自动将系统中的数据向新的存储单元进行数据迁移，确保存储单元（磁盘或者节点）数据负载达到一个平衡状态。负载均衡完成后，存储池内所有卷的Segment
Unit，都会根据存储池内每个磁盘容量大小来均衡地承载Segment
Unit分布，磁盘空间得到充分使用，卷的性能得到最大化的发挥。负载均衡期间不会影响存储业务的正常访问。负载均衡可以人工触发也可以自动触发，用户可根据负载均衡的速度和和对业务的影响程度选择不同级别的负载均衡规则。

- **创建负载均衡策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“负载均衡”。

<img src="https://zdbs.io/manual/media/image132.png" />

2. 选择“创建”，输入“策略名称”、“相对时间”、“绝对时间”，选择“创建”。

<img src="https://zdbs.io/manual/media/image133.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “策略名称”只能输入中英文、数字、“\_”，长度2-64，“策略名称”不能重复。

2. “相对时间”指卷可用且稳定后的时间，至少为1分钟。

3. “绝对时间”指具体时间区间。

- **查看负载均衡策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“负载均衡”，查看负载均衡详细信息。

<img src="https://zdbs.io/manual/media/image134.png" />

- **修改负载均衡策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“负载均衡”，选择策略名称，选择“操作”，选择“修改”，修改相关参数，选择“修改”。

<img src="https://zdbs.io/manual/media/image135.png" />

<img src="https://zdbs.io/manual/media/image136.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. 负载均衡策略支持修改“策略名称”和“相对时间”。

2. 策略修改之后即时生效。

- **关联负载均衡策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“负载均衡”，选择策略名称“操作”，选择“关联设置”。

<img src="https://zdbs.io/manual/media/image137.png" />

2. 选择“存储池名”，选择“应用”或“撤销”，关联存储池或撤销关联，选择“完成”。

<img src="https://zdbs.io/manual/media/image138.png" />

- **开启或关闭负载均衡策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“负载均衡”，选择右上角开关，开启或关闭QoS策略。

<img src="https://zdbs.io/manual/media/image139.png" />

<img src="https://zdbs.io/manual/media/image140.png" />

- **删除负载均衡策略**

1. 在左侧导航栏选择“存储”--\>“QoS策略”，选择“负载均衡”，勾选策略名称，选择“删除”，选择“删除”，删除QoS策略。

<img src="https://zdbs.io/manual/media/image141.png" />

<img src="https://zdbs.io/manual/media/image142.png" />

## 七、硬件管理


1. ### 磁盘管理

存储磁盘页面展示了DataNode节点的磁盘详情，包括磁盘名称、磁盘类型、存储类型、主机IP、状态、所属域、所属存储池、槽位、逻辑容量、已分配容量、操作。

- **磁盘查询**

1. 在左侧导航栏选择“硬件”--\>“存储磁盘”。

<img src="https://zdbs.io/manual/media/image143.png" />

2. 在右侧查询栏选择“状态”和“存储类型”，选择“查询”。

<img src="https://zdbs.io/manual/media/image144.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

1. “状态”分为“所有状态”、“健康”、“错误”。

2. “存储类型”分为“所有存储类型”、“SATA盘”、“SAS盘”、“SSD盘”、“NVME盘”。

## 八、系统管理

### 系统服务管理

#### 服务界面

服务界面向用户展示系统当前所有服务器上的服务列表及服务状态，包括DataNode、DIH、DriverContainer、InfoCenter等服务。<img src="https://zdbs.io/manual/media/image145.png"
alt="Screenshot from 2019-01-08 18-04-03" />代表服务是正常状态，<img src="https://zdbs.io/manual/media/image146.png"
alt="Screenshot from 2019-01-08 18-05-50" />代表服务是告警状态，<img src="https://zdbs.io/manual/media/image147.png"
alt="Screenshot from 2019-01-08 18-05-05" />代表服务是异常状态，<img src="https://zdbs.io/manual/media/image148.png"
alt="Screenshot from 2019-01-08 18-03-15" />代表服务是挂起状态。

1. 在左侧导航栏选择“系统”--\>“服务”，查看服务。

<img src="https://zdbs.io/manual/media/image149.png" />

#### 服务管理

- **服务管理**

1. 在左侧导航栏选择“系统”--\>“服务”，选择“管理”，查看所有服务的服务名称、状态、所在组编号、主机IP和端口，其中服务状态包括正常、挂起、停止、失败、丢失、未知。

<img src="https://zdbs.io/manual/media/image150.png" />

#### 查询服务

1. 在左侧导航栏选择“系统”--\>“服务”，选择“管理”，输入“查询服务名称”、“状态”，自动列出查询服务，其中状态包括所有状态、健康、告警、错误。

<img src="https://zdbs.io/manual/media/image151.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

查询服务名称支持模糊查询。

### 组件管理

- **查看zookeeper组件**

1. 在左侧导航栏选择“系统”--\>“组件”，选择“zookeeper”，查看zookeeper组件。

<img src="https://zdbs.io/manual/media/image152.png" />

2. zookeeper列表包含“节点IP”和“服务状态”。

<img src="https://zdbs.io/manual/media/image153.png" />

### 操作日志管理

ZettaStor分布式块设备存储系统中用户所有操作都记录在操作日志列表中，包含用户名、操作类型、目标类型、操作对象、状态、开始时间、结束时间和错误信息，方便对系统的操作进行查看、管理和溯源。

- **查询日志**

1. 在左侧导航栏选择“系统”--\>“操作日志”，查看操作日志。

<img src="https://zdbs.io/manual/media/image154.png" />

2. 输入用户名、操作目标、操作类型、目标类型、状态、开始时间和结束时间，选择“查询”。

<img src="https://zdbs.io/manual/media/image155.png" />

## 九、用户管理

### 角色管理

ZettaStor分布式块设备存储系统支持角色管理，管理员可以给用户分配不同角色，使用户访问系统更安全、便捷和灵活。

- **查看角色**

1. 在左侧导航栏选择“用户”--\>“角色”。

<img src="https://zdbs.io/manual/media/image156.png" />

2. 选择角色名称，查看权限分配和详细信息。

<img src="https://zdbs.io/manual/media/image157.png" />

- **创建角色**

1. 在左侧导航栏选择“用户”--\>“角色”。

<img src="https://zdbs.io/manual/media/image158.png" />

2. 选择“创建”，输入“角色名”、“描述”，勾选相关权限，选择“创建”。

<img src="https://zdbs.io/manual/media/image159.png" /><img src="https://zdbs.io/manual/media/image160.png" />

<img src="https://zdbs.io/manual/media/image161.png" />

3. 查看角色信息。

<img src="https://zdbs.io/manual/media/image162.png" />

- **修改角色**

1. 在左侧导航栏选择“用户”--\>“角色”，勾选角色名称。

<img src="https://zdbs.io/manual/media/image163.png" />

2. 选择“修改”，修改“角色名”、“描述”，勾选相关权限，选择“更新角色”。

<img src="https://zdbs.io/manual/media/image164.png" /><img src="https://zdbs.io/manual/media/image165.png" />

<img src="https://zdbs.io/manual/media/image166.png" />

3. 查看角色信息。

<img src="https://zdbs.io/manual/media/image167.png" />

- **删除角色**

1. 在左侧导航栏选择“用户”--\>“角色”，勾选角色名称。

<img src="https://zdbs.io/manual/media/image168.png" />

2. 选择“删除”，选择“删除”。

<img src="https://zdbs.io/manual/media/image169.png" />

3. 查看角色信息。

<img src="https://zdbs.io/manual/media/image170.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

admin和user是系统内建角色，无法删除。

### 用户管理

用户登录Console需通过用户名和密码验证、鉴权，超级管理员拥有所有权限和一切资源。

- **查看用户**

1. 在左侧导航栏选择“用户”--\>“用户”。

<img src="https://zdbs.io/manual/media/image171.png" />

2. 选择用户名称，查看资源分配和详细信息。

<img src="https://zdbs.io/manual/media/image172.png" />

- **创建用户**

1. 在左侧导航栏选择“用户”--\>“用户”。

<img src="https://zdbs.io/manual/media/image173.png" />

2. 选择“创建”，输入用户名、密码、确认密码、角色分配，选择“创建”。

<img src="https://zdbs.io/manual/media/image174.png" />

3. 查看用户信息。

<img src="https://zdbs.io/manual/media/image175.png" />

- **重置密码**

1. 在左侧导航栏选择“用户”--\>“用户”，勾选用户名称。

<img src="https://zdbs.io/manual/media/image176.png" />

2. 选择“重置密码”，选择“重置”。

<img src="https://zdbs.io/manual/media/image177.png" />

<img src="https://zdbs.io/manual/media/image5.png" />

重置后的密码为admin。

- **更新角色**

1. 在左侧导航栏选择“用户”--\>“用户”，勾选用户名称。

<img src="https://zdbs.io/manual/media/image176.png" />

2. 选择“更新角色”，通过“+”和“-”重新分配角色，选择“更新”。

<img src="https://zdbs.io/manual/media/image178.png" />

3. 查看用户信息。

<img src="https://zdbs.io/manual/media/image179.png" />

- **资源分配**

1. 在左侧导航栏选择“用户”--\>“用户”，勾选用户名称。

<img src="https://zdbs.io/manual/media/image176.png" />

2. 选择“资源分配”，勾选相关资源，选择“确定”。

<img src="https://zdbs.io/manual/media/image180.png" />

3. 选择用户名称，查看资源分配和详细信息。

<img src="https://zdbs.io/manual/media/image181.png" />

- **删除用户**

1. 在左侧导航栏选择“用户”--\>“用户”，勾选用户名称。

<img src="https://zdbs.io/manual/media/image176.png" />

2. 选择“删除”，选择“删除”。

<img src="https://zdbs.io/manual/media/image182.png" />
