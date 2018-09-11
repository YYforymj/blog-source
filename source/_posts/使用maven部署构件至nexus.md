---
title: 使用 maven 部署构件至 nexus
date: 2018-09-11 20:33:01
categories: maven


---

> 之前写了篇博客记录了 nexus 私服的部署，其实这篇跟那一篇是一起写的，但是忘记发布了，这里补上。这篇主要记录了使用 nexus 私服后，自己开发的项目如何推送至 nexus ，方便公司或者项目组内部使用。 

<!-- more -->

首先在项目中添加 nexus 服务器信息，在<project>节点下添加如下内容，其中 <url>为nexus服务器url，区分release与snapshot；x.x.x.x 为 nexus 部署的ip ；port为端口，一般默认为8081。

```
<distributionManagement> <!-- 远程部署管理信息 -->
        <repository>
            <id>localNexusRelease</id>
            <name>Nexus Release Repository</name>
          <url>http://x.x.x.x:port/nexus/content/repositories/releases/</url>
        </repository>
        <snapshotRepository>
            <id>localNexusSnapshots</id>
            <name>Nexus Snapshot Repository</name>
            <url>http://x.x.x.x:port/nexus/content/repositories/snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
```

其次在maven的setting.xml下的<servers>节点添加如下内容（推荐使用外部maven，如果要使用idea自带maven，setting.xml在JetBrains安装目录下 \IntelliJ IDEA 2018.1.4\plugins\maven\lib\maven3\conf\setting.xml，此处没做测试）

```
	<server>
      <id>localNexusRelease</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
  	<id>localNexusSnapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
```

其中username与password为用户名与密码，nexus默认为admin与admin123；

部署时，使用maven的deploy命令，当提示build success时即代表部署至nexus成功。