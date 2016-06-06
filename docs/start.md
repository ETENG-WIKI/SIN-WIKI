#Gradle使用

## Gradle安装
Gradle官网 http://gradle.org/
从Gradle官网下载Gradle安装文件，下载完成之后配置环境变量 方法同jdk
检查gradle是否安装完成：gradle –v 

##	Gradle和Eclipse搭建

* 方法一：http://dist.springsource.com/release/TOOLS/gradle
	
	安装方法：
	Windows --->help--->Install New Software

* 方法二：http://marketplace.eclipse.org/content/gradle-ide-pack

	安装方法：
	打开上边网页拖拽到eclipse中 后续同方法一安装过程
## Eclipse和Gradle配置

Eclipse安装插件gradle之后配置

* 配置Gradle(STS)

	 Windows--->Preferences-->Gradle(STS)

	![配置Gradle(STS)](http://i.imgur.com/lRCG9mH.png)

 	Gradle Distribution folder:配置gradle安装目录

	Gradle User Home directory：自定义文件夹.gradle或者配置成gradle安装目录

* 配置jdk

	展开Gradle(STS) 选择arguments如图所示

	![配置Gradle(STS)--->jdk](http://i.imgur.com/PB1WgJN.png)

	选择右边Workspace JRE：选择所需要的jdk

* 配置Gradle EnIDE

	Windows---> Preferences--->Gradle EnIDE
	![配置Gradle EnIDE](http://i.imgur.com/3Pc6Adc.png)

	Gradle home to use:配置Gradle安装目录

	Alternative JAVA_HOME to use 配置jdk的安装目录
##Gradle项目免提交内容配置

Windows--->Preferences--->Team--->Ignored Resources
![gradle忽略提交部分文件](http://i.imgur.com/lZRWLVi.jpg)

根据项目团队要求配置忽略提交的文件或文件夹：build.gradle 
## 配置项目
	
项目地址：http://103.10.84.37/svn/VIS/VISReportBE/trunk

检出后可自行体验 

项目运行 使用gradle appStart（普通启动） 或者gradle appStartDebug（bug模式启动）

使用gradleappStartDebug启动后需配置远程访问如下图

![](http://i.imgur.com/CoJEanj.jpg)

选择用红线选中的任意一个都可以 
点开之后选择Debug Configurations或者Run Configurations

这边选择的是第一个 走的是Debug Configurations如下图：

![](http://i.imgur.com/a8j7EFl.png)

选择 Remote Java Application右键New 

![](http://i.imgur.com/oxSkvXJ.png)

Name:自定义名字建议和项目同名

Project：选择自己使用的项目

Host：项目所在服务器ip

Port：appStartDebug启动时的端口号 一般为5005


##gradle优化方法
* 不用中央仓库。如果你的 repository 配置的是 mavenCentral，放开它吧，全世界的人都在琢磨着怎么虐它，你就不要瞎掺和了。试试 jCenter。

            mavenCentral vs jCenter
            jcenter是世界上最大的Java仓库
            jcenter通过CDN服务，使用的是https协议，安全性更高
            jcenter是mavenCentral的超集，包括许多额外的仓库
            jcenter性能方面比mavenCentral更优
            mavenCentral会自动下载很多与IDE相关的index，而这些用到的少，且不是必需

* 升级最新的 Gradle 版本。

* 开启Gradle的电动小马达：gradle.properties

	文件存放位置 ~/.gradle/ ~表示当前用户目录
	
	gradle.properties文件内容如下

	    #如果你的任务没有时序要求，那么打开这个选项可以并发处理多个任务，充分利用硬件资源。
	     org.gradle.parallel=true
	    #这个也可以在命令行通过参数的形式启动，3个小时有效。守护进程可以使编译时间大大缩短
	     org.gradle.daemon=true
	    #这个看需求吧，Gradle 是运行在 Java 虚拟机上的，这个指定了这个虚拟机的堆内存初始化为256M，最大为1G。如果你内存只有2G，那当我没说。
	    org.gradle.jvmargs=-Xms256m -Xmx1024m