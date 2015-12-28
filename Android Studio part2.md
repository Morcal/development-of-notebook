# Android Studio工具条
---
# 1. AndroidStudio项目查看模式

![](http://i.imgur.com/ZQ8Fa5A.png)

## 1.1 Project

展示项目的全部文件信息

![](http://i.imgur.com/h07bENg.png)

* app/build/ app模块build编译输出的目录    
* app/build.gradle app模块的gradle编译文件    
* app/app.iml app模块的配置文件  
* app/proguard-rules.pro app模块proguard文件  
* build.gradle 项目的gradle编译文件  
* settings.gradle 定义项目包含哪些模块  
* gradlew 编译脚本，可以在命令行执行打包  
* local.properties 配置SDK/NDK  
* MyApplication.iml 项目的配置文件  
* External Libraries 项目依赖的Lib, 编译时自动下载的

## 1.2 Packages

只展示Module相关的代码和资源文件

![](http://i.imgur.com/9xjBDFB.png)

## 1.3 Android

这种模式将个文件通过类型归类，如：java文件、资源文件、manifests文件

![](http://i.imgur.com/pfU9TqM.png)

* app/manifests AndroidManifest.xml配置文件目录  
* app/java 源码目录  
* app/res 资源文件目录  
* Gradle Scripts gradle编译相关的脚本  

## 1.4 Project Files

这种模式显示更像Eclipse中的目录结构，可展示多个Project

![](http://i.imgur.com/sX7FZIn.png)

# 2. Android Studio导入jar包和第三方开源库

## 2.1 导入jar

jar：只包含class文件与清单文件，不包含资源文件。  
在项目的libs中放入要导入的jar包，然后右键jar文件，选择“add as a library”即可在项目中使用这个该jar包

> Tip: 如果在gradle中已有`compile fileTree(include: ['*.jar'], dir: 'libs')`,就再不用add,直接同步一些gradle即可。

## 2.2 导入aar

aar:包含jar包和资源文件，如图片等所有res中的文件。  
*.aar有有两种方式，分别为本地加载以及网络加载，网络加载就会涉及到发布到mavenCentral托管的问题；另一种本地加载的方式，简单快捷。

* step1：拷贝到genius.aar到libs目录  
* step2：build.gradle 配置文件中更改为

```
	repositories {  
    flatDir {  
        dirs 'libs'  
    }  
	}  
	dependencies {  
    compile(name:'genius', ext:'aar')  
	}  
```

## 2.3导入第三方开源库

### 2.3.1 添加远程开源库

无需将开源库下载下来，在项目目录（一般是 app 目录），编辑 build.gradle 文件。

* 一般会在README.md提示，如：

![](http://i.imgur.com/AFEnMrT.png)

* 在build.gradle中添加

![](http://i.imgur.com/vh6RF6v.png)

* sync同步gradle

### 2.3.2 添加本地开源库

将开源库下载下来，放置在与app目录同级的目录下，然后编辑 setting.gradle 文件。

* Import Module->选择已下载的本地开源库

![](http://i.imgur.com/UyrpzbC.png)

* 修改setting.gradle添加本地库

![](http://i.imgur.com/Ncy7Ggq.png)

* 在app目录下的build.gradle文件，在 dependencies { }节点下加入：`compile project(':autolayout')`

![](http://i.imgur.com/nII08Eb.png)

* sync同步gradle



