---
title: 在Mac上使用Multipass运行Ubuntu虚拟机
abbrlink: 5ba8a50c
date: 2022-01-16 15:47:37
updated: 2022-03-16 20:16:37
tags:
---

[**Multipass**](https://multipass.run/) 是一个轻量虚拟机管理器，是由 **Ubuntu** 运营公司 **Canonical** 所推出的[开源项目](https://github.com/canonical/multipass)。目前支持 **Linux**、**Windows**、**macOS**。**Multipass** 提供了一个命令行工具，通过它可以在几分钟内启动并运行一个崭新的 **Ubuntu** 虚拟机

<!-- more -->
## Install

### 下载安装包

在 [**Multipass**](https://multipass.run/)官网下载安装包

![](image.png)

### Homebrew 安装

没有安装 `Homebrew`? [点击这里安装 Homebrew](https://link.segmentfault.com/?enc=1L8at0RPojKOLDBF33vHSw==.qRVlnGJxmxiBPCV618xdCQ==)

```Bash
# 安装
$ brew install multipass

# 如果想要更新multipass版本
$ brew upgrade multipass
```




## Use

首先先看一下 **multipass** 有哪些命令

```Bash
$ multipass  -h
Usage: multipass [options] <command>
Create, control and connect to Ubuntu instances.

This is a command line utility for multipass, a
service that manages Ubuntu instances.

Options:
  -h, --help     Displays help on commandline options.
  --help-all     Displays help including Qt specific options.
  -v, --verbose  Increase logging verbosity. Repeat the 'v' in the short option
                 for more detail. Maximum verbosity is obtained with 4 (or more)
                 v's, i.e. -vvvv.

Available commands:
  alias     Create an alias
  aliases   List available aliases
  delete    Delete instances
  exec      Run a command on an instance
  find      Display available images to create instances from
  get       Get a configuration setting
  help      Display help about a command
  info      Display information about instances
  launch    Create and start an Ubuntu instance
  list      List all available instances
  mount     Mount a local directory in the instance
  networks  List available network interfaces
  purge     Purge all deleted instances permanently
  recover   Recover deleted instances
  restart   Restart instances
  set       Set a configuration setting
  shell     Open a shell on a running instance
  start     Start instances
  stop      Stop running instances
  suspend   Suspend running instances
  transfer  Transfer files between the host and instances
  umount    Unmount a directory from an instance
  unalias   Remove an alias
  version   Show version details
```




## Launch

我们使用 **multipass** 来创建一个 **Ubuntu** 虚拟机。**multipass** 中也有镜像的概念，通过 `multipass find` 命令可以查看当前有哪些镜像

```Bash
$ multipass find
Image                       Aliases           Version          Description
18.04                       bionic            20220104         Ubuntu 18.04 LTS
20.04                       focal,lts         20220111         Ubuntu 20.04 LTS
21.04                       hirsute           20220113         Ubuntu 21.04
21.10                       impish            20220111         Ubuntu 21.10
anbox-cloud-appliance                         latest           Anbox Cloud Appliance
minikube                                      latest           minikube is local Kubernetes
```


使用 `multipass launch` 命令创建一个 **Ubuntu20.04** 的虚拟机

```Bash
# -n 虚拟机名称 -c CPU -m 内存 -d 磁盘大小  20.04 指定镜像
$ multipass launch -n ubuntu -c 2 -m 4G -d 10G 20.04
Launched: ubuntu

```
{% note info %}
`launch` 时更多参数可以通过 `multipass launch -h` 查看
{% endnote %}



## List && Info

查看运行中的虚拟机列表

```Bash
$ multipass list
Name                    State             IPv4             Image
ubuntu                  Running           192.168.64.4     Ubuntu 20.04 LTS
```


还可以通过 `multipass info` 查看虚拟机详细信息

```Bash
$ multipass info ubuntu            
Name:           ubuntu
State:          Running
IPv4:           192.168.64.4
Release:        Ubuntu 20.04.3 LTS
Image hash:     e9cae16ff305 (Ubuntu 20.04 LTS)
Load:           0.00 0.00 0.00
Disk usage:     1.2G out of 4.7G
Memory usage:   161.6M out of 3.8G
Mounts:         --
```




## Exec && Shell

进入虚拟机执行命令，有两种方式可以进入虚拟机执行命令。

### exec

```Bash
$ multipass exec ubuntu uname      
Linux

# 如果要传递参数就需要用以下这种格式
$ multipass exec ubuntu -- uname -a
Linux ubuntu 5.4.0-94-generic #106-Ubuntu SMP Thu Jan 6 23:59:31 UTC 2022 aarch64 aarch64 aarch64 GNU/Linux

```




### shell

使用 `exec` 时每次都要添加 `multipass exec ubuntu` ，`multipass` 还支持直接进入虚拟机的 `shell`

```Bash
$multipass shell ubuntu     
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-94-generic aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Jan 16 15:04:58 CST 2022

  System load:             0.0
  Usage of /:              26.6% of 4.68GB
  Memory usage:            5%
  Swap usage:              0%
  Processes:               119
  Users logged in:         0
  IPv4 address for enp0s1: 192.168.64.4
  IPv6 address for enp0s1: fddf:1872:81b9:5408:5054:ff:fea7:7dc4


0 updates can be applied immediately.


Last login: Sun Jan 16 15:03:53 2022 from 192.168.64.1
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ubuntu:~$ uname 
Linux
ubuntu@ubuntu:~$
```




## mount && transfer

**multipass** 也提供了宿主机和虚拟机之间同步文件

### mount

顾名思义，**mount** 可以将宿主机和虚拟机中的目录或者文件进行挂载。**mount** 后无论在宿主机还是虚拟机内进行操作，两边看到的都是一样的。熟悉 **docker** 的会知道这和 `docker -v` 参数一样。

```Bash
$ multipass mount /var/ubuntu/test  ubuntu:/test

```


### transfer

**multipass** 还提供了 **transfer** 命令，用来在宿主机和虚拟机直接传输文件，但使用 **transfer** 命令拷贝的文件，在拷贝完成后就不会同步，相当于宿主机和虚拟机对应两份文件。

```Bash
$ echo hello > test.txt
$ multipass transfer ./test.txt ubuntu:/home/ubuntu/test.txt

```




## Delete

如果要删除一个虚拟机也很方便

```Bash
$ multipass stop ubuntu
$ multipass delete vm01
$ multipass purge

```






## cloud-init

在  [multipass官方文档](https://multipass.run/docs)中说到

![](image_1.png)

意思是可以在创建虚拟机时通过 `—cloud-int` 指定 `cloud-init.yml` 文件。

有什么用呢？比方说我们每个虚拟机都要配置 `ssh` 或者需要通过 `apt` 安装一些工具，就可以将这些操作都写到 `cloud-init.yml` 文件中，这样每次创建虚拟机时就会自动执行这些环境的配置。

```Bash
#cloud-config

# run commands
# default: none
# runcmd contains a list of either lists or a string
# each item will be executed in order at rc.local like level with
# output to the console
# - runcmd only runs during the first boot
# - if the item is a list, the items will be properly executed as if
#   passed to execve(3) (with the first arg as the command).
# - if the item is a string, it will be simply written to the file and
#   will be interpreted by 'sh'
#
# Note, that the list has to be proper yaml, so you have to quote
# any characters yaml would eat (':' can be problematic)
runcmd:
 - [ ls, -l, / ]
 - [ sh, -xc, "echo $(date) ': hello world!'" ]
 - [ sh, -c, echo "=========hello world'=========" ]
 - ls -l /root
 # Note: Don't write files to /tmp from cloud-init use /run/somedir instead.
 # Early boot environments can race systemd-tmpfiles-clean LP: #1707222.
 - mkdir /run/mydir
 - [ wget, "http://slashdot.org", -O, /run/mydir/index.html ]
```


更多关于[cloud-init.yml 文件的写法](https://cloudinit.readthedocs.io/en/latest/)



