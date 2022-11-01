### 文档简介

本文档为公司分布式块存储产品的配置说明文档，适用于 `v1.0_OpenSource` 分支版本，对于其它分支版本的配置仅做参考。本文档用于指导工程师在使用产品的时候，能够详细了解各个模块的参数及其作用，进而可以根据用户需求、现场硬件环境以及需求规划，进行合理的配置。

### 更新记录

2022-08-30

结合产品最新的配置进行解释说明

2022-09-30

评审后发布

##  环境配置概览

本章节主要对产品使用中的一些参数进行介绍。使用者可以针对各自需求及环境特点对参数进行更改，使得产品能够满足用户需求，并发挥较优性能。

通常情况下，在配置文件中没有提及的属性，在部署完毕之后没有影响正常功能使用的情况下，无需在配置文件中额外添加，做显式说明。

产品环境配置主要是指 `/opt/pengyun-deploy/config` 目录下的配置文件及内容。这些配置数据的修改一般在部署之前完成或者完成修改时再进行升级操作，如果修改了 xml 配置文件，需要执行 `perl bin/update_system_config.pl` 脚本更新本地版本配置，然后再进行部署或者升级操作。将配置应用于环境中的对应的节点和服务中。

环境配置主要修改 `/opt/pengyun-deploy/config` 目录下的如下配置文件：

| **配置文件**         | **说明**                                                                |
|---------------------|-------------------------------------------------------------------------|
| zookeeper.properties | ZooKeeper服务部署                                                       |
| deploy.properties    | 主要描述各个服务名称、版本、节点信息、服务端口以及服务部署时长          |
| module_settings.xml  | 修改对应的\*.properties部分某些属性值，使得服务按照期望的属性值进行运行 |

##  服务和组件分布规划

### 服务分布规划

本章节主要讲解 `deploy.properties` 文件在配置过程中需要更新的字段以及所表示的含义。

| **字段名称**                                  | **默认值** | **配置说明**                                                                                                                                                                   |
|-----------------------------------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| remote.network                                | 网段/掩码    | 从部署节点访问集群节点进行部署等相关操作所使用的网段信息，通常情况下为控制网段的网络信息。                                                                                     |
| remote.user                                   | root         | 部署等相关操作登陆到集群节点的用户名称，通常是root用户或者是拥有root权限的其它用户。                                                                                           |
| remote.password                               |              | 部署等相关操作登陆到集群节点的用户所对应的密码，根据实际情况填写。密码中不要包含单引号、双引号、反引号、斜杠、反斜杠等这些容易引起字符转移的特殊字符。                         |
| remote.platform                               | x86_64       | 集群节点的平台类型，本存储产品支持主流的信创平台，根据实际环境信息填写。                                                                                                       |
| platform.update                               | false        | 是否需要根据实际的硬件信息来更新平台类型，对于主流的信创平台可以设置为ture来自动更新平台类型字段。                                                                             |
| jdbc.type                                     | postgresql   | 集群所使用的数据库类型，需要跟xml文件配置中的内容保持一致，通常是postgresql类型。                                                                                              |
| jdbc.user                                     | py           | 数据库访问的用户名，根据实际情况填写。                                                                                                                                         |
| jdbc.password                                 | 312          | 数据库访问的密码，根据实际信息填写                                                                                                                                             |
| XXX.dir.name                                  |              | 服务的运行目录，通常是在节点的/var/testing/packages目录下                                                                                                                      |
| XXX.version                                   |              | 通常不需要改动                                                                                                                                                                 |
| XXX.deploy.host.list                          |              | 服务的节点信息，冒号表示连续的IP地址，逗号表示离散的IP地址                                                                                                                     |
| XXX.deploy.port                               |              | 通常不需要改动                                                                                                                                                                 |
| XXX\. deploy.agent.jmx.port                   |              | 通常不需要改动                                                                                                                                                                 |
| DIH.center.host.list                          |              | DIH服务特有的字段，需要跟xml文件配置中 `center.dih.endpoint` 属性的IP地址保持一致                                                                                                  |
| XXX.remote.timeout                            |              | 服务部署等相关操作的超时时间，到达设定时间之后服务还没有达到期望状态认定为超时。该字段用于防止操作无限制等待下去，超时不会对已经执行的操作做其它额外处理。通常此字段不需要改动 |
| DataNode.deployment.host.group.enabled        | false        | DataNode服务特有的字段，是否指定存储节点的DataNode服务的Group ID，配置为false，由产品自动对存储节点进行分组；配置为true，根据指定的IP地址来进行分组                            |
| DataNode.deployment.host.group.x=\*\*\*\*\*\* |              | `DataNode.deployment.host.group.enabled` 设置为true，此配置生效，用于指定存储节点的Group ID，必须要配置3个以及以上不同的Group以及对应的节点IP地址                                 |

对于上表中XXX所表示的服务名称，在进行配置的时候，可以参考如下信息进行规划。

| **服务名称**      | **分布规划**                                                                        |
|-------------------|-------------------------------------------------------------------------------------|
| DIH               | 集群所有节点信息                                                                    |
| InfoCenter        | 通常选取2-3个节点进行配置，建议选取的节点不要在同一个Group里面                      |
| SystemDaemon      | 集群所有节点信息                                                                    |
| DriverContainer   | 根据需要配置，通常是集群所有节点信息                                                |
| DataNode          | 提供存储资源的节点，通常是集群所有节点信息                                          |
| Console           | 通常是选择一个或者多个节点进行配置。多节点的话，建议选取的节点不要在同一个Group里面 |
| deployment_daemon | 集群所有节点信息                                                                    |
| Coordinator       | 与DriverContainer节点配置一致                                                       |

### 组件ZooKeeper分布规划

本章节主要讲解 `zookeeper.properties` 文件在配置过程中需要更新的字段以及所表示的含义。

| **字段名称**    | **默认值** | **配置说明**                                                                                                                                                                                                                                    |
|-----------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| remote.user     | root         | Zookeeper组件部署等相关操作登陆到集群节点的用户名称，通常是root用户或者是拥有root权限的其它用户                                                                                                                                                 |
| remote.password |              | 部署等相关操作登陆到集群节点的用户密码，根据实际情况填写。密码中不要包含单引号、双引号、反引号、斜杠、反斜杠等这些容易引起字符转移的特殊字符。                                                                                                  |
| server.X        |              | 部署ZooKeeper服务的节点IP，通常使用控制网络的IP地址，X从1开始，数量必须为奇数个，通常为3-7个节点，节点数量约占总集群数量的一半以上，建议不超过7个，ZooKeeper节点较为均衡地分布在不同的Group里面，节点信息与xml文件中ZooKeeper相关的配置保持一致 |

##  xml文件配置

本章节主要介绍 `config` 目录下 xml 文件中各项参数的作用及其配置。

在 xml 文件中，属性值的设置格式如下：

> `<property name="name_value" value="value_string"/>`

第一个双引号中的内容为属性名称，第二个双引号中的内容为属性值。

通常物理机环境下，除了涉及IP地址、分组数量、磁盘数量之外的参数，不需要对xml文件中配置做大量更改。

如果某些配置在xml文件中没有出现，进行配置时，依据project name、file
name、property name进行添加即可（注意格式）。

### 通用配置

对于某些配置参数，在各个模块中都需要使用，因此没有必要在每个模块中都进行设置。这时候设置 `<project
name="*">` 表示对所有模块都生效。如果某个模块进行了单独设置，以模块设置的属性值为准。如果某个模块需要对某些参数设置其它的值，通常是 `jvm.properties`，在模块下面单独设置即可。

#### log4j.properties

| **属性名称**                       | **默认值** | **参数说明**                                                                                               |
|------------------------------------|--------------|------------------------------------------------------------------------------------------------------------|
| log4j.rootLogger                   | WARN, FILE   | 日志文件中记录的日志级别，默认情况下只打印高于此级别的日志，可选值为ERROR, FILE，DEBUG, FILE，INFO, FILE   |
| log4j.appender.FILE.MaxFileSize    | 200MB        | 每个日志文件的最大大小，根据存储节点/var/testing目录可使用的最大空间的实际环境作适当调整，可选单位为GB，MB |
| log4j.appender.FILE.MaxBackupIndex | 15           | 设置log日志的文件数量，当前日志文件名称为\*.log，历史日志文件名称为\*.log.x，x最大取值就是该参数的值       |

#### network.properties

| **属性名称**                    | **默认值** | **参数说明**                                                              |
|---------------------------------|--------------|---------------------------------------------------------------------------|
| control.flow.subnet             |              | 控制网络信息                                                              |
| monitor.flow.subnet             |              | 监控网络信息段，通常与control.flow.subnet的值保持一致。                   |
| enable.data.depart.from.control | true         | 控制、监控、数据、业务网络是否分离，当使用2个以及以上网段的时候配置为true |
| data.flow.subnet                |              | 存储私网网段，存储节点之间数据通信使用，通常为万兆网络或者IB网络          |
| outward.flow.subnet             |              | 业务网络信息，计算节点与存储进行IO传输所使用的网络                        |

#### storage.properties

| **属性名称**      | **默认值** | **参数说明**                                                                                                                                                                                                                                                                                                                                                                                                         |
|-------------------|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| page.size.byte    | 4096         | page size大小，实际环境使用的磁盘physical size是4096，配置为4096；使用的磁盘physical size是512，配置为8192，此配置跟page.metadata.need.flush.to.disk配置相关联                                                                                                                                                                                                                                                       |
| segment.size.byte | 1073741824   | segment size大小，单位是字节，默认大小为1G，最直接的理解是卷容量的最小单位，单个节点segment的数量理想值建议不超过5000左右，理论上最大极限值为15000。，为了充分利用磁盘上的空间，在进行配置的时候，使用单个节点的磁盘总容量，除以segment数量（通常为5000）选取离2的N次方最接近的数值，例如2G、4G、8G等，系统中总的segment unit的数量不要超过10万.在进行此参数的配置时，还需要考虑节点上磁盘是否存在后续库容的可能性。 |
| io.timeout.ms     | 60000        | I/O超时时间，通常情况下无需调整，对于虚拟机环境或者网络延时比较大情况，可适当调整。                                                                                                                                                                                                                                                                                                                                  |

### JVM配置

文件 `jvm.properties` 的主要作用是限制（或者称为配置）每个服务运行所使用的内存大小（每个服务都是由一个或者多个JAVA进程组成的）。`jvm.properties` 在除Console之外的其余模块都需要配置，但是每个模块运行时所需要设置的值是不一样的，因此需要根据环境硬件信息，针对每个服务单独进行配置。对于DriverContainer模块，还存在 `coordinator-jvm.properties`，配置参数与 `jvm.properties` 中字段含义类似，对于DataNode模块，其服务运行时还需要申请其它的内存资源，配置相对复杂，具体参考DataNode模块中详细介绍。

| **属性名称**          | **默认值** | **参数说明**           |
|-----------------------|--------------|------------------------|
| initial.mem.pool.size | xxm/xxg      | 运行时初始分配内存大小 |
| max.mem.pool.size     | yym/yyg      | 运行时占用最大内存大小 |

### 组件配置

这里集中介绍一下在多个模块中需要使用到的配置，比如数据库、ZooKeeper的配置，这些配置在InfoCenter、Coordinator等模块都可能需要用到，并且配置内容需要保持一致。

#### ZooKeeper配置

为了保证系统的稳定性，对于InfoCenter服务，通常部署在2个或者2个以上节点上，其中一个是Master节点，其余节点为Slave节点。当Master节点出现故障时，一个Slave节点转为Master节点，主备状态之间切换借助ZooKeeper来实现。

| **属性名称**                | **默认值**                        | **参数说明**                                                                                                         |
|-----------------------------|-------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| zookeeper.connection.string | ZK_IP1:2181, ZK_IP2:2181, ZK_IP3:2181 | ZooKeeper集群信息，ZK_IP为zookeeper.properties文件中配置的IP，节点数量必须为奇数个，通常使用控制网段的IP地址进行配置 |
| zookeeper.election.switch   | true                                | ZooKeeper开关，部署主备服务时必须开启                                                                                |

#### 数据库配置

在目前的软件环境中，数据库通常使用PostgreSQL方式（无论是单节点还是集群方式）。此配置涉及到InfoCenter模块。

PostgreSQL模式下的配置：

<table>
<colgroup>
<col style="width: 26%" />
<col style="width: 39%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><strong>属性名称</strong></td>
<td><strong>默认值</strong></td>
<td><strong>参数说明</strong></td>
</tr>
<tr class="even">
<td>jdbc.driver.class</td>
<td>org.postgresql.Driver</td>
<td>数据库驱动</td>
</tr>
<tr class="odd">
<td>jdbc.url</td>
<td><p>InfoCenter模块配置：</p>
<p>jdbc: postgresql://IP_address:5432/controlandinfodb</p></td>
<td>PostgreSQL数据库IP地址以及名称</td>
</tr>
<tr class="even">
<td>hibernate.dialect</td>
<td>py.db.sqlite.dialect.PostgresCustomDialect</td>
<td>PostgreSQL数据库方言</td>
</tr>
<tr class="odd">
<td>package.hbm</td>
<td>hibernate-config</td>
<td>PostgreSQL模式下配置</td>
</tr>
</tbody>
</table>

### Coordinator模块配置

#### coordinator.properties

|                                          |              |                                                                                                                                                     |
|------------------------------------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| **属性名称**                             | **默认值** | **参数说明**                                                                                                                                        |
| io.depth                                 | 128          | 队列深度，通常无需修改                                                                                                                              |
| enable.logger.tracer                     | false        | 日志跟踪，调试配置时使用，生产环境必须关闭                                                                                                          |
| trace.all.logs                           | false        | 跟踪所有日志，调试配置时使用，生产环境必须关闭                                                                                                      |
| debug.io.timeout.threshold.ms            | 1500         | I/O超时阈值                                                                                                                                         |
| network.checksum.algorithm               | DUMMY        | 网络校验算法，和datanode.properties中page.checksum.algorithm和network.checksum.algorithm取值保持一致，可选值为DUMMY，ALDER32，DIGEST，CRC32，CRC32C |
| ping.host.timeout.ms                     | 500          | Coordinator到DataNode网络检测超时时间，POC环境建议设置100，生产环境建议500                                                                          |
| network.connection.detect.retry.maxtimes | 3            | 网络检测重试次数                                                                                                                                    |
| network.healthy.check.time               | 3            | 网络检测频率                                                                                                                                        |

### DIH模块配置

#### instancehub.properties

|                     |                  |                                                                                            |
|---------------------|------------------|--------------------------------------------------------------------------------------------|
| **属性名称**        | **默认值**     | **参数说明**                                                                               |
| center.dih.endpoint | IP_Address:10000 | CenterDIH节点配置，IP_Address与deploy.properties文件中DIH.center.host.list配置的IP保持一致 |

### DataNode模块配置

#### datanode.properties

|                                          |              |                                                                                                                                                                                |
|------------------------------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **属性名称**                             | **默认值** | **参数说明**                                                                                                                                                                   |
| threshold.to.request.for.new.member.ms   | 1800000      | 新的Secondary加入segment的等待时间，单位是毫秒                                                                                                                                 |
| wait.time.ms.to.move.segment.to.deleted  | 300000       | 卷从删除之后可以执行回收操作的时间长度，超过此时间之后，卷无法进行回收操作，单位是毫秒                                                                                         |
| memory.size.for.data.logs.mb.per.archive | 300          | 每个磁盘需要的fastbuffer空间大小，节点上总fastbuffer空间大小为（磁盘数量+1）乘以该参数的值，通常无需修改，单位是MB                                                             |
| page.system.memory.cache.size            | 2G           | 系统页需要的内存空间，0表示为根据物理实际内存大小进行适配，不建议配置为0，内存不够可以根据实际环境进行缩减                                                                     |
| page.metadata.need.flush.to.disk         | false        | 元数据页刷到磁盘，0.5k扇区为true，4k扇区为false，此配置跟page.size.byte配置大小相关                                                                                            |
| delay.record.storage.exception.ms        | 2000         |                                                                                                                                                                                |
| page.checksum.algorithm                  | DUMMY        | 分页校验算法，和coordinator.properties中network.checksum.algorithm、datanode.properties中network.checksum.algorithm取值保持一致，可选值为DUMMY，ALDER32，DIGEST，CRC32，CRC32C |
| network.checksum.algorithm               | DUMMY        | 网络校验算法，和coordinator.properties中network.checksum.algorithm、datanode.properties中page.checksum.algorithm取值保持一致，可选值为DUMMY，ALDER32，DIGEST，CRC32，CRC32C    |
| archive.init.mode                        | overwrite    | DataNode初始化模式，可选值为append，overwrite，通常情况下不需要改动                                                                                                            |
| max.io.pending.requests                  | 5000         | Pending队列，通常无需修改                                                                                                                                                      |
| max.io.depth.per.hdd.storage             | 64           | 磁盘最大iodepth，通常无需修改                                                                                                                                                  |

#### jvm.properties

此处针对DataNode的JVM配置单独进行说明，如下是针对生产环境的建议值，实际环境中需要略做调整。

|                       |              |                                                  |
|-----------------------|--------------|--------------------------------------------------|
| **属性名称**          | **默认值** | **参数说明**                                     |
| initial.mem.pool.size | 40G          | 运行时初始分配内存大小                           |
| max.mem.pool.size     | 40G          | 运行时占用最大内存大小                           |
| parallel.gc.threads   | 15           | 并行线程数，该参数配置为单个节点CPU线程数5/8左右 |
| conc.gc.threads       | 4            | 该参数配置为parallel.gc.threads值1/4左右         |

### DriverContainer模块配置

#### coordinator-jvm.properties

该属性用于配置Coordinator内存设置，对应于Web界面上驱动内容，每挂载一个驱动都需要在对应的容器上消耗一定的内存资源。

|                          |              |                          |
|--------------------------|--------------|--------------------------|
| **属性名称**             | **默认值** | **参数说明**             |
| initial.mem.pool.size    | 1024m        | 每个驱动初始分配内存大小 |
| min.mem.pool.size        | 1024m        | 每个驱动最小内存需求     |
| max.mem.pool.size        | 1024m        | 每个驱动占用最大内存大小 |
| max.direct.memory.size   | 512m         |                          |
| netty.allocator.maxOrder | 7            |                          |

#### drivercontainer.properties

|                              |              |                                                                                                                                                                                                                                                                                                                       |
|------------------------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **属性名称**                 | **默认值** | **参数说明**                                                                                                                                                                                                                                                                                                          |
| system.memory.force.reserved | 512M         | 系统保留内存，驱动的挂载和卸载操作由DriverContainer执行，在挂载一个驱动之前，DriverContainer会计算当驱动挂载完毕之后，系统剩余内存是否还大于该值，如果挂载完毕之后系统剩余内容小于该值，就会提示内存资源不足，并且不会进行挂载操作，防止挂载完毕之后系统资源不足影响其它服务，在生产环境中该值可以适当调大，比如2048M |
| iet.target.flag              | false        | iSCSI实现方式，该字段设置为true表示iet方式，false表示cli方式                                                                                                                                                                                                                                                          |
| iscsi.portal.type            | IPV6         | Coordinator监听地址类型，该字段设置为IPV4表示只监听IPv4地址，IPV6表示监听IPv4和IPv6地址                                                                                                                                                                                                                               |

#### lioCommandManager.properties

|                              |                             |                                                        |
|------------------------------|-----------------------------|--------------------------------------------------------|
| **属性名称**                 | **默认值**                | **参数说明**                                           |
| default.saveConfig.file.path | /etc/target/saveconfig.json | 在RPM和DEB环境下，这个文件的路径不大一样，需要注意修改 |

#### liotarget.properties

|                 |                                                        |                                                                     |
|-----------------|--------------------------------------------------------|---------------------------------------------------------------------|
| **属性名称**    | **默认值**                                           | **参数说明**                                                        |
| saveconfig.path | /etc/target/saveconfig.json                            | saveconfig路径，CentOS和Ubuntu操作系统下路径不同，根据实际环境配置  |
| restore.command | /usr/bin/targetcli restore /etc/target/saveconfig.json | restore执行命令，CentOS和Ubuntu操作系统下路径不同，根据实际环境配置 |

### InfoCenter模块配置

#### infocenter.properties

|                                          |              |                                                                                                                                              |
|------------------------------------------|--------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| **属性名称**                             | **默认值** | **参数说明**                                                                                                                                 |
| is.arbiter.group.set                     | false        | 是否使用专门的组去创建arbiter功能，该字段设置为true时，需要通过arbiter.group.id设置arbiter组号                                               |
| arbiter.group.id                         | 0            | arbiter组号，当is.arbiter.group.set为true时生效                                                                                              |
| page.wrapp.count                         | 128          | 条带宽度，顺序读写时，操作每个segment的数据量为page.wrapp.count\*segment.size，完成后切换下一个segment                                       |
| segment.wrapp.count                      | 8            | 单节点上数据盘的数量，通常需要根据单个节点上实际数据盘的数量进行调整，如果环境中一个组内有多个节点的情况，该值需要修改为每个组内数据盘的数量 |
| group.count                              | 4            | 存储节点组数量，通常取值在3-5之间，对于两副本或者三副本卷来说，每个segment要分布在3个不同的组里面，该配置在部署完毕之后修改无效              |
| jdbc.url                                 | 略           | 相关的配置请参考数据库配置说明                                                                                                               |
| max.rebalance.task.count.volume.datanode | 50           | rebalance并发数，按照segment unit数量计算，在生产环境中该值建议设置为50                                                                      |

#### jvm.properties

此处针对InfoCenter的JVM配置单独进行说明，该配置与segment
unit数量有关系。

|                       |              |                                                                                                                                                                       |
|-----------------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **属性名称**          | **默认值** | **参数说明**                                                                                                                                                          |
| initial.mem.pool.size | 1024m        | 运行时初始分配内存大小                                                                                                                                                |
| max.mem.pool.size     | 2048m        | 运行时占用最大内存大小，该值与segment unit数量有关，segment unit数量在2万以内，该值建议配置为2G，segment unit数量超出2万，每2万增加1G，总的segment unit数量不超过10万 |

### SystemDaemon模块配置

#### systemdaemon.properties

|                 |              |                                         |
|-----------------|--------------|-----------------------------------------|
| **属性名称**    | **默认值** | **参数说明**                            |
| record.save.day | 15           | eventdata数据保留时间，根据实际环境配置 |

### DeploymentDaemon模块配置

此模块暂时没有配置需要单独说明。

##  配置更新方法

参数的配置并不是固定不变的，随着需求的变更，参数会出现删减的情况，受制于文档的限制，本文只列出了可能需要修改的部分参数（显式设置），还有一些参数并没有在文中进行说明（隐式设置）。

对于参数删除的情况，多余的参数并不会影响功能的使用。

对于参数增加的情况，需要在 xml 文件中增加对应的参数名称，并进行显式设置。

参数显式设置的方式如下：

- 在 xml 文件中定位出需要在哪些模块的哪些文件中添加或者修改参数

- 在 xml 文件中设置属性名称以及对应的取值

为了便于修改，建议在属性增加的时候，按照 xml 文件中的字母排序进行添加，设置完毕之后，务必在部署或者升级之前调用 `perl bin/update_system_config.pl` 更新本地软件包中的配置。
