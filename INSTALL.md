##  System Requirements

### Hardware Requirements

- Deployment of Distributed Block Storage (hereinafter "DBS") requires at least **3 nodes**. For different usage scenarios and different customer environments, the configuration parameters may vary, please refer to [Advanced Configuration](docs/configuration.md). A functional demo can be deployed using virtual machines, while performance testing should be deployed on high-performance physical servers.
- With default configuration, each node requires at least **128 GB of RAM**; for a Proof of Concept deployment, each node requires **8 GB of RAM**.
- In addition to the operating system disk, each storage node requires at least **one additional 1TB blank hard disk**.

> For the convenience of explanation, the following documents use 3 nodes ( `192.168.1.10`, `192.168.1.11`, `192.168.1.12`) to illustrate the deployment use case.

### Software Requirements

For a list of supported operating system and installation requirements, please refer to [OS Installation](docs/operatingsystem.md) or contact the corresponding engineer for further information.

## Configuration and Deployment

>**Note**  
The following instructions assume that you already have sufficient privileges, we will not go into details about using `su` or `sudo` and other privilege escalation operations.

### I. Preparing for Installation
The packages provided by ZettaStor DBS [Downloads](https://zdbs.io/en/download/) usually consist of the following two files:  
`Installation-1.0.0-OS-*.tar.gz`  
`pengyun-deploy-1.0.0-OS-*.tar.gz`

1. Put `Installation-1.0.0-OS-*.tar.gz` toolkit into the `~` directory of the deployment node (e.g. the first node of the cluster) and unzip
```bash
cd ~
ls
Installation-1.0.0-OS-20230103.tar.gz

mkdir -p /opt/deploy && tar -zxf Installation-1.0.0-OS-20230103.tar.gz -C /opt/deploy
# list files in /opt/deploy directory
ls /opt/deploy/
Installation
```

2. Put the `pengyun-deploy-1.0.0-OS-*.tar.gz` into the `/opt/deploy/` directory of the deployment node (e.g. the first node of the cluster)
```bash
# list files in /opt/deploy directory
ls /opt/deploy/
Installation  pengyun-deploy-1.0.0-OS-20230101.tar.gz
```

### II. Configuration Wizard

Execute the following commands and follow the prompts to customize the relevant configuration.
```bash
cd /opt/deploy/Installation
./install.sh
```
1. Configure Cluster Credentials
```
=== (1/17): Configure Cluster Credentials ===
Username for cluster nodes: root
Password for cluster nodes: 111111
Is this configuration correct and complete?
Press Enter to continue, [M] to modify:
```

2. Configure Cluster Addresses
```
=== (2/17): Configure Cluster Addresses ===
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
=== (3/17): Configure Cluster Hostname ===
Hostname prefix for cluster nodes: server
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```

4. Configure NTP Mode 
```
=== (4/17): Configure NTP Mode ===
NTP mode for cluster nodes: ntpdate
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
The default time synchronization service is `ntpdate`, for CentOS 8 or Redhat 8, please choose `chrony`.

5. Configure NTP Server 
```
=== (5/17): Configure NTP Server ===
NTP server addresses for cluster nodes:
192.168.1.10
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
To ensure time synchronization between nodes in the cluster, specify a node in the cluster to automatically install the NTP service, or an existing NTP service address.

6. Configure Database Mode  
```
=== (6/17): Configure Database Mode ===
Database mode for cluster nodes: single
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
`single` mode requires 1 node, `pacemaker` mode requires 2 nodes, `etcd` mode requires 3 nodes.

7. Configure Database Master  
Only applicable when `etcd` or `pacemaker` is selected as database cluster mode, otherwise this step will be skipped automatically.

8. Configure Database Slave  
Only applicable when `pacemaker` is selected as database cluster mode, otherwise this step will be skipped automatically.

9. Configure Database Address  
```
=== (9/17): Configure Database Address ===
Specify IP address ranges for database cluster nodes:
192.168.1.10
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```

10. Configure HA Service
```
=== (10/17): Configure HA Service ===
Specify IP address ranges for HA services：
192.168.1.11,192.168.1.12
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
The number of nodes used for high availability service should ≥ 2

11. Configure Storage Service  
```
=== (11/17): Configure Storage Service ===
Specify IP address ranges for storage service:
192.168.1.10:192.168.1.12
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
Usually configured as all nodes of the cluster, **in addition to the hard disk for installing operating system, each node should have additional blank disk(s) for Storage Service**.

12. Configure Web Interface
```
=== (12/17): Configure Web Interface ===
IP address of the web interface: 192.168.1.10
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
When the deployment is complete, management interface can be accessed through a web browser via the specified IP address.

13. Enable Control-Data Separation?
```
=== (13/17): Enable Control-Data Separation? ===
Current configuration: false
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```
If `false` is selected, step 16 and 17 will be automatically skipped.

>**Control-Data Separation**  
In general, it is recommended that DBS uses separate networks for control flow and data flow. Control flow does not require high bandwidth, but high stability; data flow requires high bandwidth. It is recommended to use optical fiber or InfiniBand network to obtain better transmission performance. It is recommended to configure redundant links for data flow and control flow in a production environment.

14. Subnet for Control Communications
```
=== (14/17): Subnet for Control Communications ===
Subnet for control communications (e.g. 192.168.1.0/24).
Current configuration: 192.168.1.0/24
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```

15. Subnet for Data Communications  
Subnet for transferring internal data between storage nodes.

16. Subnet for User Communications  
Subnet for I/O communication between computing nodes and storage nodes.

17. Are you deploying on a physical server?
```
=== (17/17): Are you deploying on a physical server? ===
Each physical node requires at least 128 GB of RAM. Please choose "false" for a Proof of Concept deployment, in which case each node requires 8 GB of RAM.
Current configuration: false
Is this configuration correct and complete?
Press Enter to continue, [M] to modify, [P] to return to the previous step:
```

Select `false` if deploying in a virtual machine such as VMware.

So far, the Configuration Wizard has been completed, you can press `Ctrl+C` to quit the Wizard, and refer to the next section for an automated deployment. If you continue with the Wizard, you will enter an interactive deployment process, which involves the same 10 steps as described in the next section.

### III. Deployment
1. To deploy the DBS software, execute the following commands:
```
cd /opt/deploy/Installation
./install.sh --batch
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
