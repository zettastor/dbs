This document mainly lists the operating system related information for ZettaStor Distributed Block Storage product, and explains the OS installation requirements.
According to the steps specified in the document, after the OS is installed and configured, the IP address can be changed, and the root user may login via ssh.

### Scope of This Document

This document covers OS installation and other parts that require manual configuration.

- Configuration guide for some models of RAID cards
- Selection of language, time zone, disk partition, etc. during OS installation. (For some operating systems, PXE installation is supported, and the configuration related to PXE installation also needs to follow the settings in this document)
- The driver installation method required for normal operation of the basic environment (such as the installation of network card drivers) after OS installation.
- Installation of the computing node software (the installation of the computing node OS is not within the scope of this document)

### Release Notes 

2022-09-19
updated documents according to the list of supported OS's of open source product

2022-09-30
release after review

## OS Installation

### General Principles

Regardless of the type of OS, the following rules need to be followed when installing (special operating systems are documented separately):

- When setting the installation language, please select `English(United States)`
- When setting the date and time, please select `Asia / Shanghai`
- When setting the installation disk and partition, please select the disk where you want to install OS, and when setting the partition, keep the `/boot` partition and the `/` root directory partition, other partitions such as `swap` and `/home` are not recommended to be preserved. That is to say, except the mandatory `/boot` partition, all the remaining space of the system disk is allocated to the root partition, and the root partition file system format is recommended to be `ext4`.
- Set the IP addresses for the nodes according to the network planning
- Set other options according to the OS recommendation
- Some OS's require the hostname to be set manually. It is recommended that the hostnames consist only lowercase letters and numbers. The reason is that we usually use the hostname of the node as part of the replication slot name, postgresql cluster's replication slot name can only consist of lowercase letters, numbers, and underscores.

Most of the OS's that ZettaStor DBS can run on are listed in this chapter. 

Notice:
- For the OS's listed in this chapter, the names are sorted alphabetically.
- The name of each OS must be consistent with the name planned in the `Setup Wizard`.

### CentOS 7 (1908)

#### Image

File name: `CentOS-7-x86_64-DVD-1908.iso`

MD5: `dc5932260da8f26bcdce0c7ebf0f59ca`

#### Installation

Use USB drive or CD-ROM to start the OS installation interface.

Select language as English

<img src="https://zdbs.io/operatingsystem/media/image2.png" />

Set the time zone

<img src="https://zdbs.io/operatingsystem/media/image3.png" />

Software selection

<img src="https://zdbs.io/operatingsystem/media/image4.png" />

Select the `Compute Node` mode in the software list. The Add-ons on the right are not required.

<img src="https://zdbs.io/operatingsystem/media/image5.png" />

Start the system partition configuration

<img src="https://zdbs.io/operatingsystem/media/image6.png" />

Select the disk on which the OS is installed, and then select `I will configure partitioning`.

<img src="https://zdbs.io/operatingsystem/media/image7.png" />

Change the partition mode to `Standard Partition`

<img src="https://zdbs.io/operatingsystem/media/image8.png" />

After selecting the `Standard Partition` mode, click `create them automatically` to adjust the partition.

<img src="https://zdbs.io/operatingsystem/media/image9.png" />

After the adjustment, only `/boot` and `/` partitions remain. All remaining space of the system disk is allocated to the root partition, and the file system format of the root partition is recommended to be `ext4` format

<img src="https://zdbs.io/operatingsystem/media/image10.png" />

(Here is a method to set the size of the root partition: After deleting the unnecessary partitions, select the root partition, set the size of the root partition to a capacity larger than the entire system disk, and then adjust the file system format of the root partition to `ext4`. The system will automatically allocate as much free space as possible to the root partition)

`Accept Changes` of the partition configuration and write to the disk

<img src="https://zdbs.io/operatingsystem/media/image11.png" />

The IP address of the node can be configured here.

<img src="https://zdbs.io/operatingsystem/media/image12.png" />

Configure the password for the root user during installation. No other users need to be created.

<img src="https://zdbs.io/operatingsystem/media/image13.png" />

After the system is installed, restart it and check whether you can enter the system correctly.

#### Version and Kernel Information

After the system is installed:

version information
```bash
cat /etc/*release | grep PRETTY_NAME):

PRETTY_NAME="CentOS Linux 7 (Core)"
```

kernel information
```
uname -a

Linux localhost.localdomain 3.10.0-1062.el7.x86_64 \#1 SMP Wed Aug 7
18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

## Driver Installation

### InfiniBand

Use the command `lspci` to identify the model of the IB card, you can use `yum install
pciutils` on CentOS to install the package.

```bash
lspci | grep Mellanox
```

Download the corresponding driver according to the operating system type

http://cn.mellanox.com/page/products_dyn?product_family=26&mtag=linux_sw_drivers

```bash
tar zxf MLNX_OFED_LINUX-*.tar.gz
cd MLNX_OFED_LINUX-*
./mlnxofedinstall
```
The installer will check the system library, if a dependency is missing, a prompt to use `yum install` will appear.

```
/etc/init.d/openibd restart
```

After the installation is complete, restart the host and use `ip a` to see the IB network cards with names such as ib0, ib1...

On CentOS, you need to start openibd and opensmd services and set them to start at boot. The operation method is as follows:
```
service openibd start
chkconfig openibd on
```

Start the subnet manager opensmd and set it to start on boot:  
```
service opensmd start  
chkconfig opensmd on
```

Please refer to [Setup Guide](https://www.cloudibee.com/network-bonding-modes/) for network binding under Linux.

`ifconfig` is no longer maintained, please use the `ip addr show` command to display the IP address of the HCA.

## Frequently Asked Questions

### Installation from a USB drive

Take CentOS 7 as an example.

Use the ISO image to make a USB boot drive (the production method is relatively simple, you can search for relevant information on the Internet). Then change the boot order of the server to boot from USB first. After the server boots, the following interface should appear:

<img src="https://zdbs.io/operatingsystem/media/image33.png" />

When entering the installation interface, Press the `Tab` key and modify the command to the following:

`vmlinuz initrd=initrd.img linux dd quiet`

The command in this step is to confirm the drive letter of the USB drive. For example, the drive letter obtained from the USB drive is `sdc4`. Then start the server again through the USB drive. Press the `Tab` key and modify the command to the following:

`vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdc4 quiet`

If there are no other errors, you should enter the language selection interface. The OS installation operation begins.

### Disable VT-D and Console Redirection on a RH2288 server
In some cases, the RAID card cannot read the hard disk information correctly due to insufficient resources of the RAID card. In this case, it is necessary to disable unnecessary services on the RAID card. The following describes the configuration on a Huawei RH2288 server.

Enter the BIOS menu. Select `Socket Configuration` under `Advanced` on the left

<img src="https://zdbs.io/operatingsystem/media/image34.png" />

Select `IIO Configuration` in `Socket Configuration`

<img src="https://zdbs.io/operatingsystem/media/image35.png" />


Change `Enabled` to `Disabled` in `IIO Configuration` - `Intel(R) VT for Directed I/O(VT-d)`

<img src="https://zdbs.io/operatingsystem/media/image36.png" />

Change `Enabled` to `Disabled` in `Advanced` - `Console Redirection`

<img src="https://zdbs.io/operatingsystem/media/image37.png" />

### "NMI watchdog: soft lockup CPU stuck" during boot

This problem occurs when the OS is installed from a USB drive, and the USB drive cannot be found, so that the installation program cannot boot normally.

What you can try is to disable the Intel Graphics feature by setting the `nomodeset` option before booting as follows (take CentOS as an example):

Insert the USB drive, boot from the USB drive, when the startup item is loaded, press the `Tab` key (some systems may require to press the `e` key) to modify the startup command. Add `nomodeset` field in the `quiet` line, .

For example, to determine the USB drive letter, it can be modified as follows:

`vmlinuz initrd=initrd.img linux dd nomodest quiet`

and **after determining the drive letter**

`vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdc4 nomodest quiet`

The rest of the operations can proceed normally.
