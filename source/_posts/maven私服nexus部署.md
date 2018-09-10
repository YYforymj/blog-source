---
title: maven 私服 nexus 部署
date: 2018-09-04 09:17:01
categories: maven

---

> maven 是 java web 开发中最常用的依赖管理工具，而 nexus 是 maven 的镜像仓库，本文参照了 《maven 实战》一书与 nexus 官网文档，进行了 nexus私服的部署。

<!-- more -->

#### maven 私服 nexus 部署

nexus 是sonatype提供的maven私服软件，其他的私服软件可以参照《maven实战》中私服章节或通过网络搜索。

本次部署使用nexus2 安装在 windows server 2012 环境下，sonatype提供nexus2与nexus3，之前本地部署测试的时候3有一点问题，直接使用的2，故本文档是针对nexus2的部署教程。对于3的下载与部署，可以参考官方文档。官网文档链接如下：

https://help.sonatype.com/repomanager3

本次使用 2.14.8 版本，可以在下面的连接下载。

https://help.sonatype.com/repomanager2/download/download-archives---repository-manager-2

官方文档位置

https://help.sonatype.com/repomanager2

具体步骤：

1. 从官网下载 `nexus-2.14.8-01-bundle.zip` 后，拷贝至服务器，此处需要注意windows系统下不要拷贝到 `program files` 文件夹，官方文档提示说可能会有问题。
2. 解压到需要安装的文件夹，例如 `C:\nexus-2.14.8-01-bundle` ，解压后会有两个文件夹，其中 `nexus-2.14.8-01` 为程序文件夹，`sonatype-work` 为数据文件夹；
3. 进入`"安装目录"\nexus-2.14.8-01-bundle\nexus-2.14.8-01\bin\jsw`，根据当前系统进入不同文件夹，此次使用windows server 2012 64bit，故进入`windows-x86-64`
4. 开启nexus服务直接使用管理员身份运行 `start-nexus.bat` 即可，停止服务使用 `stop-nexus.bat\` ；如果想要开机自启，可以使用管理员身份运行`install-nexus.bat` ，nexus会注册为系统服务，`uninstall-nexus.bat` 会卸载注册的nexus 系统服务。

其他注意事项：

1. 端口与url配置，nexus默认使用8081端口，url为`/nexus`，这两项可以在`"安装目录"\nexus-2.14.8-01-bundle\nexus-2.14.8-01\conf`下的`nexus.properties`文件中进行配置，实际上nexus就是使用的jetty作为容器；
2. 部署后一般外部仍然无法访问，需要在防火墙中配置放开对应的端口；对于windows server，在防火墙中配置入栈规则，配置端口为需要的端口即可。
3. nexus 默认管理员用户名为`admin`，密码为`admin123`，登录管理员用户后可以对nexus进行设置，添加代理仓库等，详细信息可以参考《maven实战》第九章。

