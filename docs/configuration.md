### Introduction

This document is the configuration documentation for ZettaStor Distributed Block Storage product, for the branch `v1.0_OpenSource`. The configuration of other branches is only for reference. This document is used to guide engineers to understand the parameters and functions of each module in detail when using the product, and to make reasonable configurations according to user needs, on-site hardware environment, and demand planning.

### Release Notes 

2022-08-30
Explain in combination with the latest configuration of the product

2022-09-30
release after review

## Overview

This chapter introduces some configurable parameters. Users can change these parameters according to their own needs and environmental specifications, so that the product can meet the needs of users and achieve optimal performance.

Under normal circumstances, the parameters not mentioned in the document do not affect the functionality of the product after deployment, there is no need to add them explicitly to the configuration file.

The product environment configuration refers to the configuration files and contents under `/opt/pengyun-deploy/config` directory. Modification of these configuration data should be done before the deployment or upgrade operation is performed. If the xml configuration file is modified, you should execute `perl bin/update_system_config.pl` to update the local configuration, before the deployment or upgrade operation is performed. The new configuration will be applied and transferred to the corresponding nodes and services.

The environment configuration consists of the following files in the `/opt/pengyun-deploy/config` directory:

| **Configuration File** | **Description**                                                                |
|------------------------|-------------------------------------------------------------------------|
| zookeeper.properties | ZooKeeper service deployment |
| deploy.properties    | Service name, version, node information, service port and service deployment time |
| module_settings.xml  | Modify property values in the corresponding\*.properties, so that the service runs according to the specified properties |

## Distribution Planning

### Service Distribution Planning

This chapter explains the fields that need to be updated during the configuration process of `deploy.properties`.

| **Attribute Name** | **Default Value** | **Description**
|-----------------------------------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| remote.network                                | Net mask     | The network used to access cluster nodes for deployment and other related operations from the deployment node, usually it is the Control Network.                              |
| remote.user                                   | root         | The name of the user who logs in to the cluster node for deployment and other privileged operations, usually root or other users with root privileges.                         |
| remote.password                               |              | The password of the corresponding user. Do not include special characters such as single quotes, double quotes, backquote, slashes, backslashes, etc. that may cause characters to escape. |
| remote.platform                               | x86_64       | The platform architecture of the cluster node, this storage product supports mainstream platforms. |
| platform.update                               | false        | Whether to update the platform architecture according to the actual hardware information, for the mainstream platform, it can be set to true to automatically update the platform architecture field. |                                                                            |
| jdbc.type                                     | postgresql   | The database type used by the cluster needs to be consistent with the content in the xml file configuration, usually postgresql. |
| jdbc.user                                     | py           | The user name for database access. |
| jdbc.password                                 | 312          | Password for database access. |
| XXX.dir.name                                  |              | The running directory of the service, usually in the `/var/testing/packages` directory of the node |
| XXX.version                                   |              | Usually no changes needed. |
| XXX.deploy.host.list                          |              | The node information of the service, a colon indicates continuous IP addresses, and a comma indicates discrete IP addresses |
| XXX.deploy.port                               |              | Usually no changes needed. |
| XXX\. deploy.agent.jmx.port                   |              | Usually no changes needed. |
| DIH.center.host.list                          |              | DIH service-specific fields, need to be consistent with the IP address of the `center.dih.endpoint`. |
| XXX.remote.timeout                            |              | The timeout period for related operations such as service deployment, if the service has not reached the desired state after set time, it will be considered as timeout. This field is used to prevent the operation from waiting indefinitely. When the timeout expires, no additional processing will be performed on the executed operations. Usually no changes needed. |
| DataNode.deployment.host.group.enabled        | false        | DataNode service-specific field, whether to specify the Group ID of the DataNode service of the storage node. If it is set to `false`, the product will automatically group the storage nodes; If it is set to `true`, it will be grouped according to the specified IP addresses. |
| DataNode.deployment.host.group.x=\*\*\*\*\*\* |              | This configuration takes effect when `DataNode.deployment.host.group.enabled` is set to `true`, and is used to specify the Group ID of the storage node. It is necessary to configure 3 or more different Groups and corresponding node IP addresses. |

For the service name XXX in the above table, please refer to the following information for planning.

| **Service Name**      | **Distribution Planning** |
|-------------------|-------------------------------------------------------------------------------------|
| DIH               | All nodes in the cluster                                          |
| InfoCenter        | Usually 2-3 nodes in the cluster, it is recommended that the selected nodes should be in different groups |
| DriverContainer   | Configure as required, usually all nodes in the cluster |
| DataNode          | The node that provides storage resources, usually all nodes in the cluster |
| Console           | Usually one or more nodes. If multiple nodes are configured, it is recommended that the selected nodes should be in different groups |
| deployment_daemon | All nodes of the cluster |
| Coordinator       | Consistent with configuration in the DriverContainer |

### ZooKeeper Distribution Planning

This chapter explains the fields that need to be updated during the configuration process of the `zookeeper.properties`.

| **Attribute Name** | **Default Value** | **Description**                                                                                                                                                                                                                                    |
|-----------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| remote.user     | root         | The name of the user who logs in to the cluster node for Zookeeper component deployment and other related operations, usually the root user or other users with root privileges |
| remote.password |              | Deployment and other privileged user's password. Do not include special characters such as single quotes, double quotes, backquote, slashes, backslashes, etc. that may cause characters to escape. |
| server.X        |              | The IP address of the node where the ZooKeeper service is deployed, usually the same as the IP address of the Control Network, X starts from 1, the number must be odd, usually 3-7 nodes, and the number of nodes accounts for more than half of the total number of nodes in the clusters, no more than 7 is recommended. ZooKeeper nodes are evenly distributed in different groups, and the node information is consistent with the ZooKeeper-related configuration in the xml file |

##  xml Configuration

This chapter introduces the function and configuration of each parameter in the xml file under the `config` directory.

The format of the attribute value is as follows:

> `<property name="name_value" value="value_string"/>`

The content in the first double quotation mark is the attribute name, and the content in the second double quotation mark is the attribute value.

Usually, in a physical machine environment, there is no need to make many changes to the configuration in the xml file except for the parameters related to the IP address, the number of groups, and the number of disks.

If some configuration items are missing in the xml file, please add manually.

### General Configuration

Some configuration parameters are mandatory in every module, it is not necessary to set them separately in each module. In this case, set `<project
name="*">` means it will take effect globally for all modules. If a parameter is set separately in a module, the values set in the module override global values. If you want to override values in a module for certain parameters, usually `jvm.properties`, you may set it separately.

#### log4j.properties

| **Attribute Name**                 | **Default Value** | **Description** |
|------------------------------------|--------------|----------------------|
| log4j.rootLogger                   | WARN, FILE   | The log level recorded in the log file. By default, only logs higher than this level are printed. The valid levels are ERROR, FILE, DEBUG, FILE, INFO, FILE |
| log4j.appender.FILE.MaxFileSize    | 200MB        | The maximum size of each log file is adjusted according to the actual environment of the maximum space available for the storage node /var/testing directory. The valid units are GB, MB |
| log4j.appender.FILE.MaxBackupIndex | 15           | Set the number of log files. The name of the current log file is *.log, and the name of the historical log file is *.log.x. The maximum value of x is the value of this parameter |

#### network.properties

| **Attribute Name**                    | **Default Value** | **Description**       |
|---------------------------------|--------------|---------------------------------------------------------------------------|
| control.flow.subnet             |              | Control Network configuration. |
| monitor.flow.subnet             |              | Monitor Network configuration, usually consistent with the value of control.flow.subnet. |
| enable.data.depart.from.control | true         | Whether the Control, Monitoring, Data and Business Networks are separated. When using 2 or more network segments, set it to `true` |
| data.flow.subnet                |              | Storage Private Network configuration, used for data communication between storage nodes, usually 10 Gigabit network or IB network |
| outward.flow.subnet             |              | Business Network configuration, the network used by computing nodes and storage for I/O transmission |

#### storage.properties

| **Attribute Name**      | **Default Value** | **Description**                                                                                                                                                                                                                                                                                                                                                                                                         |
|-------------------|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| page.size.byte    | 4096         | page size, if the physical size of the disk used in the actual environment is 4096, the configuration should be 4096; if the physical size of the disk used is 512, and the configuration should be 8192. This configuration is related to `page.metadata.need.flush.to.disk configuration` |
| segment.size.byte | 1073741824   | Segment size in bytes, the default size is 1G, it is the smallest unit of volume capacity, the ideal value of the number of segments of a single node is recommended not to exceed about 5000, and the theoretical maximum limit is 15000. In order to make full use of the space on the disk, use the total disk capacity of a single node, divide it by the number of segments (usually 5000), and select the closest value to the nth power of 2, such as 2G, 4G, 8G etc. The total number of segment units in the system should not exceed 100,000. When configuring this parameter, it is also necessary to consider the possibility of storage expansion in the future   . |
| io.timeout.ms     | 60000        | Normally, there is no need to adjust the I/O timeout, but it can be adjusted appropriately for the virtual machine environment or when the network delay is relatively large. |

### JVM Configuration

The main function of `jvm.properties` is to limit (or configure) the memory size used by each service (each service is composed of one or more JAVA processes). `jvm.properties` needs to be configured in all modules except Console. The values that need to be set for each module are different, and have to be configured individually for each service according to the environment hardware specifications. For the DriverContainer module, an additional file `coordinator-jvm.properties` exists. The configuration parameters are similar to those in `jvm.properties`. For the DataNode module, additional memory resources need to be allocated when the service is running. The configuration is relatively complicated. For details, refer to DataNode module in detail.

| **Attribute Name**          | **Default Value** | **Description**           |
|-----------------------|--------------|------------------------|
| initial.mem.pool.size | xxm/xxg      | The initial memory allocated at runtime |
| max.mem.pool.size     | yym/yyg      | The maximum memory allocated at runtime |

### Component Configuration

Here we focus on the configurations in multiple modules, such as database and ZooKeeper configurations. These configurations may be used in InfoCenter, Coordinator and other modules, the configuration content shall be consistent.

#### ZooKeeper Configuration

In order to ensure the stability of the system, the InfoCenter service is usually deployed on two or more nodes, one of which is the Master node, and the rest are Slave nodes. When the Master node fails, a Slave node turns into a Master node, and the switch between the active and standby states is implemented by ZooKeeper.

| **Attribute Name**                | **Default Value**                        | **Description**                                                                                                         |
|-----------------------------|-------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| zookeeper.connection.string | ZK_IP1:2181, ZK_IP2:2181, ZK_IP3:2181 | ZooKeeper cluster configuration, ZK_IP is the IP configured in the `zookeeper.properties` file, the number of nodes must be an odd number, usually using the IP address of the Control Network. |
| zookeeper.election.switch   | true                                | Enable/disable ZooKeeper, must be enabled when deploying active and standby services. |

#### Database Configuration

Currently the database uses PostgreSQL (whether it is a single node or a cluster method). This configuration is for the InfoCenter module.

Configuration in PostgreSQL mode:

<table>
<colgroup>
<col style="width: 26%" />
<col style="width: 39%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><strong>Property Name</strong></td>
<td><strong>Default</strong></td>
<td><strong>Description</strong></td>
</tr>
<tr class="even">
<td>jdbc.driver.class</td>
<td>org.postgresql.Driver</td>
<td>Database driver</td>
</tr>
<tr class="odd">
<td>jdbc.url</td>
<td><p>InfoCenter module configuration:</p>
<p>jdbc: postgresql://IP_address:5432/controlandinfodb</p></td>
<td>PostgreSQL database IP address and name</td>
</tr>
<tr class="even">
<td>hibernate.dialect</td>
<td>py.db.sqlite.dialect.PostgresCustomDialect</td>
<td>PostgreSQL database dialect</td>
</tr>
<tr class="odd">
<td>package.hbm</td>
<td>hibernate-config</td>
<td>Configuration in PostgreSQL mode</td>
</tr>
</tbody>
</table>

### Coordinator Configuration

#### coordinator.properties

|                                          |              |                            |
|------------------------------------------|--------------|----------------------------|
| **Attribute Name**                       | **Default Value** | **Description**       |
| io.depth                                 | 128          | Queue depth, usually no need to modify |
| enable.logger.tracer                     | false        | Log tracing, used only for debugging, in production environment must be disabled |
| trace.all.logs                           | false        | Trace all logs, used only for debugging, in production environment must be disabled |
| debug.io.timeout.threshold.ms            | 1500         | I/O timeout threshold |
| network.checksum.algorithm               | DUMMY        | Network checksum algorithm, consistent with `page.checksum.algorithm` and `network.checksum.algorithm` values ​​in `datanode.properties`, valid values ​​are DUMMY, ALDER32, DIGEST, CRC32, CRC32C |
| ping.host.timeout.ms                     | 500          | Coordinator to DataNode network detection timeout, in a POC environment it is recommended to be 100, in a production environment it is recommended to be 500 |
| network.connection.detect.retry.maxtimes | 3            | Network detection retry times |
| network.healthy.check.time               | 3            | Network check frequency |

### DIH Configuration

#### instancehub.properties

|                     |                  |                                                                                            |
|---------------------|------------------|--------------------------------------------------------------------------------------------|
| **Attribute Name**        | **Default Value**     | **Description**                                                                               |
| center.dih.endpoint | IP_Address:10000 | Center DIH node configuration, IP_Address is consistent with the IP configured in `DIH.center.host.list` in the `deploy.properties` file |

### DataNode Configuration

#### datanode.properties

|                                          |              |                                                                                                                                                                                |
|------------------------------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Attribute Name** | **Default Value** | **Description** |
| threshold.to.request.for.new.member.ms | 1800000 | The waiting time for a new Secondary to join a segment, in milliseconds |
| wait.time.ms.to.move.segment.to.deleted | 300000 | The duration of time that the volume can be recovered after being deleted. After this time, the volume cannot be recovered. The unit is in milliseconds |
| memory.size.for.data.logs.mb.per.archive | 300 | The fastbuffer space size required by each disk, the total fastbuffer space size on the node is (the number of disks + 1) multiplied by the value of this parameter, usually no need modify, the unit is in MB |
| page.system.memory.cache.size | 2G | The memory space required by the system page, 0 means to adapt according to the actual physical memory size, it is not recommended to configure it as 0, if there is not enough memory, it can be reduced according to the actual environment |
| page.metadata.need.flush.to.disk | false | Flushing metadata pages to disk, for 0.5k sector set to `true`, for 4k sector set to `false`, this configuration is related to the `page.size.byte` |
| delay.record.storage.exception.ms | 2000 | |
| page.checksum.algorithm | DUMMY | Paging verification algorithm, consistent with the value of `network.checksum.algorithm` in `coordinator.properties` and `network.checksum.algorithm` in `datanode.properties`, valid values ​​are DUMMY, ALDER32, DIGEST, CRC32 , CRC32C |
| network.checksum.algorithm | DUMMY | Network checksum algorithm, consistent with the value of `page.checksum.algorithm` in `network.checksum.algorithm` in `coordinator.properties` and `page.checksum.algorithm` in `datanode.properties`, valid values ​​are DUMMY, ALDER32, DIGEST, CRC32 , CRC32C |
| archive.init.mode | overwrite | DataNode initialization mode, optional values ​​are `append`, `overwrite`, usually no need to change |
| max.io.pending.requests | 5000 | Pending queue, usually no need to modify |
| max.io.depth.per.hdd.storage | 64 | The maximum iodepth of the disk, usually no need to modify


#### jvm.properties

The JVM configuration of DataNode is explained separately. The following are suggested values for the production environment, which need to be slightly adjusted.

|                       |              |                                                  |
|-----------------------|--------------|--------------------------------------------------|
| **Attribute Name** | **Default Value** | **Description** |
| initial.mem.pool.size | 40G | The initial memory allocated at runtime |
| max.mem.pool.size | 40G | The maximum memory allocated during runtime |
| parallel.gc.threads | 15 | The number of parallel threads, it is configured as about 5/8 of the number of CPU threads on a single node |
| conc.gc.threads | 4 | This parameter is about 1/4 of the value of `parallel.gc.threads` |

### DriverContainer Configuration

#### coordinator-jvm.properties

This attribute is used to configure the memory settings of the Coordinator, corresponding to the driver listed on Web Console, and each driver needs to consume a certain amount of memory resources on the corresponding container.

|                          |              |                          |
|--------------------------|--------------|--------------------------|
| **Attribute Name** | **Default Value** | **Description** |
| initial.mem.pool.size | 1024m | The initial memory allocated for each driver |
| min.mem.pool.size | 1024m | Minimum memory size for each driver |
| max.mem.pool.size | 1024m | Maximum memory size for each driver |
| max.direct.memory.size   | 512m         |                          |
| netty.allocator.maxOrder | 7            |                          |

#### drivercontainer.properties

|                              |              |                                                                                                                                                                                                                                                                                                                       |
|------------------------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Attribute Name**           | **Default Value** | **Description**                                                                                                                                                                                                                                                                                                          |
| system.memory.force.reserved | 512M         | The system-reserved memory, driver mounting and unmounting operations are performed by DriverContainer. Before mounting a driver, DriverContainer will calculate whether the remaining system memory is still greater than this value after the driver is mounted. If the remaining system content is less than the value set, it will prompt that the memory resources are insufficient, and the mounting operation will not be performed, so as to prevent insufficient system resources from affecting other services after the mounting is completed. In the production environment, this value can be adjusted appropriately, such as 2048M |
| iet.target.flag              | false        | iSCSI mode, this field is set to `true` for IET mode, `false` for cli mode |
| iscsi.portal.type            | IPV6         | Coordinator listening address type, if this field is set to `IPV4`, it only monitors IPv4 addresses, and `IPV6` means it monitors both IPv4 and IPv6 addresses |



#### lioCommandManager.properties

|                              |                             |                                                        |
|------------------------------|-----------------------------|--------------------------------------------------------|
| **Attribute Name**                 | **Default Value**                | **Description**                                           |
| default.saveConfig.file.path | /etc/target/saveconfig.json | The path of this file is not the same for RPM and DEB environments, please pay attention. |

#### liotarget.properties

|                 |                                                        |                                                                     |
|-----------------|--------------------------------------------------------|---------------------------------------------------------------------|
| **Attribute Name**    | **Default Value**                                           | **Description**                                                        |
| saveconfig.path | /etc/target/saveconfig.json                            | saveconfig path, the path is different under CentOS and Ubuntu |
| restore.command | /usr/bin/targetcli restore /etc/target/saveconfig.json | restore executable path, the path is different under the CentOS and Ubuntu |

### InfoCenter Configuration

#### infocenter.properties

|                                          |                   |                                                                                                                                              |
|------------------------------------------|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| **Attribute Name**                       | **Default Value** | **Description**                                                                                                                              |
| is.arbiter.group.set                     | false             | Whether to use a special group for the arbiter function, when this field is set to `true`, an arbiter group number should be specified in `arbiter.group.id` |
| arbiter.group.id                         | 0                 | Arbiter group number, effective when `is.arbiter.group.set` is `true` |
| page.wrapp.count                         | 128               | Stripe width, when reading and writing sequentially, the data size of each segment is `page.wrapp.count * segment.size`, then the system switches to the next segment. |
| segment.wrapp.count                      | 8                 | The number of data disks on a single node usually needs to be adjusted according to the actual number. If there are multiple nodes in a group in the environment, this value needs to be modified to the number of data disks in each group. |
| group.count                              | 4                 | The number of storage node groups, usually between 3 and 5. For 2-copy or 3-copy volumes, each segment should be distributed in 3 different groups. This configuration cannot be modified after deployment. |
| jdbc.url                                 |                   | Please refer to the database configuration description. |
| max.rebalance.task.count.volume.datanode | 50                | The number of rebalance concurrency is calculated according to the number of segment units. It is recommended to set this value to 50 in a production environment. |

#### jvm.properties

This is a separate description for the JVM configuration of InfoCenter, which is related to the segment unit.

|                       |                   |                                      |
|-----------------------|-------------------|--------------------------------------|
| **Attribute Name**    | **Default Value** | **Description**                      |
| initial.mem.pool.size | 1024m        | Initial memory allocation size at runtime |
| max.mem.pool.size     | 2048m        | The maximum memory allocatable during runtime. This value is related to the number of segment units. If the number of segment units is less than 20,000, it is recommended to configure this value to 2G. If the number of segment units exceeds 20,000, increase 1G on every 10,000 units, and the total number of segment units should not exceed 100,000 |

## Updating Configuration

Parameters may be removed as the requirement changes. Due to the limitations, this document only lists certain parameters that may need to be modified (set explicitly), other parameters are not specified in the file (set implicitly).

In the case of parameter removal, redundant parameters will not affect the functionality of the product.

In the case of new parameter addition, you need to add the corresponding parameter to the xml file and set it explicitly.

Parameters are explicitly set as follows:

- Locate in the xml file which need to add or modify parameters
- Set the attribute name and corresponding value in the xml file

For your convenience, it is recommended that when adding attributes, add them in alphabetical order in the xml file. After setting, make sure to execute `perl bin/update_system_config.pl` to update the configuration in the local software package before deploying or upgrading.