---
title: 搭建maven私服nexus
date: 2017-12-26 21:30:16
tags: [maven,nexus]
categories: maven
---
maven私服搭建，目的就是我们在使用maven打包时，如果私服中有相对应的包时，可以直接拉取过来，而不需要去下载，仅仅第一次使用该包时才会下载，这样会减少很多的时间，提高效率。
<!-- more -->
# 安装配置nexus
下载：<a href=https://www.sonatype.com/download-oss-sonatype>nexus下载</a>
解压：
```
$ tar zxvf nexus-3.6.1-02-unix.tar.gz
```
nexus详解文档参考<a href=https://www.xncoding.com/2017/09/02/tool/nexus.html>
## 修改启动用户
```
$ vim $NEXUS_HOME/bin/nexus.rc
```
## 修改默认端口
```
$ vim $NEXUS_HOME/ etc/nexus-default.properties
```
## 启动
```
$ ./bin/nexus run
```
## 访问
浏览器访问8081端口，默认登陆：
user: admin
password: admin123
## 配置
maven-central：maven中央库，默认从https://repo1.maven.org/maven2上拉取jar包；
maven-releases：私库发行版jar，初次安装nexus请将Deployment policy设置成Allow redeploy；
maven-snapshots：私库快照调试版本jar；
maven-public：仓库分组，把上面三个仓库组合起来一起对外提供服务，在本地maven配置settings.xml中使用。

# 本地maven使用私服nexus
## maven默认配置settings.xml
```
<servers>
    <server>
        <id>releases</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
    <server>
        <id>snapshots</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</servers>

<mirrors>
    <mirror>
        <id>nexus</id>
        <mirrorOf>*</mirrorOf>
        <url>http://123.207.66.156:8081/repository/maven-public/</url>
    </mirror>
</mirrors>

<profiles>
    <profile>  
      <id>dev</id>
      <repositories>
        <repository>
          <id>Nexus</id>
          <url>http://123.207.66.156:8081/repository/maven-public/</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </repository>
      </repositories>
    </profile>
</profiles>
<activeProfiles>
    <activeProfile>dev</activeProfile>
</activeProfiles>
```
## 修改工程pom.xml
```
<distributionManagement>
    <repository>
        <id>releases</id>
        <name>Releases</name>
        <url>http://123.207.66.156:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <name>Snapshot</name>
        <url>http://123.207.66.156:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```
注意上面的repository的id值一定要跟settings.xml文件中配置的server一致。