---
layout: post
title: Apache Maven上手指南
categories:
- Java开发
tags:
- [java,maven,上手,指南]
---
<link rel="stylesheet" href="/media/highlight/styles/github.css">
<script src="/media/highlight/highlight.pack.js"></script>
<script>
$(document).ready(function() {
  $('h5').css('margin-bottom','10px')
  $('h4').css('margin-bottom','16px')
  $('pre code').each(function(i, block) {
    hljs.highlightBlock(block);
  });
});
</script>

###什么是Maven
Maven 是一个项目管理与构建（Build）工具，不过对于程序员来说我们主要关注它提供的自动构建和依赖jar包的管理功能。

> 1. 自动化构建。Maven自定义了一套构建生命周期过程，从清理（clean）、编译（compile）、测试（test）与生成报告，到打包（package）和部署（deploy），只需执行相应命令（如 mvn compile）即可由maven帮我们完成相应的操作。
2. 第三方依赖（jar包）管理。Maven在构建过程中会自动从配置的maven 库（默认为[Maven的中央库][1]）中下载该项目依赖的所有jar包到本地库中，并将其引入项目的编译路径中。

###安装Maven
####命令行
 1.确保已经安装过JDK，否则需要先到Oracle官网下载JDK安装。然后到https://maven.apache.org/download.cgi下载apache-maven-x.x.x-bin.zip后解压到安装目录（如D:\Program Files (x86)\apache-maven）。  
 2.接下来配置环境变量，新增`M2_HOME`指向之前的解压目录，并在path中增加% `%M2_HOME%\bin;`，如下：
> ![M2_HOME](/media/pic2015/m2home.png)
![M2_HOME_PATH](/media/pic2015/maven-path.png)

 打开命令行，输入`mvn -v`查看是否安装成功：
<pre><code class="dos">C:\Users\Walker>mvn -v
Apache Maven 3.2.5 (12a6b3acb947671f09b81f49094c53f426d8cea1; 2014-12-15T01:29:2
3+08:00)
Maven home: D:\Program Files (x86)\apache-maven
Java version: 1.8.0, vendor: Oracle Corporation
Java home: D:\Java\jdk1.8.0\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 8.1", version: "6.3", arch: "x86", family: "dos"
</code></pre>
3.接下来打开maven安装目录下`conf\settings.xml`文件，查找`localRepository`，若是没找到或是被注释掉了，则新增一行：
<pre><code class="xml">&lt;localRepository&gt;/path/to/local/repo&lt;/localRepository&gt;
</code></pre>
这个字段主要用来配置本地库的路径，因为以后所有从远程库下载的jar包都会存放在这里，windows下建议改到一个空间较大的盘下（默认路径为`${user.home}/.m2/repository`），比如将上面的`/path/to/local/repo`替换为`D:/Users/walker/.m2/repository`。

4.增加Maven远程库镜像。因为默认的远程库可能被墙等原因导致无法访问的，所以需要增加国内的maven库镜像，不过目前国内只有开源中国的源。
 同样是修改`conf\settings.xml`，在`<mirrors>..</mirrors>`中新增如下：
<pre><code class="xml">&lt;mirror&gt;
	&lt;id&gt;nexus-osc&lt;/id&gt;
	&lt;mirrorOf&gt;*&lt;/mirrorOf&gt;
	&lt;name&gt;Nexus osc&lt;/name&gt;
	&lt;url&gt;http://maven.oschina.net/content/groups/public/&lt;/url&gt;
&lt;/mirror&gt; 
</code></pre>
具体的修改可以参考 [开源中国 Maven 库使用帮助][2]。
####IDE插件
如果你使用的是最新版的Eclipse或IntelliJ IDEA，这些工具已经集成maven，否则需要到Eclipse Marketplace或IDEA Plugin Repository下载。  
Eclipse需要修改的地方如下：

> ![Eclipse Maven](/media/pic2015/eclipse-maven.png)

###使用Maven

通常情况下，我们会从命令行进入pom.xml所在目录，执行命令。  
比如运行：
<pre><code class="dos">mvn compile</code></pre>
该命令将编译整个工程，生成class文件。
或者运行：
<pre><code class="dos">mvn clean intall</code></pre>
将清理整个工程（自动生成的class和其他资源文件都被清理掉）重新构建并安装。  
另外Maven 使用惯例优于配置的原则 。它要求项目有如下的结构：
<pre><code class="dos">project
 ├─src
 │  ├─main
 │  │  ├─java
 │  │  │  └─org
 │  │  │      └─greycode
 │  │  └─resources
 │  └─test
 │      ├─java
 │      │  └─org
 │      │      └─greycode
 │      └─resources
 └─target
    ├─classes
    │  └─org
    │      └─greycode
    └─...
</code></pre>
`main` 和 `test` 分别为项目的源代码和测试代码目录，`resources`主要是用于存放其他资源文件比如log4j2.xml，spring-root.xml等。 使用Maven编译后的class会放在`target/classes`目录下。

等等！你可能会想问pom.xml是什么鬼，而`mvn compile`和`mvn install`有什么区别，这些命令又让maven在后台做了些什么？

嗯……所以有些概念还是得了解一下。
###一些概念

#####POM（Project Object Model）
Maven使用名为pom.xml的文件来描述项目的信息、输入输出目录、依赖的第三方jar包、需要使用的maven插件以及在构建项目过程中会采取的行为（比如如何打包部署等）。
一个典型的pom.xml：
<pre><code class="xml">&lt;project xmlns=&quot;http://maven.apache.org/POM/4.0.0&quot; xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
	xsi:schemaLocation=&quot;http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd&quot;&gt;

	&lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;
	&lt;groupId&gt;org.greycode.demo&lt;/groupId&gt;
	&lt;artifactId&gt;maven-demo&lt;/artifactId&gt;
	&lt;version&gt;0.0.1-SNAPSHOT&lt;/version&gt;

	&lt;dependencies&gt;
		&lt;dependency&gt;
			&lt;groupId&gt;com.google.guava&lt;/groupId&gt;
			&lt;artifactId&gt;guava&lt;/artifactId&gt;
			&lt;version&gt;18.0&lt;/version&gt;
		&lt;/dependency&gt;
	&lt;/dependencies&gt;

	&lt;build&gt;
		&lt;sourceDirectory&gt;src&lt;/sourceDirectory&gt;
		&lt;plugins&gt;
			&lt;plugin&gt;
				&lt;artifactId&gt;maven-compiler-plugin&lt;/artifactId&gt;
				&lt;version&gt;3.1&lt;/version&gt;
				&lt;configuration&gt;
					&lt;source&gt;1.8&lt;/source&gt;
					&lt;target&gt;1.8&lt;/target&gt;
				&lt;/configuration&gt;
			&lt;/plugin&gt;
		&lt;/plugins&gt;
	&lt;/build&gt;

&lt;/project&gt;
</code></pre>
你可能注意到了`groupId:artifactId:version`这样的组合。  
`groupId:artifactId:version`通常被理解为Maven中的坐标，Maven以此来标记一个唯一的组件，比如`org.springframework:spring-context:4.1.6.RELEASE`将定位到 spring-context-4.1.6.RELEASE.jar，maven使用坐标主要是为了仓库和依赖管理，因为第三方的jar包实在太多，使用坐标将非常方便地帮助我们找到需要的组件。  
其中groupId为某组织机构的项目名称（或者干脆就是项目名），artifactId为项目下模块的名称/标识，version为组件的版本。  
我们自己的项目也需要配置`groupId:artifactId:version`。  
其他更多pom.xml的选项可以参考https://gist.github.com/greycode/ca9775743f7e4ef827c1 。  
#####插件
Maven大多数的功能都是通过插件来完成的，包括我们之前用到的`mvn compile`命令。
每个插件都包含一些“目标（goal）”，所以使用插件是通常是下面的命令语法：
<pre><code class="dos">mvn [插件名]:[目标]
</code></pre>
比如几个常见的：
<pre><code class="dos"> mvn compiler:compile
 mvn surefire:test
 mvn jar:jar
</code></pre>
注意上面的`mvn compiler:compile`，其实这个才是`mvn compile`的完全体，我们可以直接使用`mvn compile`是因为maven内置了一些插件和生命周期的绑定。
插件的配置需要在`<plugins>`节点下。

#####构建生命周期
Maven内部有三套独立的生命周期，分别为clean、default和site。clean生命周期的主要目的是清理项目，default是构建项目，而site是建立项目站点。  
生命周期包含一些阶段（phase），而每个生命周期都是顺序按照这些阶段来执行的。  
clean包含了pre-clean、clean、post-clean三个阶段。  
site包含了pre-site、site、post-site和site-deploy。  
而default生命周期的阶段则定义了构建项目时主要的步骤：

>  1. validate
 2. generate-sources
 3.  process-sources 
 4. generate-resources
 5. process-resources
 处理项目主目录的资源文件。主要是对`src/main/resources`文件夹下的内容进行一些处理并输出到`target\classes`下。
 6. compile
 编译项目主代码。目录为`src/main/java`，生成class文件到`target\classes`下。
 7. process-test-sources
 8. process-test-resources
 处理项目测试目录的资源文件。主要是对`src/test/resources`文件夹下的内容进行一些处理并输出到`target\test-classes`下。
 9. test-compile
 编译项目测试代码。目录为`src/test/java`，生成class文件到`target\test-classes`。
 10. test
 使用测试框架测试代码。
 11. package
 对编译好的代码进行打包，如jar、、war。
 12. install
 将包安装到maven本地库。
 13. deploy
 将包安装到maven远程库。

以上并不是完整的default生命周期阶段，详细请参考官方文档。  

你会发现在`mvn` 接上面default生命周期的阶段名都会执行相应的操作，另外执行某个命令比如`mvn compile`时将先执行`compile`的前置阶段（上面的 1-5）再执行compile。  
因此当运行
<pre><code class="dos">mvn clean install
</code></pre>
其实是先执行clean生命周期的pre-clean、clean，然后执行上面default生命周期的1-12。

当然平常项目开发中，我们最常用的还是`mvn compile` 和 `mvn test`，最多也就执行到`package`。

#####依赖
项目开发会用到很多第三方jar包，如果将所有的jar（编译用的，测试用的，运行时必须用的）全放到lib目录下很不明智，比如至少得区别开测试用的jar包吧，没必要把一个junit-4.12.jar打包放到运行环境中。  
而且某个jar包 A.1可能会依赖于其他的第三方jar包同B.1，B.1又依赖于C.1，我们的项目中可能有了C.2。  
所以手动新增一个jar时需要同时添加这个jar嵌套依赖的所有jar，同时还要考虑版本冲突的问题以及是否运行时部署的问题，是不是很麻烦？  

所以需要Maven这样的工具来帮助我们自动解决依赖的问题。  

对于依赖的jar包需要在`<dependencies>`节点下声明， 如一个项目的pom.xml中：
<pre><code class="xml">&lt;dependencies&gt;
    
    &lt;dependency&gt;
        &lt;groupId&gt;org.springframework&lt;/groupId&gt;
        &lt;artifactId&gt;spring-context&lt;/artifactId&gt;
        &lt;version&gt;4.1.6.RELEASE&lt;/version&gt;
    &lt;/dependency&gt;
    
    &lt;!-- ...其他依赖项...  --&gt;
&lt;/dependencies&gt;
</code></pre>
当执行`mvn compile`时，maven 会首先在`<localRepository>`配置的本地库的`org\springframework\spring-context\4.1.6.RELEASE`目录中查找该jar，没找到的话再从配置的远程库（默认为Maven官方的中央库）下载到本地库的上面的位置，再将该jar加入到编译路径中完成编译。

上面的命令执行时不只是下载了spring-context-4.1.6.RELEASE.jar一个包，而是所有依赖的以及嵌套依赖的包都会采用同样的方式添加到编译路径参与编译。


###最佳实践
[TBD]



  [1]: http://central.sonatype.org/
  [2]: http://maven.oschina.net/help.html