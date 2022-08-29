# ZettaStor DBS

[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/1486/badge)](https://bestpractices.coreinfrastructure.org/projects/1486)
[![LICENSE](https://img.shields.io/badge/licence-AGPL--3-blue.png)](https://github.com/lanewu/dbs/blob/master/LICENSE)

[English](README.md) | 简体中文

# 项目简介

云计算是产业发展的潮流和趋势，对于传统企业来说，云环境部署更加复杂。块存储作为云上不可或缺的一部分，其可靠性和稳定性至关重要。ZettaStor DBS可为大规模虚拟化、私有云和容器环境，提供高可用、高性能、易扩展、易维护的企业级用户业务存储解决方案，成为核心应用坚实的数据底座。

2015年8月，ZettaStor DBS分布式块存储系统在国内率先发布，填补了该领域的国内空白，并实现了多项关键技术首创。由于ZettaStor DBS从底层到整个架构都由鹏云网络自主研发，使其具备不断优化、创新的持久生命力，能更好适应IT技术日新月异的发展。

- [x] 去中心化架构：全对称分布式架构设计，基于类区块链的去中心化设计，消除了中心节点对系统规模、IO性能、稳定性和可靠性等方面的种种限制和不利因素。
- [x] 大规模节点部署：支持上万节点规模部署，且始终保持系统的稳定性及高性能。容量和存储性能随存储节点增加线性增长，从而获得海量规模的支撑能力。
- [x] 亚毫秒级延迟：在处理数据读写时直接操作硬盘本身，无需文件层转换；釆用优化的网络数据路径，数据直接并发写到存储节点，无中间代理转发，从而最大化缩短IO处理路径，在磁盘介质做主存的配置情况下，可实现毫秒以内的IO延迟。
- [x] 无感知故障恢复：在硬盘或节点故障时，能够立即由健康硬盘或节点接管任务，无需任何等待，也不会发生IO性能急剧下降或中断情况。故障恢复时间小于1秒，业务无感知。
- [x] 自主可控，安全可靠：拥有完整知识产权，自主研发，兼容主流国产化硬件和操作系统，提供全国产化存储解决方案。

| 功能特性 | 社区版  | 商业版 | 
| ------------- | ------------- |  ------------- | 
| 域管理 | ![supported](https://img.shields.io/badge/-支持-brightgreen) | ![supported](https://img.shields.io/badge/-支持-brightgreen) |
| 存储池 | ![supported](https://img.shields.io/badge/-支持-brightgreen) | ![supported](https://img.shields.io/badge/-支持-brightgreen) |
| 存储卷 | ![create](https://img.shields.io/badge/-创建-blue) ![delete](https://img.shields.io/badge/-删除-blue) ![resize](https://img.shields.io/badge/-扩展-blue) ![wts](https://img.shields.io/badge/-直写模式-blue) | ![create](https://img.shields.io/badge/-创建-blue) ![delete](https://img.shields.io/badge/-删除-blue) ![resize](https://img.shields.io/badge/-扩展-blue) ![migrate](https://img.shields.io/badge/-迁移-brightgreen) ![clone](https://img.shields.io/badge/-克隆-brightgreen) ![copy](https://img.shields.io/badge/-拷贝-brightgreen) ![snapshot](https://img.shields.io/badge/-快照-brightgreen) ![wts](https://img.shields.io/badge/-直写模式-blue) ![cache](https://img.shields.io/badge/-缓存加速-brightgreen) |
| 存储访问控制 | ![unsupported](https://img.shields.io/badge/-不支持-red) | ![supported](https://img.shields.io/badge/-支持-brightgreen) |
| 存储驱动 | ![supported](https://img.shields.io/badge/-支持-brightgreen) | ![supported](https://img.shields.io/badge/-支持-brightgreen) ![csi](https://img.shields.io/badge/-Kubernetes%20CSI适配-brightgreen) |
| 存储QoS策略 | ![rebalance](https://img.shields.io/badge/-负载均衡-blue) | ![rebalance](https://img.shields.io/badge/-负载均衡-blue) ![access](https://img.shields.io/badge/-数据访问-brightgreen) ![rebuild](https://img.shields.io/badge/-数据重构-brightgreen) |
| 磁盘管理 | ![mount](https://img.shields.io/badge/-挂载-blue) ![umount](https://img.shields.io/badge/-卸载-blue) | ![mount](https://img.shields.io/badge/-挂载-blue) ![umount](https://img.shields.io/badge/-卸载-blue) ![diskled](https://img.shields.io/badge/-磁盘点灯-brightgreen) |
| 节点管理 | ![nodegui](https://img.shields.io/badge/-图形界面-brightgreen) | ![nodegui](https://img.shields.io/badge/-图形界面-brightgreen) |
| 服务管理 | ![servicecli](https://img.shields.io/badge/-命令行-blue) | ![servicegui](https://img.shields.io/badge/-图形界面-brightgreen) |
| 告警管理 | ![unsupported](https://img.shields.io/badge/-不支持-red) | ![alarmgui](https://img.shields.io/badge/-图形界面-brightgreen) |
| 高可用架构 | ![noint](https://img.shields.io/badge/-业务不中断-blue) ![restore](https://img.shields.io/badge/-数据库恢复-blue) ![dic](https://img.shields.io/badge/-分布式InfoCenter-blue) | ![noint](https://img.shields.io/badge/-业务不中断-blue) ![restore](https://img.shields.io/badge/-数据库恢复-blue) ![dic](https://img.shields.io/badge/-分布式InfoCenter-blue) ![distdb](https://img.shields.io/badge/-分布式数据库-brightgreen) |
| 技术支持 | ![community](https://img.shields.io/badge/-社区-blue) | ![supported](https://img.shields.io/badge/-支持-brightgreen) |

# 快速上手
如果您使用的是类 UNIX 系统（如 Linux），可以通过键入下列命令安装编译所需要的软件包：

## CentOS 7 / RHEL 7 编译环境
```bash
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install net-tools maven thrift protobuf-compiler
```

## CentOS 8 / RHEL 8 编译环境
```bash
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install net-tools maven compat-openssl10 protobuf-compiler
yum install https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/t/thrift-0.9.1-15.el7.x86_64.rpm
```

## Debian 10 / Debian 11 / Ubuntu 18 / Ubuntu 20 编译环境
```bash
sudo apt-get install net-tools curl maven protobuf-compiler
curl -LO http://ftp.debian.org/debian/pool/main/t/thrift-compiler/thrift-compiler_0.9.1-2.1+b1_amd64.deb
sudo dpkg -i thrift-compiler_0.9.1-2.1+b1_amd64.deb
```

## openSUSE 15 编译环境
```bash
zypper install net-tools-deprecated curl unzip maven thrift
curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v3.5.1/protoc-3.5.1-linux-x86_64.zip
unzip protoc-3.5.1-linux-x86_64.zip -d /usr/local
```

## 编译命令
在pom.xml所在目录，使用下列 Maven 命令编译软件包：
```bash
mvn versions:use-dep-version -DdepVersion=$(thrift --version | awk '{print $3}') -Dincludes=org.apache.thrift:libthrift
mvn versions:use-dep-version -DdepVersion=$(protoc --version | awk '{print $2}') -Dincludes=com.google.protobuf:protobuf-java
mvn clean install
```

# 更多文档
请参阅 [详细开发文档和使用说明](https://github.com/lanewu/dbs/wiki)。

# 贡献代码

## 当前构建状态
| 操作系统/编译环境   | 状态        | 
| ------------- |:-------------:| 
| Ubuntu 18 / Java 8 | ![ubuntu18_jdk8](https://github.com/lanewu/testci/actions/workflows/ubuntu18_jdk8.yml/badge.svg) |
| Ubuntu 20 / Java 11 | ![ubuntu20_jdk11](https://github.com/lanewu/testci/actions/workflows/ubuntu20_jdk11.yml/badge.svg) |
| Debian 10 / Java 11 | ![debian10_jdk11](https://github.com/lanewu/testci/actions/workflows/debian10_jdk11.yml/badge.svg) |
| Debian 11 / Java 11 | ![debian11_jdk11](https://github.com/lanewu/testci/actions/workflows/debian11_jdk11.yml/badge.svg) |
| CentOS 7 / Java 8 | ![centos7_jdk8](https://github.com/lanewu/testci/actions/workflows/centos7_jdk8.yml/badge.svg) |
| CentOS 8 / Java 8 | ![centos8_jdk8](https://github.com/lanewu/testci/actions/workflows/centos8_jdk8.yml/badge.svg) |

## 编码规范
请在编辑器中设置2空格缩进后，查看和编辑本项目的源代码，每个缩进级别使用一次缩进。空格可用于一行内的其他对齐方式。

大部分代码遵循 [Google Java 风格](https://google.github.io/styleguide/javaguide.html)；也有一些代码遵循 [Oracle 编码约定](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html) —— 这主要取决于最初的版本。 **最重要的是，您修改代码的时候请保持一致，并在修改现有源代码时将空格更改保持在最低限度。** 对于新代码，请使用 Google Java 风格。

## 提交代码
完成代码开发后，您应该向 master 分支提交拉取请求 (PR) 并填写拉取请求模板。拉取请求会自动触发持续整合 (CI) 流程，代码只有在通过 CI 并审核后才会被合并。 如果 CI 运行失败，您可以登录 Jenkins 平台查看失败原因。虽然在审核您的 PR 之前必须满足上述先决条件，但审核者可能会要求您完成额外的设计工作、测试、或其他更改，然后才能最终接受 PR。

更多详细信息，请参阅 [CONTRIBUTING](CONTRIBUTING.md)。

# 许可证
[AGPL 3.0](LICENSE)
