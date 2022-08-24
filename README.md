# ZettaStor DBS

[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/1486/badge)](https://bestpractices.coreinfrastructure.org/projects/1486)
[![LICENSE](https://img.shields.io/badge/licence-AGPL--3-blue.png)](https://github.com/lanewu/dbs/blob/master/LICENSE)

English | [简体中文](README-zh.md)

# What is ZettaStor DBS？

ZettaStor DBS provides enterprise-level business storage solutions with high availability, high performance, easy expansion and easy maintenance for large-scale virtualization, private cloud and container environments. It serves as a solid digital base for core applications.

- [x] IT architecture upgrade: software-defined technology, storage resource pooling, multi-scenario applications support, silo-free environment
- [x] Cost saving: reducing TCO by more than 60% compared to traditional storage, based on standard servers and network hardwares
- [x] On-demand customization and flexible expansion: horizontal and linear expansion, on-demand performance and capacity, lower costs
- [x] Safe, efficient and seamless migration: the original system remains untouched, with simple and visualized migration process

| Features | Community Edition  | Enterprise Edition | 
| ------------- | ------------- |  ------------- | 
| Decentralized architecture | ![CI (Linux)](https://img.shields.io/badge/-supported-brightgreen)| ![CI (Linux)](https://img.shields.io/badge/-supported-brightgreen) |
| Sub-millisecond latency | ![CI (macOS)](https://img.shields.io/badge/-supported-brightgreen) | ![CI (macOS)](https://img.shields.io/badge/-supported-brightgreen) |
| Large-scale node deployment | ![CI (Windows)](https://img.shields.io/badge/-supported-brightgreen) | ![CI (Windows)](https://img.shields.io/badge/-supported-brightgreen) |
| Real-time customer support | ![Build documentation](https://img.shields.io/badge/-unsupported-red)| ![Build documentation](https://img.shields.io/badge/-supported-brightgreen) |
| Cluster size | ![minimum](https://img.shields.io/badge/minimum-%E2%89%A53-blue) ![maximum](https://img.shields.io/badge/maximum-%E2%89%A510000-blue) | ![minimum](https://img.shields.io/badge/minimum-%E2%89%A53-blue) ![maximum](https://img.shields.io/badge/maximum-%E2%89%A510000-blue) |
| Data redundancy mode | ![redundancy](https://img.shields.io/badge/-2%20copies-blue) | ![redundancy](https://img.shields.io/badge/-2%20copies-blue) ![redundancy](https://img.shields.io/badge/-3%20copies-blue) |
| Data access protocol | ![SCSI](https://img.shields.io/badge/-scsi-blue)  ![ISCSI](https://img.shields.io/badge/-iscsi-blue) | ![SCSI](https://img.shields.io/badge/-scsi-blue)  ![ISCSI](https://img.shields.io/badge/-iscsi-blue) ![CIFS](https://img.shields.io/badge/-cifs-blue)  ![NFS](https://img.shields.io/badge/-nfs-blue)  |

# Quick Start

If you're on a UNIX-like system (including Linux), the packages required for compilation can be installed by the folowing commands

## CentOS 7 / RHEL 7
```bash
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install net-tools maven thrift protobuf-compiler
```

## CentOS 8 / RHEL 8
```bash
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install net-tools maven compat-openssl10 protobuf-compiler
yum install https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/t/thrift-0.9.1-15.el7.x86_64.rpm
```

## Debian 10 / Debian 11 / Ubuntu 18 / Ubuntu 20
```bash
sudo apt-get install net-tools curl maven protobuf-compiler
curl -LO http://ftp.debian.org/debian/pool/main/t/thrift-compiler/thrift-compiler_0.9.1-2.1+b1_amd64.deb
sudo dpkg -i thrift-compiler_0.9.1-2.1+b1_amd64.deb
```

## openSUSE 15
```bash
zypper install net-tools-deprecated curl unzip maven thrift
curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v3.5.1/protoc-3.5.1-linux-x86_64.zip
unzip protoc-3.5.1-linux-x86_64.zip -d /usr/local
```

## Build from source
To build the package, Use the following commands in the directory where pom.xml is located
```bash
mvn versions:use-dep-version -DdepVersion=$(thrift --version | awk '{print $3}') -Dincludes=org.apache.thrift:libthrift
mvn versions:use-dep-version -DdepVersion=$(protoc --version | awk '{print $2}') -Dincludes=com.google.protobuf:protobuf-java
mvn clean install
```

# Where can I find out more?
Documentation

# Contributing

## Coding standard
Source code should be viewed and edited with your editor set to use two spaces per tab, with one tab used per indentation level. Spaces are used for other alignment within a line.

Most parts of the code follow [Google Java Style](https://google.github.io/styleguide/javaguide.html); some parts of the code follow [Oracle's Code Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html) -- mostly depending on who wrote the original version. **Above all else, be consistent with what you modify, and keep whitespace changes to a minimum when modifying existing source.** For new code, use Google Java Style.

## Submit your code
After finishing the development of your code, you should submit a pull request to master branch and fill out a pull request template. The pull request will trigger the CI automatically, and the code will only be merged after passing the CI and being reviewed. The Jenkins username and password of CI is netease/netease. If the CI fails to run, you can login to the Jenkins platform to view the reason for the failure.While the prerequisites above must be satisfied prior to having your pull request reviewed, the reviewer may ask you to complete additional design work, tests, or other changes before your pull request can be ultimately accepted.

For more details, please refer to [CONTRIBUTING](CONTRIBUTING.md).

# License
[AGPL 3.0](LICENSE)
