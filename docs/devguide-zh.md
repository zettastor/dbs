## 一、系统架构

### 功能架构

ZettaStor
DBS的总体功能框架如下图所示。存储资源层由标准服务器和通用以太网设备构成，负责为存储系统提供底层物理资源；存储平台层负责分布式系统的构建及核心功能的实现；存储服务层负责为客户端提供具备企业级功能特性、满足企业级环境高可靠及高可用性要求的存储资源服务；存储接口层负责提供相关访问协议及接口，供客户端接入并访问存储资源；存储管理负责系统内部各类管理功能的实现，并为用户管理员提供操作界面，同时也负责提供API或特定集成接口，以实现与云平台或第三方管理平台的对接。

<img src="https://zdbs.io/devguide/media/image1.png" />

### 软件架构

ZettaStor
DBS由一系列承担不同任务的软件模块构成，通过模块之间的通信与数据传递来实现分布式存储系统的各项功能。如图表3所示，这些模块可以分成数据I/O处理相关和控制操作相关两类：

- 数据I/O处理相关。这些模块负责管理底层存储资源，建立并管理I/O通路，接受用户数据访问请求并完成数据的读写操作；

- 控制操作相关。这些模块与用户数据访问过程分离，负责收集并呈现系统内各类信息，提供用户管理界面及接口，以及完成系统各项管理任务。

<img src="https://zdbs.io/devguide/media/image2.jpg" />

## 二、各服务之间数据流

### 控制流

控制操作相关软件模块及数据流基本构成如图所示，通过Thrift协议进行通信：

<img src="https://zdbs.io/devguide/media/image3.png" style="width:50%" />

- InfoCenter

InfoCenter是系统的信息中心，负责从DataNode获取包括DataNode和后端存储资源信息、块设备挂载和使用信息、及账户信息在内的各类信息，供MonitorServer及Console使用。

- SystemDaemon

SystemDaemon作为监控探针，用于检测节点硬件信息以及存储各类服务产生的日志信息。

- Console

Console为系统的控制台，用户可以通过Web浏览器方式进行系统配置、查看各类系统信息，包括：各部件运行状态，各类故障或告警，以及I/O性能数据及报告等操作。

以上各个软件模块均与数据I/O处理分离，模块发生故障仅影响管理类操作，而不会影响数据的正常读写访问。为避免控制操作相关模块故障给系统管理带来不便，InfoCenter、Console模块采用分布式架构设计采用HA方式部署在集群内多个不同节点上，相互保护，当其中一个节点故障，另一个可快速接管工作。

### 数据流

数据I/O处理相关软件模块包括客户端驱动、Coordinator以及DataNode，数据流使用Netty4.0框架，通过Protocol
buffer编解码方式进行通信。

- 客户端驱动

客户端驱动负责与Coordinator进行通讯，以识别由ZettaStor DBS系统所提供的标准块存储设备，并建立数据访问通路。客户端驱动还将所识别到的块设备提交给操作系统进行读写访问，将接收到的用户I/O请求提交给Coordinator，并返获取的执行结果。

客户端驱动可以是标准的iSCSI
Initiator，用以访问Coordinator基于iSCSI协议提供的块设备；也可以是鹏云PYD客户端驱动，用以访问Coordinator基于私有LBD协议（Lightning
Block
Device）提供的块设备。前者具有更为广泛的适应性，后者提供更高的性能表现。

<img src="https://zdbs.io/devguide/media/image1n.png" style="width:75%" />

- Coordinator

每个Coordinator负责管理一个标准块存储设备。Coordinator作为服务端来响应客户端驱动所提交的I/O请求并进行寻址，再将I/O请求分发给对应的存储节点服务器上的DataNode进程，并向客户端驱动返回处理结果。

- DataNode

DataNode模块部署于每台存储节点服务器上，是物理存储资源的实际管理者，以及数据读写操作的实际执行者。DataNode模块承担以下任务：

识别并管理所在存储节点服务器内部的硬盘，按照自定义的数据结构对磁盘进行格式化并管理每块硬盘上存储空间的使用；

接收Coordinator分发的读写请求，进行数据读写操作；

维护本地数据与其它DataNode上数据之间的副本关系，并在故障时执行主从副本切换、数据重构等操作。

## 三、产品特性介绍

### 节点分组

ZettaStor
DBS节点分组，即Group。针对存储节点服务器的组织形式，以保证数据可靠性和系统可用性为主要目的。

### 存储域

ZettaStor
DBS存储域，即Domain。以存储节点为粒度进行资源划分并管理，可以有效地进行故障隔离。

### 存储池

ZettaStor DBS存储池，即Storage
Pool。针对硬盘等存储资源的组织形式，可按介质类型划分不同的存储池，以适应不同类型应用的需求。

### 副本共存

ZettaStor
DBS系统针对数据卷来设置数据副本个数。数据卷可支持两副本，其中一个作为主副本（Primary），另外一个作为从副本（Secondary），还存在一个仲裁副本（Arbiter）；数据卷可支持三副本，其中一个作为主副本（Primary），另外两个作为从副本（Secondary）。数据副本的个数越多，能容忍的故障节点数就越多，数据卷的可靠性及可用性级别就越高。

### QoS策略

ZettaStor
DBS系统可以为创建的卷提供QoS保障，通过对卷的I/OPS和带宽进行限制，确保关键应用能够获得足够的I/O资源。

QoS机制一方面能确保卷的性能按需、保质供给，另一方面可以把更多的I/O资源分配给对I/O资源要求高的应用，避免互相竞争。

支持对卷、卷组进行吞吐量I/OPS和带宽设置上限。系统限制卷、卷组的I/OPS和带宽不超过配置的上限，同时系统保证为卷、卷组预留的I/OPS和带宽资源不低于配置的下限。

### 数据重构

ZettaStor
DBS系统基于分布式数据副本技术来保障数据可靠性和可用性。每一份数据在系统中都会存储至少两个副本，并分布在不同存储节点上，这样当某个存储节点或该存储节点的磁盘介质发生故障时，只会影响其中一个数据副本，用户数据不会丢失，访问也不会中断。系统会自动将受影响的数据副本重构到其它位置，数据卷从而重新恢复至稳定状态。

<img src="https://zdbs.io/devguide/media/image4.png" />

当硬盘离线、节点宕机，网络异常或者亚健康时，受影响的Segment会有两种情况：主副本缺失，或某一从副本缺失。如主副本缺失，会在剩余的健康Segment
Unit之间选举出新的主副本来继续响应I/O请求；如从副本缺失，则主副本持续响应I/O请求。

当达到某种触发条件时，系统会进行数据重构。从当前主副本复制Segment
Unit到同一存储池中的其它位置，新位置的选择首先会保证从属于同一Segment的Segment
Unit不会出现在同一组内的存储节点上，其次会尽量保证各个组内存储节点上数据的均衡分布。图表9简要描述了三副本配置情况下的数据重构操作流程。

<img src="https://zdbs.io/devguide/media/image5.png" />

数据的重构是由存储池内所有硬盘，以及硬盘所属存储节点共同参与的，数据的复制是完全交叉分布的，而不会出现从某一位置集中读取，或向某一位置集中写入的情况。这样就使得数据重构能够快速完成，同时又避免了性能热点。

数据重构需要进行跨节点的数据复制操作，会占用一定的带宽和主机资源。用户可以对数据重构操作的工作模式进行设定，如设定为业务优先模式，则后台会自动控制数据重构操作以减少资源占用，减少对业务的影响；如设定为数据优先模式，则优先完成数据重构，以使系统尽快恢复稳定状态，保障数据安全性及系统可用性。

### 数据再均衡

智能数据再平衡（Rebalance）是指当系统中的存储节点之间出现资源利用不均衡时，通过数据迁移重新达到平衡的过程。

<img src="https://zdbs.io/devguide/media/image2n.png" />

在系统运行过程中，很多原因都会造成资源利用不均衡的情况出现，例如向系统内增加新的节点或磁盘，原有数据卷的删除，数据卷快照等。资源不均衡会导致资源利用不充分，造成浪费。同时也可能会造成I/O负载的分布不均衡，影响系统整体的性能表现。

ZettaStor DBS系统支持手动数据再平衡和智能数据再平衡两种类型的再平衡策略，能够根据不同情况灵活采用。还可以限制数据再平衡操作的执行时间窗口，或者执行过程对I/O带宽的占用，以避免对业务应用的正常处理造成影响。

- 智能触发

在ZettaStor系统中，触发数据再平衡操作迁移操作的条件包括：

1.  某个DataNode的存储空间利用率大幅偏离系统平均值。
2.  某个DataNode上主副本分布比例大幅偏离系统平均值。

- 智能迁移

智能选择要把哪些Segment Unit迁移至其它DataNode，选择依据包括：

1.  除非是为了平衡主副本的分布，否则尽量选择从副本。
2.  尽量选择使用率低的，以降低迁移操作带来的性能影响。

## 四、关键业务流程

### 磁盘初始化

ZettaStor DBS直接以裸设备的方式管理存储节点服务器上的硬盘设备。

在ZettaStor
DBS系统中，将所识别并完成初始化操作的硬盘（包括磁盘及固态盘）称为Archive，并将其存储空间划成分为一个个指定大小的Segment
Unit，作为执行存储空间分配的基本单元。

ZettaStor
DBS管理下的Archive内部格式如图表6所示。其中，位于硬盘起始位置的Archive
Metadata负责记录自身存储空间的使用和Segment
Unit的分布信息。在每一个Segment
Unit的起始位置也有相应的Metadata记录自身相关信息。

<img src="https://zdbs.io/devguide/media/image6.jpg" />

### 数据卷创建

ZettaStor DBS数据卷，即Volume。ZettaStor
DBS以数据卷的形式将存储资源提供给客户端主机使用，客户端使用数据卷，如同使用本地磁盘。

### 缓存机制

ZettaStor
DBS采用多级分布式缓存。其中，在每台存储节点服务器的内存中开辟专门区域作为第一级缓存区，使用每台存储节点服务器内部的固态硬盘（SAS/SATA接口或PCI-e接口）作为可选的第二级缓存区，两级缓存统一管理。各节点的缓存区之间遵循Shared
Nothing的全对称分布式，在高度独立自治的同时，还基于私有P2P协议及专利技术高效协同工作。

<img src="https://zdbs.io/devguide/media/image7.png" />

图表7

缓存空间的页面置换管理采用LRU（Least Recently
Used），即近期最少使用算法。系统会将最近使用频率最低的数据块移出缓存而腾出空间来加载其他新近被请求的数据块，以获得更高的缓存命中率。

### 写I/O流程

写请求会先发送到Coordinator，Coordinator在接收到用户的写请求时，会根据卷组成信息，执行条带策略进行计算，决定用户的数据落在哪个Segment上。

条带策略的实现方式是：每个卷独立拥有自己的固定条带大小，大小为Page
Size整数倍，具体倍数可以根据用户业务场景指定。根据用户I/O根据I/O请求起始位置及卷组成信息，计算出该起始位置应该落在哪个Segment的哪个位置，计算完一个条带后如果还有请求数据，则继续计算下一个条带对应的Segment以及位置，以此类推。

假设用户发起一个写请求，写请求起始位置为0，大小为5MB，经过条带换算后，则会有5个小块写请求，分别为\[0,1MB)、\[1MB,2MB)、\[2MB,3MB)、\[3MB,4MB)、\[4MB,5MB)，写入顺序如下：

1.  \[0,1MB)会从第0个Segment的0位置开始写入，长度为一个条带大小：1MB；
2.  \[1MB,2MB)会从第1个Segment的0位置开始写入，长度为一个条带大小：1MB；
3.  \[2MB,3MB)会从第2个Segment的0位置开始写入，长度为一个条带大小：1MB；
4.  \[3MB,4MB)会从第3个Segment的0位置开始写入，长度为一个条带大小：1MB；
5.  \[4MB,5MB)会从第4个Segment的0位置开始写入，长度为一个条带大小：1MB。

<img src="https://zdbs.io/devguide/media/logicw.png" />

引入条带策略的目的是用户在进行大块顺序写的时候保证用户的写请求均匀的落在所有数据盘上，保证磁盘级别的负载均衡，而不是对着某几块盘写，导致某些磁盘成为性能瓶颈。

Coordinator根据数据的Offset计算好数据所在的Segment，会先根据Segment的Membership来决定数据发送的目标和发送的个数。Membership中包含了节点信息的节点状态，其中Primary、Secondary和JoinSecondary状态的Segment
Unit需要发送数据；Arbiter状态的Segment
Unit是稳定状态，但是Arbiter只是作为voting的仲裁者并不存储数据；InActiveSecondary的Segment
Unit说明Segment
Unit暂时无法正常写入。Coordinator不需要发送写请求。Coordinator会以广播的方式同时发送写请求给所有的Active
Segment Unit所在的DataNode。

Coordinator在广播之后会统计所有Segment
Unit的返回结果，并根据Volume的VolumeType来确定写结果：

两副本：稳定状态下要求写两副本成功；在发生宕机磁盘等问题是可以允许单副本写入就算成功，此时保证了环境的可用性但是增加了数据丢失的风险。稳定状态下只允许一个副本损坏或者丢失。

三副本：Primary所在副本一定要写成功，任意一个Secondary写成功即算成功。稳定状态下只允许一个副本损坏或者丢失。

三副本高可用：稳定状态下要求写三副本成功；在发生宕机磁盘等问题是允许单副本写入就算成功，此时保证了环境的可用性但是增加了数据丢失的风险。稳定状态下允许两个副本损坏或者丢失。

在三副本模式下，写I/O操作只需得到两个数据副本的成功确认即可认为写入成功，其它数据副本可以通过节点间的缓存比对和同步来保证数据成功写入。这样就避免了某些存储节点的网络或主机发生异常而造成写入延迟的增加。

DataNode在接收到Coordinator的写请求时会在内存中创建对应的Log。Primary会对为每个Log产生一个唯一且自增的LogId，并将该LogId广播给所有的Secondary，并收集所有Secondary是否都已经收到LogId和Data，并将结果返回给Coordinator，如果满足条件（条件和Coordinator判断写成功的条件相同）则返回用户写成功，否则就一直重发并进行上述处理，直到I/O超时。

在响应Coordinator之后就可以把数据真正落盘了，数据落盘的最小单位是Page，一份数据一定属于一个Page，以Page为单位落盘有两个好处：

1.  当多份数据落在同一个Page里的时候，并不需要写多次，而是在内存中将这个Page对应数据按照LogId的顺序进行整理，组成一份数据写入磁盘，提高了磁盘性能。
2.  写Page的时候按照Page对应的磁盘Offset进行排序，这样做的好处是将用户的随机I/O整理成半顺序I/O，以提高磁盘的吞吐量。

### 读I/O流程

读请求先发送到Coordinator，Coordinator根据卷组成信息并通过条带策略计算出用户所需数据所在的Segment
Index集合后，会同时向Segment中的主副本（Primary）发送读请求，向其他副本发送Check读请求，Check读请求只会检查Membership是否一致，Primary是否发生变化，而不会真正的读数据。这样做的好处是在Membership没有发生动荡的时候并不需要向多节点发送读请求从而提高性能，在发生动荡的时候又能及时感知变化，防止Membership发生脑裂，导致数据读取不一致情况产生。

<img src="https://zdbs.io/devguide/media/logicr.png" />