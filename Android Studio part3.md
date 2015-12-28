# Gradle构建项目及打包

---

# 1. 简介

Gradle是一种依赖管理工具，基于Groovy语言，面向Java应用为主，它抛弃了基于XML的各种繁琐配置，取而代之的是一种基于Groovy的内部领域特定（DSL）语言。  
Gradle是一个高级构建系统和构建工具，允许通过插件自定义构建逻辑，使得代码和资源的重用更加简单。  
Gradle项目通过项目根目录下的 build.gradle 文件来描述构建过程

# 2.Gradle基础

项目中和Gradle相关的几个文件

![](http://i.imgur.com/9kH8Z0I.png)

## 文件joke-android/joke/build.gradle

```  
//声明Android程序，最新gradle的写法apply plugin: 'com.android.application'  
apply plugin: 'android'

dependencies {  
	//编译libs目录下的所有jar   
    compile fileTree(include: '*.jar', dir: 'libs')   
    compile('com.feibo:fbcore:1.1.3') {  
        exclude module: 'support-v4', group: 'com.android.support'  
        exclude module: 'support-annotations', group: 'com.android.support'  
        exclude module: 'gson', group: 'com.google.code.gson'  
    }  
    compile('com.feibo:social-component:1.0.5') {  
        exclude module: 'support-v4', group: 'com.android.support'  
        exclude module: 'support-annotations', group: 'com.android.support'  
        exclude module: 'gson', group: 'com.google.code.gson'  
    }  
    compile 'com.google.code.gson:gson:2.2.4'  
    compile 'com.android.support:appcompat-v7:22.0.0'  
    compile 'com.android.support:recyclerview-v7:22.2.0'  
    compile 'in.srain.cube:ultra-ptr:1.0.11'  
    compile 'com.pnikosis:materialish-progress:1.7'  
    compile 'com.squareup.picasso:picasso:2.5.2'  
	//编译所依赖的模块  
    compile project(':autolayout')  

}

android {  
	//编译SDK的版本  
    compileSdkVersion 21  
	//build tools的版本  
    buildToolsVersion '21.0.1'

    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
            jniLibs.srcDirs = ['libs']
        }

        // Move the tests to tests/java, tests/res, etc...
        instrumentTest.setRoot('tests')

        // Move the build types to build-types/<type>
        // For instance, build-types/debug/java, build-types/debug/AndroidManifest.xml, ...
        // This moves them out of them default location under src/<type>/... which would
        // conflict with src/ being used by the main source set.
        // Adding new build types or product flavors should be accompanied
        // by a similar customization.
        debug.setRoot('build-types/debug')
        release.setRoot('build-types/release')
    }

    defaultConfig {  
		//有用的包名  
        applicationId "com.feibo.joke"
        minSdkVersion 14
        targetSdkVersion 22
        versionCode 45
        versionName "4.0.1"
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "feibo"] //默认渠道号
    }  
	//签名配置  
    signingConfigs {
        debug {
            storeFile file(KEYSTORE)
            storePassword STORE_PASSWORD
            keyAlias KEYALIAS
            keyPassword KEY_PASSWORD
        }
        myConfig {
            storeFile file(KEYSTORE)
            storePassword STORE_PASSWORD
            keyAlias KEYALIAS
            keyPassword KEY_PASSWORD
        }
    }

    buildTypes {
        release {
            minifyEnabled proguardOn()  
			//混淆文件位置  
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-project.txt'
            signingConfig signingConfigs.myConfig
        }
    }  
	//移除lint检查的error  
    lintOptions{
        abortOnError false
    }

//    flavors()
}

def flavors() { // 多渠道打包
    if (market_channels.trim().length() == 0) {
        return
    }

    String[] channels = market_channels.split(",")

    if (channels == null || channels.length == 0) {
        return
    }

//    List<String> channels_no = new ArrayList<String>();
//
//    int start = 100100;
//    for (int i = 0; i < 30; i++) {
//        channels_no.add("fb" + (start + i));
//    }
//    String[] channels = channels_no.toArray(new String[0]);
    channels.each {
        channel ->
            android.productFlavors.create(channel, {
                manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
            })
    }
}

def proguardOn() {
    return "true".equals(proguard_on)
}

tasks.withType(JavaCompile){
    options.encoding = "UTF-8"
}

```  

* 文件joke-android/autolayout/build.gradle,autolayout作为一个库被joke依赖

```  
apply plugin: 'com.android.library'  
//其他与别的Module一样
```

## 目录joke-android/gradle

![](http://i.imgur.com/iATEBWh.png)  


gradle-wrapper.properties 这个文件的内容

```  
distributionBase=GRADLE_USER_HOME  
distributionPath=wrapper/dists  
zipStoreBase=GRADLE_USER_HOME  
zipStorePath=wrapper/dists  
distributionUrl=http\://services.gradle.org/  distributions/gradle-2.2.1-all.zip  

```  
可以看到里面声明了gradle的目录与下载路径以及当前项目使用的gradle版本  

## 文件joke-android/build.gradle

该文件是整个项目的gradle基础配置文件

```  
buildscript {  
    repositories {  
        mavenCentral()  
        jcenter()  
        maven {  
            url 'http://192.168.45.248:8081/nexus/content/groups/public'  
        }  
    }  
    dependencies {  
		//声明gradle plugin的版本  
        classpath 'com.android.tools.build:gradle:1.2.3'  
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.0'  
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3'  
    }  
}  

allprojects {  
    repositories {  
        mavenCentral()  
		//jccenter为中央远程仓库  
        jcenter()  
        maven {  
            url 'http://192.168.45.248:8081/nexus/content/groups/public'  
        }  
    }  
}   
```  

## 文件joke-android/setting.gradle
  
这个文件是全局的项目配置文件，里面主要声明一些需要加入gradle的module

```
include ':joke', ':autolayout'
```

# 3.Gradle命令

## gradlew -v 来查看下项目所用的Gradle版本

下载成功后会输出一下信息

![](http://i.imgur.com/Y5qLKnS.png)

## gradlew clean 清除project/app目录下的build文件夹，下载gradle的一些依赖并进行编译  

![](http://i.imgur.com/WEFonKt.png)

## gradlew build 检查依赖并编译打包

![](http://i.imgur.com/YuiAyTq.png)

## gradlew assemble
assembleDebug 打Debug包；assembleDebug 打Release包

# 4.多渠道打包

由于国内Android市场众多渠道，为了统计每个渠道的下载及其它数据统计，就需要我们针对每个渠道单独打包，如果让你打几十个市场的包岂不烦死了，不过有了Gradle，这再也不是事了。

## 友盟多渠道打包

以友盟统计为例，在AndroidManifest.xml中会有：

```  
<meta-data  
    android:name="UMENG_CHANNEL"  
    android:value="Channel_ID" />  
```  
  
这里面的Chanael_ID就是渠道标识，我们的目标就是在编译的时候这个值能够自动变化。

* step1:在AndroidMainfest.xml中配置PlaceHolder  

```  
<meta-data  
    android:name="UMENG_CHANNEL"  
    android:value="${UMENG_CHANNEL_VALUE}" />  
```   

* step2:在build.gradle设置productFlavors

```  
android{  
	dafaultConfig{  
		//默认渠道号  
		mainfestPlaceholders=[UMENG_CHANNEL_VALUE:"feibo"]
	}  

def flavors() { // 多渠道打包   
    if (market_channels.trim().length() == 0) {  
        return  
    }  
    String[] channels = market_channels.split(",")  
    if (channels == null || channels.length == 0) {  
        return  
    }  

	channels.each {  
        channel ->  
            android.productFlavors.create(channel, {  
                manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]  
            })  
    } 
} 
   
```

> Tip: 在配置文件gradle.properties中可以看到所有渠道名 market_channels=feibo,qihoo,QQyingyongbao,baidu,baidu91,wandoujia...

* step3 :命令gradlew assembleDebug/gradlew assenbleRelease
这样就可以生成所有渠道的包了

> Tip:如果只想打某一渠道的release版本，执行gradlew assembleWandoujiaRelease

通常情况下apps发布后也就是release模式下log是不显示的，debug模式下是显示log的，但是在特殊情况下我们测试release包的时候需要log的时候，就无法使用BuildConfig.DEBUG来达到要求，因为在release模式下自动设置为false，debug模式下是true，这个时候我们需要自定义可控制的log开关。  

在/app/build/generated/source/buildConfig/ 文件下的产品目录里面，找到想要的包名下会自动生成BuildConfig.java文件。我们可以看看下release模式下该文件的内容：

```  
package com.feibo.joke;  

public final class BuildConfig {  
  public static final boolean DEBUG = false;  
  public static final String APPLICATION_ID = "com.feibo.joke";  
  public static final String BUILD_TYPE = "release";  
  public static final String FLAVOR = "3G_market";  
  public static final int VERSION_CODE = 45;  
  public static final String VERSION_NAME = "4.0.3";  
}  
```

在app/Constant.java文件

```    
package com.feibo.joke.app;  
import com.feibo.joke.BuildConfig;  
public class Constant {  
	public static final boolean DEBUG = BuildConfig.DEBUG;  

	public static final boolean TRACE = BuildConfig.DEBUG;   

	public static final boolean USE_REAL_SEVER = false;  

```

在build.gradle里配置如下：

```  
buildTypes {  

        dubug {  
            //显示log  
            buildConfigField "boolean","LOG_DEBUG","true"  
            buildConfigField "boolean","USE_REAL_SEVER",   "false"  
            signingConfig signingConfigs.debug  

        }  

        release {  
            //不显示log  
            buildConfigField "boolean","LOG_DEBUG", "false"  
            buildConfigField "boolean","USE_REAL_SEVER", "true"  
            minifyEnabled proguardOn()  
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-project.txt'  
            signingConfig signingConfigs.myConfig  
        }  

```

## 美团打渠道包方案

* step1：我们直接修改apk的渠道号，直接解压apk,解压后的根目录会有一个META-INF目录：  

![](http://i.imgur.com/uO3Sqs7.png)

* step2：在META-INF目录下添加空文件，可以不用重新签名应用，因此可以为不同渠道的应用添加不同的空文件，可以唯一标识一个渠道。用python代码给apk添加空的渠道文件，渠道名的前缀machannel_:

``` 
import zipfile  
zipped = zipfile.ZipFile(your_apk, 'a',   zipfile.ZIP_DEFLATED)   
empty_channel_file = "META-INF/mtchannel_{channel}".format(channel=your_channel)  
zipped.write(your_empty_file, empty_channel_file)  

```
添加完空渠道文件后，在META-INF目录下回多出一个mtchannnel_meituan的空文件

* step3：接下来在java代码中读取空渠道文件名：

```java  
public static String getChannel(Context context) {  
        ApplicationInfo appinfo = context.getApplicationInfo();  
        String sourceDir = appinfo.sourceDir;  
        String ret = "";  
        ZipFile zipfile = null;  
        try {  
            zipfile = new ZipFile(sourceDir);  
            Enumeration<?> entries = zipfile.entries();  
            while (entries.hasMoreElements()) {  
                ZipEntry entry = ((ZipEntry)   entries.nextElement());  
                String entryName = entry.getName();  
                if (entryName.startsWith("mtchannel")) {  
                    ret = entryName;  
                    break;  
                }  
            }  
        } catch (IOException e) {  
            e.printStackTrace();  
        } finally {  
            if (zipfile != null) {  
                try {  
                    zipfile.close();  
                } catch (IOException e) {  
                    e.printStackTrace();  
                }  
            }  
        }  

        String[] split = ret.split("_");  
        if (split != null && split.length >= 2) {  
            return ret.substring(split[0].length() + 1);  

        } else {  
            return "";  
        }  
    }  

```

这样，每打一个渠道包只需复制一个apk，在META-INF中添加一个使用渠道号命名的空文件即可,这种打包方式速度非常快.






