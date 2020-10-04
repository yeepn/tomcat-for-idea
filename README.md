# Tomcat IDEA源码调试环境搭建

# 前言
前段时间把tomcat的源码下载到本地，计划看看tomcat源码，了解一下其基本原理。但是，由于种种原因半途而废了。最近突然想搭建一下源码调试环境，发现网上许多文章都无法成功运行，最终参考多篇文章，使项目可以运行，做一下笔记进行记录。
# 1. 采坑记录
以tomcat8.5.x为例，笔者是从github上获取的tomcat源码，但是tomcat的源码是通过ant构建的，idea直接打开后，把java目录标记为Sources Root，test目录标记为Test Sources Root后，有部分代码报错，删除或注释报错代码即可。
#### 1.1 目录结构
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601801575207-5fd06b02-fdcf-4859-a2c8-cd423cc8f8e2.png#align=left&display=inline&height=658&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1132&originWidth=1284&size=103018&status=done&style=none&width=746)
#### 1.2 pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>tomcat</name>

    <dependencies>
        <dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.10.7</version>
        </dependency>
        <dependency>
            <groupId>javax.xml</groupId>
            <artifactId>jaxrpc</artifactId>
            <version>1.1</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.easymock</groupId>
            <artifactId>easymock</artifactId>
            <version>3.6</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jdt.core.compiler</groupId>
            <artifactId>ecj</artifactId>
            <version>4.5.1</version>
        </dependency>
        <dependency>
            <groupId>wsdl4j</groupId>
            <artifactId>wsdl4j</artifactId>
            <version>1.6.2</version>
        </dependency>
    </dependencies>

    <build>
        <finalName>tomcat</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>java</directory>
            </resource>
        </resources>
        <sourceDirectory>java</sourceDirectory>
        <testResources>
            <testResource>
                <directory>test</directory>
            </testResource>
        </testResources>
        <testSourceDirectory>test</testSourceDirectory>
    </build>
</project>
```
附件：[pom.xml](https://www.yuque.com/attachments/yuque/0/2020/xml/165192/1601825002279-d719c4d5-fe3f-4c30-8e9b-1b9d4379e877.xml?_lake_card=%7B%22uid%22%3A%221601825002207-0%22%2C%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2020%2Fxml%2F165192%2F1601825002279-d719c4d5-fe3f-4c30-8e9b-1b9d4379e877.xml%22%2C%22name%22%3A%22pom.xml%22%2C%22size%22%3A2395%2C%22type%22%3A%22text%2Fxml%22%2C%22ext%22%3A%22xml%22%2C%22progress%22%3A%7B%22percent%22%3A99%7D%2C%22status%22%3A%22done%22%2C%22percent%22%3A0%2C%22id%22%3A%22uiIog%22%2C%22card%22%3A%22file%22%7D)
#### 1.3 启动项目
我们运行org.apache.catalina.startup.Bootstrap的main方法，发现项目直接报错了，如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601801950581-247905ba-bd73-400b-b0e2-47f1fd7289e5.png#align=left&display=inline&height=546&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1092&originWidth=2518&size=333851&status=done&style=none&width=1259)
网上很多说是需要设置Tomcat的home目录，即jvm参数-Dcatalina.home参数。笔者参考网上的做法，建立一个catalina-home目录，把bin, conf, webapps, work目录拷贝到catalina-home中，并设置jvm参数-Dcatalina.home="/Users/liuyang/project/java/source/tomcat/catalina-home"，这里请根据自己的实际路径进行设置，如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601802249722-b6b71501-bbd3-48f9-afa6-cf3315b0efdc.png#align=left&display=inline&height=482&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1380&originWidth=2134&size=271966&status=done&style=none&width=746)
设置完会发现依然报错，辣鸡教程，抄的千篇一律。
我们进行分析一下，bin, conf, webapps, work目录拷贝自项目根目录，为啥不能直接设置为项目根目录。我们把这个jvm参数去除。然后在Bootstrap打印catalina.home参数配置值：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601806398756-7957c455-acd2-4a5d-a29b-4778f51bcde5.png#align=left&display=inline&height=61&margin=%5Bobject%20Object%5D&name=image.png&originHeight=122&originWidth=1910&size=28947&status=done&style=none&width=955)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601806504366-3ff3eb8a-cb57-4244-9cd8-094df8cd2394.png#align=left&display=inline&height=856&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1712&originWidth=1878&size=432710&status=done&style=none&width=939)
可以看出，默认catalina.home是当前工程目录，但是项目依然报错。
# 2. 正确打开方式
## 2.1 Maven+Ant方式
### 2.1.1 准备Ant环境
ant环境需要安装ant环境，ant安装有多种方式，通过brew安装（Mac系统且安装好brew）可以执行如下命令：
```shell
brew install ant
```
到ant官方下载最新的zip包进行安装，与maven、jdk等环境安装类似，解压zip包，导出环境变量即可，mac环境下.bash_profile可以添加如下内容：
```shell
#ant environment setting
export ANT_HOME=/usr/local/ant
export PATH=${PATH}:${ANT_HOME}/bin
```
### 2.1.2 准备Maven环境
maven环境依赖jdk，jdk和maven的安装请自行搜索。
### 2.1.3 下载源码
源码可以通过[tomcat](https://tomcat.apache.org/)官网或者[github](https://github.com/apache/tomcat)下载，下载方式方式如下：

- [https://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.58/src/apache-tomcat-8.5.58-src.zip](https://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.58/src/apache-tomcat-8.5.58-src.zip)
- git clone [https://github.com/apache/tomcat.git](https://github.com/apache/tomcat.git)

到tomcat官网自行下载时需要注意，我们要下载的源码包而不是编译好的Tomcat，可按照下图标记进行下载：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601824217077-fb8b5670-7d4d-4506-9722-ff498a3b3c54.png#align=left&display=inline&height=570&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1140&originWidth=2152&size=182947&status=done&style=none&width=1076)
### 2.1.4 导入IDEA
笔者的IDAE版本为2020.1.4 Ultimate，我们将下载后的zip包进行解压，并1.2步骤的pom.xml放到解压包的根目录下，然后用IDEA打开，如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601825129087-330e7eb0-699e-4a84-9878-e919ffd076e3.png#align=left&display=inline&height=1023&margin=%5Bobject%20Object%5D&name=image.png&originHeight=2046&originWidth=2864&size=804627&status=done&style=none&width=1432)
然后点击pom.xml右键选择“Add as Maven Project”导入为Maven工程。IDEA进行编译后，我们发现有报错的类util.TestCookieFilter，我们将此测试类删除即可。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601825347365-ef5045d8-b035-4bbe-ab64-acb652adf8fa.png#align=left&display=inline&height=859&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1718&originWidth=3108&size=508053&status=done&style=none&width=1554)
### 2.1.5 Ant构建Tomcat
#### 2.1.5.1 ant插件安装并导入为ant工程
ant构建Tomcat需要启动IDEA的ant插件，没有启用或安装的话，请自行安装并启用ant插件，启动ant插件后效果如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601825606916-518478b7-3a1c-4016-b6a4-c4d66020004e.png#align=left&display=inline&height=960&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1920&originWidth=3760&size=806368&status=done&style=none&width=1880)
我们按照图示，将build.xml进行“Add as Build File”操作后，效果如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601825762967-5978380e-1980-4791-8a8a-36b07d58b33f.png#align=left&display=inline&height=583&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1166&originWidth=2762&size=384605&status=done&style=none&width=1381)
#### 2.1.5.2 ant命令build下载依赖
我们在项目根目录命令行执行如下命令：
```shell
#下面2个命令任意一个即可
#ant -buildfile build.xml ide-intellij
ant ide-intellij
```
执行命令后，我们发现构建失败，如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601826033427-d9331657-de70-4fd9-b68b-3ad898661902.png#align=left&display=inline&height=706&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1412&originWidth=3704&size=366324&status=done&style=none&width=1852)
判断是commons-daemon-1.2.2.jar下载失败了，对应链接点击进去发现404。我们可以修改build.properties.default文件的配置，将commons-daemon-1.2.2升级为commons-daemon-1.2.3，再执行命令即可，如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601826235596-b9bd3799-ed1c-423f-8117-c264f1bc5f2f.png#align=left&display=inline&height=603&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1206&originWidth=2428&size=377641&status=done&style=none&width=1214)
实际命令执行过程中需要从中央仓库下载jar包，可能会有些慢，稍等片刻即可，命令正确执行完后，会有如下提示：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601826413610-c8af9540-56d9-489e-8ae7-25bb1690f308.png#align=left&display=inline&height=380&margin=%5Bobject%20Object%5D&name=image.png&originHeight=844&originWidth=1658&size=94138&status=done&style=none&width=746)
默认情况下，下载的jar包会放置到用户根目录下的tomcat-build-libs目录下，文件如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601826559824-3434a132-f2da-4de8-89ab-b264b34367f3.png#align=left&display=inline&height=437&margin=%5Bobject%20Object%5D&name=image.png&originHeight=708&originWidth=1210&size=166692&status=done&style=none&width=746)
#### 2.1.5.3 IDEA设置**PATH VARIABLES**
在上一步ant ide-intellij执行完，最后其实我们发现命令行已经提示我们设置**PATH VARIABLES**，设置过程可以参考下图：
```shell
ANT_HOME=/usr/local/ant
TOMCAT_BUILD_LIBS=/Users/liuyang/tomcat-build-libs
```
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601826873998-e506882a-504d-4fd1-86c6-86920c763e78.png#align=left&display=inline&height=388&margin=%5Bobject%20Object%5D&name=image.png&originHeight=776&originWidth=2442&size=192181&status=done&style=none&width=1221)
#### 2.1.5.4 ant deploy生成build文件
接着我们可以进行deploy操作进行构建了，如下图所示：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601826654103-ff430eb5-4f83-4d6f-bfc1-0c920652944d.png#align=left&display=inline&height=736&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1290&originWidth=1308&size=139411&status=done&style=none&width=746)
deploy操作完成后，我们运行org.apache.catalina.startup.Bootstrap的main方法，发现如下报错：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601827579973-6d05e715-381f-45bc-898d-e0b8f08028b6.png#align=left&display=inline&height=670&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1340&originWidth=2632&size=456203&status=done&style=none&width=1316)
发生错误的原因是我们未设置catalina.home JVM参数。
### 2.1.6 设置VM Options
tomcat源码运行需要一个编译好的目录作为catalina.home才能运行，这里我们选择上一步生成的output/build作为catalina.home，并设置jvm参数，如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601827875886-e64c62ff-25f8-4519-a2a9-9d3dbff0045b.png#align=left&display=inline&height=460&margin=%5Bobject%20Object%5D&name=image.png&originHeight=644&originWidth=1044&size=51728&status=done&style=none&width=746)
```
-Dcatalina.home="output/build"
-Dfile.encoding=UTF8
-Duser.language=en
-Duser.region=US
```
第一行是为了设置catalina.home，2~4行是为了日志解决乱码问题（非必要）。
我们设置JVM参数，可以参考下图：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601827980738-fe89bd0a-50eb-4cc2-8a4b-0451ca7c718e.png#align=left&display=inline&height=671&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1342&originWidth=2144&size=302179&status=done&style=none&width=1072)
设置好，我们运行org.apache.catalina.startup.Bootstrap的main方法，控制台提示如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601828125650-102bce5e-2277-4495-aa7c-f5973946d0e1.png#align=left&display=inline&height=661&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1322&originWidth=2086&size=363238&status=done&style=none&width=1043)
我们打开浏览器，访问[http://localhost:8080/](http://localhost:8080/)显示如下内容，则说明容器启动成功了，如下：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601828223828-ef453ddd-ffe6-45c2-a586-616ca69ac8dd.png#align=left&display=inline&height=960&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1920&originWidth=3350&size=716093&status=done&style=none&width=1675)


## 2.2 Maven+下载Tomcat编译包
2.1.5做的操作其实就是编译tomcat源码包，构建可运行的tomcat编译文件，过程有些复杂，而且要踩很多坑。我们可以去Tomcat官网，下载源码版本对应的编译输出文件的zip包，直接将zip包解压后内容(bin lib conf work webapps目录)，复制到${项目根目录}/output/build目录下即可，进而代替自己通过ant构建。如果是从Tomcat官网下载的，可以完全忽略步骤2.1.1和步骤2.1.5，如下图所示：
![image.png](https://cdn.nlark.com/yuque/0/2020/png/165192/1601829006815-466d6f2f-13a6-4db3-8abb-02792bc0b962.png#align=left&display=inline&height=594&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1188&originWidth=2910&size=205282&status=done&style=none&width=1455)
## 2.3 Git仓库下载
笔者将搭建的环境上传到了远程git仓库，可以直接下载仓库代码，然后按照步骤2.1.6配置VM Options即可。仓库地址如下：[https://gitee.com/rxyor_source/tomcat-for-idea.git](https://gitee.com/rxyor_source/tomcat-for-idea.git) 。不要忘记给笔者点个Star哟！
# 参考
[https://developer.aliyun.com/article/666276](https://developer.aliyun.com/article/666276)
[https://www.yuque.com/alipayrrql7pkhl7/ofayer/mxmcz9](https://www.yuque.com/alipayrrql7pkhl7/ofayer/mxmcz9)
