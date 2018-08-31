---
title: Android 持续集成及自动化打包
tags: Android,Gradle
grammar_cjkRuby: true
---
## 开发模式
我司现在的开发模式是分为三个环境进行：
- Dev 开发测试环境
- UAT 用户接收测试环境
- Produce 生产环境

详情请看[关于分支开发的一些想法 ]()

## 构建不同的APK
因APP的环境不同，所以请求的API址也是不一样的，而测试人员基本上也只有一台手机，所以我们需要将不同环境下的APP安装在同一台手机上，也需要有相应的标识符，也就是像下图一样：

![智慧党建](http://obh9jd33g.bkt.clouddn.com/writestory/1535026249620.jpg)

注意看，APP的名字都是一样的，只是启动图标不一样，这是我特意去找UI设计师做得icon，为此还解释了不少为什么。

至于如何进行图标配置，可以看这篇[Gradle构建变种(Product Flavors)]()文章，满满的干货，这里就不再细讲了。

现在一般一点的做法是手动点一下build,输入密钥，点击完成，然后等待编译，等待打包完成，虽然在少量的情况下我们可以手动打包编译，但是要是有几十个渠道要打的话，那岂不是要累得半死？
虽然各大厂都提供了多渠道打包插件，如美团的多渠道打包。
但是，这还是需要大段的等待时间，有这么多时间等待编译打包，不如去喝杯咖啡？

下面就开始进入自动化集成步骤。


## 安装Java并配置环境
 此步骤略....![略过](http://obh9jd33g.bkt.clouddn.com/writestory/1535106288831.png)
## 配置tomcat
当前教程针对于windows。
### 下载tomcat
点击进入官方网站，下载tomcat。
[Tomcat8下载地址](https://tomcat.apache.org/download-80.cgi)

![下载tomcat](http://obh9jd33g.bkt.clouddn.com/writestory/1535100792010.png)

### 解压Tomcat
将下载好的压缩包移动到自己的安装位置，进行解压

![下载好的Tomcat](http://obh9jd33g.bkt.clouddn.com/writestory/1535100896920.png)

解压后的文件夹

![解压后的文件夹](http://obh9jd33g.bkt.clouddn.com/writestory/1535100997579.png)

### 启动tomcat
进入bin目录，点击startup.bat

![startUp.bat](http://obh9jd33g.bkt.clouddn.com/writestory/1535104217682.png)

会运行一个命令行

![启动Tomcat](http://obh9jd33g.bkt.clouddn.com/writestory/1535104290994.png)

等待编译完成后在浏览器中输入localhost:8080  ,我们就可以看到tomcat的欢迎页面了，至此，我们的Tomcat就算启动完成了。

![Tomcat安装完成](http://obh9jd33g.bkt.clouddn.com/writestory/1535104332991.png)

接下来就是配置jenkys了。
## 配置Jenkins
### 下载Jenkins
进入jenkins 的官网 https://jenkins.io/download/ ，直接下载 通用的jenkins war 包 ，这是一个Java的可编译执行文件，每个平台都可以使用，唯一的区别就是Java的安装和tomcat的安装。

![通用war包](http://obh9jd33g.bkt.clouddn.com/writestory/1535100143543.png)


![下载好的war包](http://obh9jd33g.bkt.clouddn.com/writestory/1535100604027.png)

如果下载过慢，这里提供百度云下载地址。
### 编译jenkins
刚刚没有关闭Tomcat,可以先关闭，可以直接点击关闭按钮关闭命令行，也可以进入bin/目录中点击shutdown.bat停止运行。

![shutdown.bat](http://obh9jd33g.bkt.clouddn.com/writestory/1535104459472.png)

将jenkins.war拷贝到tomcat的webapps包下：

![webApp](http://obh9jd33g.bkt.clouddn.com/writestory/1535104537447.png)

### 启动tomcat
重复上面的步骤，进入bin目录，点击startup.bat启动tomcat，等待编译完成。第一次编译时间会比较长。

当命令行出现以下信息，证明启动成功了。

![jenkys启动成功](http://obh9jd33g.bkt.clouddn.com/writestory/1535104826266.png)

注意图中红色方框框煮的字符串，这是等一下登录jenkins做初始化需要用到密码，如果没记下来也没关系，方框下面的路径就是存放密码的位置。一般是在C:\Users\用户名\.jenkins\secrets文件夹下：

![initialAdminPassword](http://obh9jd33g.bkt.clouddn.com/writestory/1535105062331.png)


![密码内容](http://obh9jd33g.bkt.clouddn.com/writestory/1535105128248.png)

### 初始化jenkins
在浏览器中输入：http://localhost:8080/jenkins  我们会看到这样一个页面

![jenkins初始化页面](http://obh9jd33g.bkt.clouddn.com/writestory/1535105402292.png)

这里我们需要输入刚刚初始化好的密码，然后点击继续。

### 初始化插件安装
密码输入成功后会看到一个初始化页面，为了保险起见，我们直接选择安装推荐的插件。

![Jenkins初始化选择](http://obh9jd33g.bkt.clouddn.com/writestory/1535105745441.png)

然后就是等待安装，安装的时长取决于所在的网络情况，就可以先去溜达一会了。![感觉到压力](http://obh9jd33g.bkt.clouddn.com/writestory/1535105919991.png)

![Jenkins等待安装插件](http://obh9jd33g.bkt.clouddn.com/writestory/1535105812835.png)

### 创建新的用户
当插件安装完成后，会跳转到创建用户的界面，这里我们可以先创建一个登录账户。

![创建新的用户](http://obh9jd33g.bkt.clouddn.com/writestory/1535106029272.png)

### 配置访问地址
这一步我们可以对访问地址进行配置，不过大多数情况下都使用默认的即可，不做修改。

![配置访问地址](http://obh9jd33g.bkt.clouddn.com/writestory/1535106099810.png)

配置完成，启动Jenkins吧！

![启动Jenkins](http://obh9jd33g.bkt.clouddn.com/writestory/1535106138460.png)

注意：tomcat的命令行不能关闭！！
注意：tomcat的命令行不能关闭！！
注意：tomcat的命令行不能关闭！！

## 相关配置
启动成功后我们需要进行相关构建工具和环境变量的配置。

### 配置JDK
打开【系统管理】->【全局工具配置】

![JAVA_HOME](http://obh9jd33g.bkt.clouddn.com/writestory/1535365136061.png)


### 配置Android SDK
点击

- 系统管理
		- 系统设置
			- 找到全局属性

![环境配置](http://obh9jd33g.bkt.clouddn.com/writestory/1535106857221.png)

找到全局变量，点击新增

![全局变量](http://obh9jd33g.bkt.clouddn.com/writestory/1535106936182.png)

输入键值对：
键：ANDROID_HOME
值：SDK路径


![输入键值对](http://obh9jd33g.bkt.clouddn.com/writestory/1535107032480.png)
### 配置Gradle
- 系统管理
		- 系统设置
			- 全局工具配置
				- Gradle

新增Gradle

![新增Gradle]http://obh9jd33g.bkt.clouddn.com/writestory/1535697064928.png)

需要注意的是，这里的gradle版本要和你项目中的保持一致，以免发生其他错误。

![项目中的gradle版本](http://obh9jd33g.bkt.clouddn.com/writestory/1535697124129.png)

### 配置svn

### 配置Git
打开
- 系统管理
		- 系统设置
			- 全局工具配置
				- git

配置你本地的git.exe的安装位置，如果没有、可以选择自动安装

![配置git](http://obh9jd33g.bkt.clouddn.com/writestory/1535697298511.png)

## 创建自动化任务

### 创建一个新的任务

![创建新的任务](http://obh9jd33g.bkt.clouddn.com/writestory/1535106563826.png)

输入任务名称，任务类型选择【构建一个自由风格的软件项目】

![创建任务](http://obh9jd33g.bkt.clouddn.com/writestory/1535106651834.png)

### 配置任务信息
#### 源码配置
Jenkins可以使用git和Subversion，而我们公司使用的是Subversion，这里就以Subversion作为演示，两个在配置上没有什么区别。

![源码配置](http://obh9jd33g.bkt.clouddn.com/writestory/1535107277549.png)

Repository URL  填入我们的仓库地址

Credentials是认证信息，我们点击![添加](http://obh9jd33g.bkt.clouddn.com/writestory/1535107396494.png)新增一个认证信息。

![添加认证信息](http://obh9jd33g.bkt.clouddn.com/writestory/1535107478656.png)

在上图中填入访问仓库的账号和密码，点击【添加】完成。然后选择我们刚刚添加的认证信息。

![选择认证信息](http://obh9jd33g.bkt.clouddn.com/writestory/1535107586431.png)

#### 构建配置
配置Gradle

![配置Gradle](http://obh9jd33g.bkt.clouddn.com/writestory/1535364624574.png)



#### 构建成功
![构建成功](http://obh9jd33g.bkt.clouddn.com/writestory/1535698487446.png)

## 其他错误

### 2. The SDK directory 'D:\Development Tool\Android\sdk' does not exist
版本控制器中的sdk配置和自动化部署电脑上的位置不一致，需要手动更改一下。
以后的版本最好不要提交sdk配置文件。

![enter description here](http://obh9jd33g.bkt.clouddn.com/writestory/1535697557641.png)

