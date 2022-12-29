##  System Requirements

### Hardware Requirements

Deployment of Distributed Block Storage (hereinafter "DBS") requires at least 3 nodes. For different usage scenarios and different customer environments, the configuration parameters may vary, please refer to [Advanced Configuration](docs/configuration.md). A functional demo can be deployed using virtual machines, while performance testing should be deployed on high-performance physical servers.
> For the convenience of explanation, the following documents use 3 nodes ( `192.168.1.10`, `192.168.1.11`, `192.168.1.12`) to illustrate the deployment use case.

### Software Requirements

For a list of supported operating system and installation requirements, please refer to [OS Installation](docs/operatingsystem.md) or contact the corresponding engineer for further information.

## Configuration and Deployment

### I. Preparing for Installation
The installation package of DBS usually consists of the following two files:
`Installation-3.0.0.tar.gz`  
`pengyun-deploy-1.0.0-SNAPSHOT-OS-[2022-08-26_15-35-34].tar.gz`

1. Put the above packages into the `/opt/deploy/` directory of deployment node (e.g. the first node of the cluster)
```bash
cd /opt/deploy
# list files in /opt/deploy directory
ls
Installation-3.0.0.tar.gz  pengyun-deploy-1.0.0-SNAPSHOT-OS-[2022-08-26_15-35-34].tar.gz
```

2. Unzip the `Installation` toolkit
```bash
tar -zxf Installation-3.0.0.tar.gz && rm Installation-3.0.0.tar.gz
# list files in /opt/deploy directory
ls
Installation  pengyun-deploy-1.0.0-SNAPSHOT-OS-[2022-08-26_15-35-34].tar.gz
```

### II. Configuration Wizard

Execute the following commands and follow the prompts to customize the relevant configuration.
```bash
cd /opt/deploy/Installation
./install.sh
```
1. Configure Cluster Credentials
```
=== (1/18): Configure Cluster Credentials ===
Username for cluster nodes: root
Password for cluster nodes: 111111
Is this configuration correct and complete?
Press Enter to continue, [M] to modify:
```

2. Configure Cluster Addresses
```
=== (2/18): Configure Cluster Addresses ===
IP addresses for cluster nodes:
192.168.1.10:192.168.1.11,192.168.1.12
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
Use commas to separate multiple IP addresses, and use colon to indicate consecutive IP addresses.  
`192.168.1.10:192.168.1.12` refers to 3 servers `192.168.1.10`, `192.168.1.11` and `192.168.1.12`.  
`192.168.1.11, 192.168.1.12` refers to 2 servers `192.168.1.11` and `192.168.1.12`.

3. Configure Cluster Hostname
```
=== (3/18): Configure Cluster Hostname ===
Hostname prefix for cluster nodes: server
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```

5. Configure NTP Mode 
```
=== (5/18): Configure NTP Mode ===
NTP mode for cluster nodes: ntpdate
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
The default time synchronization service is `ntpdate`, for CentOS 8 or Redhat 8, please choose `chrony`.

6. Configure NTP Server 
```
=== (6/18): Configure NTP Server ===
NTP server addresses for cluster nodes:
192.168.1.10
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
To ensure time synchronization between nodes in the cluster, specify a node in the cluster to automatically install the NTP service, or an existing NTP service address.

7. Configure Database Mode  
```
=== (7/18): Configure Database Mode ===
Database mode for cluster nodes: single
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
`single` mode requires 1 node, `pacemaker` mode requires 2 nodes, `etcd` mode requires 3 nodes.

8. Configure Database Master  
Only applicable when `etcd` or `pacemaker` is selected as database cluster mode, otherwise this step will be skipped automatically.

9. Configure Database Slave  
Only applicable when `pacemaker` is selected as database cluster mode, otherwise this step will be skipped automatically.

10. Configure Database Address  
```
=== (10/18): Configure Database Address ===
Specify IP address ranges for database cluster nodes:
192.168.1.10
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```

11. Configure HA Service
```
=== (11/18): Configure HA Service ===
Specify IP address ranges for HA services：
192.168.1.11,192.168.1.12
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
The number of nodes used for high availability service should ≥ 2

12. Configure Storage Service  
```
=== (12/18): Configure Storage Service ===
Specify IP address ranges for storage service:
192.168.1.10:192.168.1.12
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
Usually configured as all nodes of the cluster, **in addition to the hard disk for installing operating system, each node should have additional blank disk(s) for Storage Service**.

13. Configure Web Interface
```
=== (13/18): Configure Web Interface ===
IP address of the web interface: 192.168.1.10
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
When the deployment is complete, management interface can be accessed through a web browser via the specified IP address.

14. Enable Control-Data Separation?
```
=== (14/18): Enable Control-Data Separation? ===
Current configuration: false
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
If `false` is selected, step 16 and 17 will be automatically skipped.

>**Control-Data Separation**  
In general, it is recommended that DBS uses separate networks for control flow and data flow. Control flow does not require high bandwidth, but high stability; data flow requires high bandwidth. It is recommended to use optical fiber or InfiniBand network to obtain better transmission performance. It is recommended to configure redundant links for data flow and control flow in a production environment.

15. Subnet for Control Communications
```
=== (15/18): Subnet for Control Communications ===
Subnet for control communications (e.g. 192.168.1.0/24).
Current configuration: 192.168.1.0/24
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```

16. Subnet for Data Communications  
Subnet for transferring internal data between storage nodes.

17. Subnet for User Communications  
Subnet for I/O communication between computing nodes and storage nodes.

18. Are you deploying on a physical server?
```
=== (18/18): Are you deploying on a physical server? ===
Current configuration: false
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```

Select `false` if deploying in a virtual machine such as VMware.

So far, the Configuration Wizard has been completed.

### III. Deployment
1. To deploy the DBS software, execute the following commands:
```
cd /opt/deploy/Installation
./install.sh -b
```
2. Deployment requires no user interaction and consists of the following 10 steps:

```
=== (1/10): Setup Deployment Server ===
Install packages on the current host for setting up deployment server.

=== (2/10): Check Deployment Server ===
Check whether current host meets the requirement to provide deployment service.

=== (3/10): Check Cluster Nodes ===
Check whether cluster hosts meet the requirement to install packages.

=== (4/10): Setup Cluster Nodes ===
Install packages on cluster hosts.

=== (5/10): Install Time Synchronization Service ===
Install ntpdate or chrony service on cluster hosts.

=== (6/10): Time Synchronization ===
Synchronize time between cluster hosts.

=== (7/10): Check Cluster Status ===
Check status of cluster hosts.

=== (8/10): Install Database Service ===
Install required database service (PostgreSQL) for the product.

=== (9/10): Update System Configuration ===
Update configuration settings for the product.

=== (10/10): Deploy Block Storage ===
Deploy dbs package under the parent folder of Installation folder, please ensure there is ONLY one tar.gz file.
```

If there is an error, the script will report and exit, please review the warning messages (**especially text in red color**).

>Due to the number of cluster nodes, hardware configuration, disk configuration, etc., the deployment time varies greatly. Please wait patiently and do not interrupt the deployment operation.

3. When the deployment is complete, the DBS web interface can be accessed through a browser, and the address is `http://192.168.1.10:8080` where `192.168.1.10` is the IP address of the “Web interface” configured in step 13 of the Configuration Wizard. For the usage of DBS, please refer to the [User Manual](docs/manual-zh.md).

### IV. Undeploying
To cleanup the deployed DBS software, execute the following command.
```bash
cd /opt/deploy/Installation
perl product_block_storage_operation.pl -o wipeout
```
