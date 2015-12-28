# Gradle 的 android插件的使用
标签（空格分隔）：gradle android  
---
[Android Gradle实战中遇到的问题与经验][1]
gradle:
gradle遵循约定优先于配置
gradle属于任务驱动型构建工具，它的构建过程是基于Task
## [一种] Gradle Scripts：
build.gradle:文件,项目与模块
setting.gradle:项目配置
proguard-rules.pro / proguard-project.txt:项目与模块的混淆规则
gradle.properties:项目属性
local.properties:SDK 位置
---
### [settings.gradle]：多模块项目构建
在项目根目录下创建该配置文件,导入各个子module,还可以设置一些函数。这些函数会在gradle构建整个工程任务的时候执行，所以，可以在settings做一些初始化的工作。
普遍:
`include ':yizhong', ':fbpay'`
> ':module name'， ':module name'   
带初始化的:
```
//定义一个名为initMinshengGradleEnvironment的函数。该函数内部完成一些初始化操作
//比如创建特定的目录，设置特定的参数等
def initMinshengGradleEnvironment(){
    println"initialize Minsheng Gradle Environment ....."
    ......//干一些special的私活....
    println"initialize Minsheng Gradle Environment completes..."
}
//settings.gradle加载的时候，会执行initMinshengGradleEnvironment
initMinshengGradleEnvironment()
//include也是一个函数：
include 'CPosSystemSdk' , 'CPosDeviceSdk' ,
      'CPosSdkDemo','CPosDeviceServerApk','CPosSystemSdkWizarPosImpl'
```
---
### [builde.gradle]:构建脚本
* Project:
配置其他子project,比如添加一些属性
```
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
    repositories {
        jcenter {		// 获取依赖的仓库,jCenter是大于mavenCentral()的一个仓库，现在是studio默认的仓库
            url "http://jcenter.bintray.com/"	// 用url的形式给出该库的位置,也可以不写
        }
    }
	/* -ANA-
	buildscript:在此处对项目的所有模块进行统一的配置,这里的配置只影响控制构建过程的代码，不影响项目源代码。项目本身需要声明自己的仓库和依赖关系   
	存储仓库:文件集合,由group,name,version组成;gradle支持Maven,Ivy等仓库,支持使用本地文件系统或http访问
	一个项目可以有多个存储库,gradle按照指定顺序在每个库中查找依赖,最先在某个库中找到就停止
	
	依赖分组:编译时和运行时使用不同的依赖分组,每一组依赖称为一个Configuration.
	Configuration定义:默认是classpath
	
	configurations {
		myDependency
	}
	
	dependencies()添加myDependency里的依赖项:
	dependencies {
		myDependency 'org.apache.commons:commons-lang3:3.0'
	}
	dependencies {
		classpath 'com.android.tools.build:gradle:1.3.0'
	}
	dependencies {
		compile project(':library')     // 对其他Project或文件系统的依赖,此处library是另一个module的名字
        compile fileTree(dir: 'libs', include: ['*.jar'])   // 对本地文件系统中Jar文件依赖
		compile 'com.android.support:recyclerview-v7:22.2.1'
		compile 'com.android.support:design:22.2.1'
		compile 'com.evernote:android-intent:1.0.1'
		testCompile 'junit:junit:4.8.2' 
	}
	! myDependency，classpath，compile，testCompile都是Configuration（一组依赖）
    android Plugin会自动定义compile和testCompile分别用于编译Java源文件和编译Java测试源文件。
	classpath应该是用于所有，我类推的。
	
	compile：编译源码需要的依赖。
	外部依赖: compile group:'',naem:'',version:''->简写compile 'group:name:version'
	
    runtime：在运行时生成的类使用的依赖。默认情况下，还包括编译时的依赖。
    testCompile：编译该项目的测试源码所需要的依赖。默认情况下，还包括编译产生类和编译时的依赖。
    testRuntime：运行测试需要的依赖。默认情况下，还包括编译，运行和测试编译依赖。
    
    使用依赖来发布文件到远程仓库Ivy或Maven,以下是发布到Maven,task uploadArchives,使用Maven插件,gradle生成并上传pom.xml
   
    apply plugin: 'maven'
    uploadArchives {
        repositories {
            mavenDeployer {
                repository(url: "file://localhost/tmp/myRepo/")
        }
    }
}
    
	-ANA-*/
	
    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.0'
        classpath 'me.tatarka:gradle-retrolambda:3.2.0'
    }
}
allprojects {
    repositories {
        jcenter()
        mavenCentral()  //PhotoView、roundedimageview配置
        maven {     // 远程Maven仓库
            url 'http://192.168.45.248:8081/nexus/content/groups/public'
        }
    }
}
/* -ANA-
allprojects将repositories配置一次性地应用于所有的module（子Project）和root-project本身，当然也包括定义的Task，这个task配置到所有module和root-project。
    allprojects {
        repositories {
            jcenter()
        }
        //通常studio项目没有，咱自己加的
       apply plugin: 'idea'
       task allTask << {
          println project.name
       }
    }
    
 同理，subprojects()方法用于配置所有的子Project（不包含根Project）   
-ANA- */
```
* Module：fbpay    
```
apply plugin: 'com.android.library' // 表明这是一个android项目,为Project引入多个Task和Property
android {
    compileSdkVersion 22
    buildToolsVersion "21.1.2"
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
/*ANA
andorid{...}配置了所有android构建过程需要的参数。这里也是Android DSL的入口点。
默认情况下，只需要配置目标编译SDK版本和编译工具版本，即compileSdkVersion和buildToolsVersion属性。
这个complieSdkVersion属性相当于旧构建系统中project.properites文件中的target属性。这个新的属性可以跟旧的target属性一样指定一个int或者String类型的值。
srcDir将会被添加到指定的已存在的源文件夹中
替换默认的源代码文件夹，你可能想要使用能够传入一个路径数组的srcDirs来替换单一的srcDir。以下是使用调用对象的另一种不同方法：
    sourceSets {
        main.java.srcDirs = ['src/java']
        main.resources.srcDirs = ['src/resources']
    }
    
将旧构建系统项目迁移到新构建系统需要做的迁移工作:使用了旧项目结构中的main源码，将这些sourceSet组件重新映射到src目录下,将androidTest sourceSet组件重新映射到tests文件夹,旧=新
    android {
        sourceSets {
            main {
                manifest.srcFile 'AndroidManifest.xml'
                java.srcDirs = ['src']
                resources.srcDirs = ['src']
                aidl.srcDirs = ['src']
                renderscript.srcDirs = ['src']
                res.srcDirs = ['res']
                assets.srcDirs = ['assets']
            }
            androidTest.setRoot('tests')
        }
    }
ANA*/
    defaultConfig {
        minSdkVersion 11
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
def proguardOn() {
    return "true".equals(proguard_on)   // 这个东西在文件gradle.properties
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.2.0'
//    compile files('libs/alipaySDK-20150602.jar')
//    compile files('libs/libammsdk.jar')
//    compile files('libs/UPPayAssistEx.jar')
//    compile files('libs/UPPayPluginEx.jar')
    compile('com.feibo:social-component:1.0.5') {
        exclude module: 'support-v4', group: 'com.android.support'
        exclude module: 'support-annotations', group: 'com.android.support'
        exclude module: 'gson', group: 'com.google.code.gson'
    }
}
```
* Module：yizhong
```
apply plugin: 'com.android.application'
apply plugin: 'me.tatarka.retrolambda'
retrolambda { //lambda配置
    jdk System.getenv("JAVA8_HOME") //java8的sdk目录
    oldJdk System.getenv("JAVA_HOME") //默认java的sdk目录
    javaVersion JavaVersion.VERSION_1_6
}
android {
    compileSdkVersion 22
    buildToolsVersion "21.1.2"
    sourceSets {
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
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
//    dexOptions {//打开dex增量编译,项目主Module下的build.grade中加入
//        incremental true
//    jumboMode = true  //大型项目dex方法超过超过65500会报错则开启
//    }
    defaultConfig {
        applicationId "com.feibo.yizhong"
        minSdkVersion 11
        targetSdkVersion 22
        versionCode 9
        versionName "2.2"
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "feibo"] //默认渠道号
    }
    signingConfigs {
        debug {
        }
        myConfig {
            storeFile file("yizhong.keystore")
            storePassword "fbandroid"
            keyAlias "yizhong"
            keyPassword "fbandroid"
        }
    }
    buildTypes {
        release {
            minifyEnabled proguardOn()
//            shrinkResources true//proguardOn()//删除无效的Resource,必须和minifyEnabled一起用;对于反射才获取资源会引起不必要BUG
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-project.txt'
            signingConfig signingConfigs.myConfig
        }
    }
    lintOptions{
        abortOnError false
    }
   flavors()
}
/*多渠道打包:
market_channels变量定义在gradle.properties，
*/ 
def flavors() {     
    if (market_channels.trim().length() == 0) {
        return
    }
/*正式发布：market_channels=feibo,qihoo,QQyingyongbao,baidu,baidu91,wandoujia,appchina,xiaomi,lenovo,huawei,meizu,jinli,oppo,ktouch,samsung,jifang,Nduoa,anzhi,hiapk,eoemarket,market3g,uc,sogou,taobao,mumayi,suning,netease,renren,telecom,unicom,mobileMM,googleplay,hao123,kuchuan,mopo,wangxun,mogu,atongyingyong,xiaoenai,camera360,poco,wannianli,chelun,memeda,baofeng,baidutv,pptv,letv,vxiazai
*/ 
    String[] channels = market_channels.split(",")
    if (channels == null || channels.length == 0) {
        return
    }
/*
    这个东西：
    int size = channels.length;
    List<String> channels_no = new ArrayList<String>();
    channels_no.addAll(channels);
    int start = 100100;
    for (int i = 0; i < 30; i++) {
        channels_no.add("fb" + (start + i));
    }
    channels = channels_no.toArray(new String[0]);
*/
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
 dependencies {
     compile fileTree(include: ['*.jar'], dir: 'libs')
     compile 'com.android.support:support-annotations:22.2.0'
     compile('com.android.support:recyclerview-v7:22.2.0') {
         exclude module: 'support-annotations', group: 'com.android.support'
     }
     compile('com.feibo:fbcore:1.1.3') {
         exclude module: 'support-v4', group: 'com.android.support'
         exclude module: 'support-annotations', group: 'com.android.support'
         exclude module: 'gson', group: 'com.google.code.gson'
     }
//     compile('com.feibo:social-component:1.0.5') {
//         exclude module: 'support-v4', group: 'com.android.support'
//         exclude module: 'support-annotations', group: 'com.android.support'
//         exclude module: 'gson', group: 'com.google.code.gson'
//     }//fbpay已引用，因为微信支付和分享的jar包libammsdk冲突
     compile 'com.nineoldandroids:library:2.4.0'
//     compile 'com.android.support:appcompat-v7:22.2.0'//fbpay已引用
     compile 'com.google.code.gson:gson:2.2.4'
     compile 'com.squareup.okhttp:okhttp:2.4.0'
     compile 'com.squareup.retrofit:retrofit:2.0.0-beta2'
     compile 'com.android.support:design:22.2.0'
     //compile 'com.facebook.rebound:rebound:0.3.8'
     compile 'in.srain.cube:ultra-ptr:1.0.10'
     compile 'com.squareup.picasso:picasso:2.5.2'
     compile 'com.github.chrisbanes.photoview:library:1.2.3'
     compile 'com.makeramen:roundedimageview:2.2.0'
     compile 'me.iwf.photopicker:PhotoPicker:0.2.7@aar'
     compile 'com.github.bumptech.glide:glide:3.6.0'
     /*compile 'io.reactivex:rxjava:1.0.14'
         compile 'io.reactivex:rxandroid:1.0.1'*/
     compile project(':fbpay')
 }
tasks.withType(JavaCompile){
    options.encoding = "UTF-8"
}
```
### [properties]:属性配置
* gradle.properties--[Project Properties]
```
market_channels= #这个值在正式打包应该会填充，是app发布的渠道
STORE_PASSWORD=fbandroid
KEYSTORE=yizhong.keystore
KEYALIAS=yizhong
KEY_PASSWORD=fbandroid
proguard_on=true
```
* local.properties--[SDK Location]
```
sdk.dir=E\:\\Program Files (x86)\\android-sdk-studio
#相当于设置ANDROID_HOME环境变量
```
---
## demo分析--多项目
* utils.gradle:手动配置，添加一些常见函数
```
import groovy.util.XmlSlurper  //解析XML时候要引入这个groovy的package
 
def copyFile(String srcFile,dstFile){
     ......//拷贝文件函数，用于将最后的生成物拷贝到指定的目录
}
def rmFile(String targetFile){
    .....//删除指定目录中的文件
}
 
def cleanOutput(boolean bJar = true){
    ....//clean的时候清理
}
 
def copyOutput(boolean bJar = true){
    ....//copyOutput内部会调用copyFile完成一次build的产出物拷贝
}
 
def getVersionNameAdvanced(){//老朋友
   defxmlFile = project.file("AndroidManifest.xml")
   defrootManifest = new XmlSlurper().parse(xmlFile)
   returnrootManifest['@android:versionName']  
}
 
//对于android library编译，我会disable所有的debug编译任务
def disableDebugBuild(){
  //project.tasks包含了所有的tasks，下面的findAll是寻找那些名字中带debug的Task。
  //返回值保存到targetTasks容器中
  def targetTasks = project.tasks.findAll{task ->
     task.name.contains("Debug")
  }
  //对满足条件的task，设置它为disable。如此这般，这个Task就不会被执行
 targetTasks.each{
     println"disable debug task  :${it.name}"
    it.setEnabled false
  }
}
//将函数设置为extra属性中去，这样，加载utils.gradle的Project就能调用此文件中定义的函数了
ext{
    copyFile= this.&copyFile
    rmFile =this.&rmFile
   cleanOutput = this.&cleanOutput
   copyOutput = this.&copyOutput
   getVersionNameAdvanced = this.&getVersionNameAdvanced
   disableDebugBuild = this.&disableDebugBuild
}
```
* settings.gradle:调用include把需要包含的子Project加进来
```
/*我们团队内部建立的编译环境初始化函数
  这个函数的目的是
  1  解析一个名为local.properties的文件，读取AndroidSDK和NDK的路径
  2  获取最终产出物目录的路径。这样，编译完的apk或者jar包将拷贝到这个最终产出物目录中
  3 获取Android SDK指定编译的版本
*/
def initMinshengGradleEnvironment(){
    println"initialize Minsheng Gradle Environment ....."
   Properties properties = new Properties()
   //local.properites也放在posdevice目录下
    FilepropertyFile = new File(rootDir.getAbsolutePath()+ "/local.properties")
   properties.load(propertyFile.newDataInputStream())
    /*
      根据Project、Gradle生命周期的介绍，settings对象的创建位于具体Project创建之前
      而Gradle底对象已经创建好了。所以，我们把local.properties的信息读出来后，通过
     extra属性的方式设置到gradle对象中
      而具体Project在执行的时候，就可以直接从gradle对象中得到这些属性了！
    */
    gradle.ext.api =properties.getProperty('sdk.api')
    gradle.ext.sdkDir =properties.getProperty('sdk.dir')
     gradle.ext.ndkDir =properties.getProperty('ndk.dir')
     gradle.ext.localDir =properties.getProperty('local.dir')
    //指定debugkeystore文件的位置，debug版apk签名的时候会用到
    gradle.ext.debugKeystore= properties.getProperty('debug.keystore')
     ......
    println"initialize Minsheng Gradle Environment completes..."
}
//初始化
initMinshengGradleEnvironment()
//添加子Project信息
include 'CPosSystemSdk' , 'CPosDeviceSdk' ,'CPosSdkDemo','CPosDeviceServerApk', 'CPosSystemSdkWizarPosImpl'
```
* local.properties:对于Android来说，local.properties文件是必须的
sdk.dir和ndk.dir是Android Gradle必须要指定的，其他都是自定义属性。当然。不编译ndk，就不需要ndk.dir属性了
```
local.dir=/home/innost/workspace/minsheng-flat-dir/
//注意，根据Android Gradle的规范，只有下面两个属性是必须的，其余都是我自己加的
sdk.dir=/home/innost/workspace/android-aosp-sdk/
ndk.dir=/home/innost/workspace/android-aosp-ndk/
debug.keystore=/home/innost/workspace/tools/mykeystore.jks
sdk.api=android-19
```
设置sdk.dir相当于设置ANDROID_HOME环境变量
*  posdevicebuild.gradle:作为multi-project根目录，一般情况下，它的build.gradle是做一些全局配置
```
//下面这个subprojects{}就是一个Script Block
subprojects {
  println"Configure for $project.name" //遍历子Project，project变量对应每个子Project
  buildscript {  //这也是一个SB
    repositories {//repositories是一个SB
       ///jcenter是一个函数，表示编译过程中依赖的库，所需的插件可以在jcenter仓库中
       //下载。
       jcenter()
    }
    dependencies { //SB
        //dependencies表示我们编译的时候，依赖android开发的gradle插件。插件对应的
       //class path是com.android.tools.build。版本是1.2.3
        classpath'com.android.tools.build:gradle:1.2.3'
    }
   //为每个子Project加载utils.gradle 。当然，这句话可以放到buildscript花括号之后
   applyfrom: rootProject.getRootDir().getAbsolutePath() + "/utils.gradle"
 }//buildscript结束
}
```
* CPosDeviceSdkbuild.gradle
一个Android Library。按Google的想法，Android Library编译出来的应该是一个AAR文件。但是我的项目有些特殊，我需要发布CPosDeviceSdk.jar包给其他人使用。jar在编译过程中会生成，但是它不属于Android Library的标准输出。在这种情况下，我需要在编译完成后，主动copy jar包到我自己设计的产出物目录中。
```
//Library工程必须加载此插件。注意，加载了Android插件就不要加载Java插件了。因为Android
//插件本身就是拓展了Java插件
apply plugin: 'com.android.library' 
//android的编译，增加了一种新类型的ScriptBlock-->android
android {
       //你看，我在local.properties中设置的API版本号，就可以一次设置，多个Project使用了
      //借助我特意设计的gradle.ext.api属性
       compileSdkVersion =gradle.api  //这两个红色的参数必须设置
       buildToolsVersion  = "22.0.1"
       sourceSets{ //配置源码路径。这个sourceSets是Java插件引入的
       main{ //main：Android也用了
           manifest.srcFile 'AndroidManifest.xml' //这是一个函数，设置manifest.srcFile
           aidl.srcDirs=['src'] //设置aidl文件的目录
           java.srcDirs=['src'] //设置java文件的目录
        }
     }
   dependencies {  //配置依赖关系
      //compile表示编译和运行时候需要的jar包，fileTree是一个函数，
     //dir:'libs'，表示搜索目录的名称是libs。include:['*.jar']，表示搜索目录下满足*.jar名字的jar
     //包都作为依赖jar文件
       compile fileTree(dir: 'libs', include: ['*.jar'])
   }
}  //android SB配置完了
//clean是一个Task的名字，这个Task好像是Java插件（这里是Android插件）引入的。
//dependsOn是一个函数，下面这句话的意思是 clean任务依赖cposCleanTask任务。所以
//当你gradle clean以执行clean Task的时候，cposCleanTask也会执行
clean.dependsOn 'cposCleanTask'
//创建一个Task，
task cposCleanTask() <<{
    cleanOutput(true)  //cleanOutput是utils.gradle中通过extra属性设置的Closure
}
//前面说了，我要把jar包拷贝到指定的目录。对于Android编译，我一般指定gradle assemble
//它默认编译debug和release两种输出。所以，下面这个段代码表示：
//tasks代表一个Projects中的所有Task，是一个容器。getByName表示找到指定名称的任务。
//我这里要找的assemble任务，然后我通过doLast添加了一个Action。这个Action就是copy
//产出物到我设置的目标目录中去
tasks.getByName("assemble"){
   it.doLast{
       println "$project.name: After assemble, jar libs are copied tolocal repository"
        copyOutput(true)
     }
}
/*
  因为我的项目只提供最终的release编译出来的Jar包给其他人，所以不需要编译debug版的东西
  当Project创建完所有任务的有向图后，我通过afterEvaluate函数设置一个回调Closure。在这个回调
  Closure里，我disable了所有Debug的Task
*/
project.afterEvaluate{
    disableDebugBuild()
}
```
*  CPosDeviceServerApk build.gradle
再来看一个APK的build，它包含NDK的编译，并且还要签名。根据项目的需求，我们只能签debug版的，而release版的签名得发布unsigned包给领导签名。另外，CPosDeviceServerAPK依赖CPosDeviceSdk。
虽然我可以先编译CPosDeviceSdk，得到对应的jar包，然后设置CPosDeviceServerApk直接依赖这个jar包就好。但是我更希望CPosDeviceServerApk能直接依赖于CPosDeviceSdk这个工程。这样，整个posdevice可以做到这几个Project的依赖关系是最新的。
```
apply plugin: 'com.android.application'  //APK编译必须加载这个插件
android {
      compileSdkVersion gradle.api
      buildToolsVersion "22.0.1"
      sourceSets{  //差不多的设置
       main{
           manifest.srcFile 'AndroidManifest.xml'
          //通过设置jni目录为空，我们可不使用apk插件的jni编译功能。为什么？因为据说
         //APK插件的jni功能好像不是很好使....晕菜
          jni.srcDirs = [] 
           jniLibs.srcDir 'libs'
            aidl.srcDirs=['src']
           java.srcDirs=['src']
           res.srcDirs=['res']
        }
    }//main结束
   signingConfigs { //设置签名信息配置
       debug {  //如果我们在local.properties设置使用特殊的keystore，则使用它
           //下面这些设置，无非是函数调用....请务必阅读API文档
           if(project.gradle.debugKeystore != null){
              storeFile file("file://${project.gradle.debugKeystore}")
              storePassword "android"
              keyAlias "androiddebugkey"
              keyPassword "android"
           }
        }
   }//signingConfigs结束
     buildTypes {
       debug {
           signingConfig signingConfigs.debug
           jniDebuggable false
        }
    }//buildTypes结束
   dependencies {
        //compile：project函数可指定依赖multi-project中的某个子project
       compile project(':CPosDeviceSdk')
       compile fileTree(dir: 'libs', include: ['*.jar'])
   } //dependices结束
  repositories{
   flatDir {//flatDir：告诉gradle，编译中依赖的jar包存储在dirs指定的目录
           name "minsheng-gradle-local-repository"
            dirsgradle.LOCAL_JAR_OUT //LOCAL_JAR_OUT是我存放编译出来的jar包的位置
   }
  }//repositories结束
}//android结束
/*
   创建一个Task，类型是Exec，这表明它会执行一个命令。我这里让他执行ndk的
   ndk-build命令，用于编译ndk。关于Exec类型的Task，请自行脑补Gradle的API
*/
//注意此处创建task的方法，是直接{}喔，那么它后面的tasks.withType(JavaCompile)
//设置的依赖关系，还有意义吗？Think！如果你能想明白，gradle掌握也就差不多了
task buildNative(type: Exec, description: 'CompileJNI source via NDK') {
       if(project.gradle.ndkDir == null) //看看有没有指定ndk.dir路径
          println "CANNOT Build NDK"
       else{
            commandLine "/${project.gradle.ndkDir}/ndk-build",
               '-C', file('jni').absolutePath,
               '-j', Runtime.runtime.availableProcessors(),
               'all', 'NDK_DEBUG=0'
        }
  }
 tasks.withType(JavaCompile) {
       compileTask -> compileTask.dependsOn buildNative
  }
  ......  
 //对于APK，除了拷贝APK文件到指定目录外，我还特意为它们加上了自动版本命名的功能
 tasks.getByName("assemble"){
       it.doLast{
       println "$project.name: After assemble, jar libs are copied tolocal repository"
       project.ext.versionName = android.defaultConfig.versionName
       println "\t versionName = $versionName"
       copyOutput(false)
     }
}
```
* 执行阶段  
在posdevice下执行gradle assemble,最终输出文件都会拷贝到指定目录
![编译项目时输出结果展示][2]
library包都编译release版的，copy到xxx/javaLib目录下
apk编译debug和release-unsigned版的，copy到apps目录下
所有产出物都自动从AndroidManifest.xml中提取versionName。
## demo分析--单项目
有三个版本，分别是debug、release和demo。这三个版本对应的代码都完全一样，但是在运行的时候需要从assets/runtime_config文件中读取参数。参数不同，则运行的时候会跳转到debug、release或者demo的逻辑上
在编译build、release和demo版本前，在build.gradle中自动设置runtime_config的内容。代码如下所示：
[build.gradle]
```
apply plugin: 'com.android.application'  //加载APP插件
//加载utils.gradle
apply from:rootProject.getRootDir().getAbsolutePath() + "/utils.gradle"
//buildscript设置脚本依赖,android app插件的位置
buildscript {
   repositories { jcenter() }
   dependencies { classpath 'com.android.tools.build:gradle:1.2.3' }
}
//androidScriptBlock
android {
   compileSdkVersion gradle.api
   buildToolsVersion "22.0.1"
   sourceSets{//源码设置SB
        main{
           manifest.srcFile 'AndroidManifest.xml'
           jni.srcDirs = []
           jniLibs.srcDir 'libs'
           aidl.srcDirs=['src']
           java.srcDirs=['src']
           res.srcDirs=['res']
           assets.srcDirs = ['assets'] //多了一个assets目录
        }
    }
   signingConfigs {//签名设置
       debug {  //debug对应的SB。注意
           if(project.gradle.debugKeystore != null){
               storeFile file("file://${project.gradle.debugKeystore}")
               storePassword "android"
               keyAlias "androiddebugkey"
               keyPassword "android"
           }
        }
    }
    /*
     最关键的内容来了： buildTypesScriptBlock.
     buildTypes和上面的signingConfigs，当我们在build.gradle中通过{}配置它的时候，
     其背后的所代表的对象是NamedDomainObjectContainer<BuildType> 和
     NamedDomainObjectContainer<SigningConfig>
     注意，NamedDomainObjectContainer<BuildType/或者SigningConfig>是一种容器，
     容器的元素是BuildType或者SigningConfig。我们在debug{}要填充BuildType或者
    SigningConfig所包的元素，比如storePassword就是SigningConfig类的成员。而proguardFile等
    是BuildType的成员。
    那么，为什么要使用NamedDomainObjectContainer这种数据结构呢？因为往这种容器里
    添加元素可以采用这样的方法： 比如signingConfig为例
    signingConfig{//这是一个NamedDomainObjectContainer<SigningConfig>
       test1{//新建一个名为test1的SigningConfig元素，然后添加到容器里
         //在这个花括号中设置SigningConfig的成员变量的值
       }
      test2{//新建一个名为test2的SigningConfig元素，然后添加到容器里
         //在这个花括号中设置SigningConfig的成员变量的值
      }
    }
    在buildTypes中，Android默认为这几个NamedDomainObjectContainer添加了
    debug和release对应的对象。如果我们再添加别的名字的东西，那么gradleassemble的时候
    也会编译这个名字的apk出来。比如，我添加一个名为test的buildTypes，那么gradle assemble
    就会编译一个xxx-test-yy.apk。在此，test就好像debug、release一样。
   */
   buildTypes{
        debug{ //修改debug的signingConfig为signingConfig.debug配置
           signingConfig signingConfigs.debug
        }
        demo{ //demo版需要混淆
           proguardFile 'proguard-project.txt'
           signingConfig signingConfigs.debug
        }
       //release版没有设置，所以默认没有签名，没有混淆
    }
      ......//其他和posdevice 类似的处理。来看如何动态生成runtime_config文件
   def  runtime_config_file = 'assets/runtime_config'
   /*
   我们在gradle解析完整个任务之后，找到对应的Task，然后在里边添加一个doFirst Action
   这样能确保编译开始的时候，我们就把runtime_config文件准备好了。
   注意，必须在afterEvaluate里边才能做，否则gradle没有建立完任务有向图，你是找不到
   什么preDebugBuild之类的任务的
   */
   project.afterEvaluate{
      //找到preDebugBuild任务，然后添加一个Action 
      tasks.getByName("preDebugBuild"){
           it.doFirst{
               println "generate debug configuration for ${project.name}"
               def configFile = new File(runtime_config_file)
               configFile.withOutputStream{os->
                   os << I am Debug\n'  //往配置文件里写 I am Debug
                }
           }
        }
       //找到preReleaseBuild任务
       tasks.getByName("preReleaseBuild"){
           it.doFirst{
               println "generate release configuration for ${project.name}"
               def configFile = new File(runtime_config_file)
               configFile.withOutputStream{os->
                   os << I am release\n'
               }
           }
        }
       //找到preDemoBuild。这个任务明显是因为我们在buildType里添加了一个demo的元素
      //所以Android APP插件自动为我们生成的
       tasks.getByName("preDemoBuild"){
           it.doFirst{
               println "generate offlinedemo configuration for${project.name}"
               def configFile = new File(runtime_config_file)
               configFile.withOutputStream{os->
                   os << I am Demo\n'
               }
            }
        }
    }
}
 .....//copyOutput
```
---
gradle文件中包含一些所谓的Script Block。ScriptBlock作用是让我们来配置相关的信息。不同的SB有不同的需要配置的东西。对应Gradle中Build script structure,里面都跟一个{},参数都是Closure,含有如:
>allproject{}:配置整个项目及其所有子项目
>artifacts{}:配置项目发布的产物
>buildscript{}:配置项目的构建脚本路径
>>buildscript：它的closure是在一个类型为ScriptHandler的对象上执行的。主意用来所依赖的classpath等信息。通过查看ScriptHandler API可知，在buildscript SB中，你可以调用ScriptHandler提供的repositories(Closure )、dependencies(Closure)函数。这也是为什么repositories和dependencies两个SB为什么要放在buildscript的花括号中的原因。明白了？这就是所谓的行话，得知道规矩。不知道规矩你就乱了。记不住规矩，又不知道查SDK，那么就彻底抓瞎，只能到网上到处找答案了！
>configuration{}:配置项目的依赖配置
>dependencies{}:配置项目的依赖
>repositories{}:配置项目存储仓库
>sourceSets{}:配置项目的源码集合
>subprojects{}:配置项目的子项目
>>它会遍历posdevice中的每个子Project。在它的Closure中，默认参数是子Project对应的Project对象。由于其他SB都在subprojects花括号中，所以相当于对每个Project都配置了一些信息。
>publishing{}:配置被发布插件添加的发布扩展
可以在[Script Block对应的api查找,此处也是gradle api索引][3]
## gradle技巧
* [多渠道打包适配productFlavors:美团][4]
* [如何使用Android studio把自己的Android library分享到jCenter和Maven Central][5]
* [使用Gradle生成一个App的不同版本，且可以同时安装在一个手机上][6]    
* 依赖更新： 手动设置Changing为true，resolutionStrategy设置检查更新
* [Android Gradle实战中遇到的问题与经验][7]
```
configurations.all {
    // check for updates every build
    resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
}
dependencies {
    compile group: "group", name: "projectA", version: "1.1-SNAPSHOT", changing: true
}
```
对于已经下载或缓存打本地的依赖文件,可以删除本地缓存,缓存在C:\Users\Administrator\.gradle\caches(win 7)再次运行即可自动重新下载远程依赖.
---
一种:
关于task
Java plugin和Android plugin都会创建以下task：
    * assemble：这个task将会组合项目的所有输出。
    * check：这个task将会执行所有检查。
    * build：这个task将会执行assemble和check两个task的所有工作
    * clean：这个task将会清空项目的输出。
实际上assemble，check，build这三个task不做任何事情。它们只是一个Task标志，用来告诉android plugin添加实际需要执行的task去完成这些工作。
这就允许你去调用相同的task，而不需要考虑当前是什么类型的项目，或者当前项目添加了什么plugin。
Android plugin使用相同的约定以兼容其他插件，并且附加了自己的标识性task，包括：
    * assemble：这个task用于组合项目中的所有输出。
    * check：这个task用于执行所有检查。
    * connectedCheck：这个task将会在一个指定的设备或者模拟器上执行检查，它们可以同时在所有连接的设备上执行。
    * deviceCheck：通过APIs连接远程设备来执行检查，这是在CL服务器上使用的。
    * build：这个task执行assemble和check的所有工作。
    * clean：这个task清空项目的所有输出。
这些新的标识性task是必须的，以保证能够在没有设备连接的情况下执行定期检查。
注意build task不依赖于deviceCheck或者connectedCheck。
一个Android项目至少拥有两个输出：debug APK（调试版APK)和release APK（发布版APK）。每一个输出都拥有自己的标识性task以便能够单独构建它们。
    * assemble：
        * assembleDebug
        * assembleRelease
它们都依赖于其它一些tasks以完成构建一个APK需要多个步骤。其中assemble task依赖于这两个task，所以执行assemble将会同时构建出两个APK。
小提示：gradle在命令行终端上支持骆驼命名法的task简称，例如，执行gradle aR命令等同于执行gradle assembleRelease。
check task也拥有自己的依赖：
    * check：
        * lint
    * connectedCheck：
        * connectedAndroidTest
        * connectedUiAutomatorTest(目前还没有应用到）
    * deviceCheck: 这个test依赖于test创建时，其它实现测试扩展点的插件。
最后，只要task能够被安装（那些要求签名的task），android plugin就会为所有构建类型（debug，release，test）安装或者卸载。
gradlew tasks --all
```
fbpay:assembleDefault [fbpay:assembleRelease]
yizhong:jarDebugClasses [fbpay:bundleRelease]
    yizhong:checkDebugManifest
    yizhong:compileDebugAidl
    yizhong:compileDebugJavaWithJavac
    yizhong:compileDebugRenderscript
    yizhong:generateDebugAssets
    yizhong:generateDebugBuildConfig
    yizhong:generateDebugResValues
    yizhong:generateDebugResources
    yizhong:generateDebugSources
    yizhong:mergeDebugAssets
    yizhong:mergeDebugResources
    yizhong:preBuild  
    yizhong:preDebugBuild
    yizhong:preReleaseBuild
    yizhong:prepareComAndroidSupportAppcompatV72220Library - Prepare com.android.support:appcompat-v7:22.2.0
    yizhong:prepareComAndroidSupportDesign2220Library - Prepare com.android.support:design:22.2.0
    yizhong:prepareComAndroidSupportRecyclerviewV72220Library - Prepare com.android.support:recyclerview-v7:22.2.0
    yizhong:prepareComAndroidSupportSupportV42220Library - Prepare com.android.support:support-v4:22.2.0
    yizhong:prepareComFeiboFbcore113Library - Prepare com.feibo:fbcore:1.1.3
    yizhong:prepareComFeiboSocialComponent105Library - Prepare com.feibo:social-component:1.0.5
    yizhong:prepareComGithubChrisbanesPhotoviewLibrary123Library - Prepare com.github.chrisbanes.photoview:library:1.2.3
    yizhong:prepareComMakeramenRoundedimageview220Library - Prepare com.makeramen:roundedimageview:2.2.0
    yizhong:prepareDebugDependencies
    yizhong:prepareInSrainCubeUltraPtr1010Library - Prepare in.srain.cube:ultra-ptr:1.0.10
    yizhong:prepareMeIwfPhotopickerPhotoPicker027Library - Prepare me.iwf.photopicker:PhotoPicker:0.2.7
    yizhong:prepareOneFbpayUnspecifiedLibrary - Prepare one:fbpay:unspecified
    yizhong:processDebugJavaRes
    yizhong:processDebugManifest
    yizhong:processDebugResources
yizhong:jarReleaseClasses [fbpay:bundleRelease]
    yizhong:checkReleaseManifest
    yizhong:compileReleaseAidl
    yizhong:compileReleaseJavaWithJavac
    yizhong:compileReleaseRenderscript
    yizhong:generateReleaseAssets
    yizhong:generateReleaseBuildConfig
    yizhong:generateReleaseResValues
    yizhong:generateReleaseResources
    yizhong:generateReleaseSources
    yizhong:mergeReleaseAssets
    yizhong:mergeReleaseResources
    yizhong:preBuild  
    yizhong:preDebugBuild
    yizhong:preReleaseBuild
    yizhong:prepareComAndroidSupportAppcompatV72220Library - Prepare com.android.support:appcompat-v7:22.2.0
    yizhong:prepareComAndroidSupportDesign2220Library - Prepare com.android.support:design:22.2.0
    yizhong:prepareComAndroidSupportRecyclerviewV72220Library - Prepare com.android.support:recyclerview-v7:22.2.0
    yizhong:prepareComAndroidSupportSupportV42220Library - Prepare com.android.support:support-v4:22.2.0
    yizhong:prepareComFeiboFbcore113Library - Prepare com.feibo:fbcore:1.1.3
    yizhong:prepareComFeiboSocialComponent105Library - Prepare com.feibo:social-component:1.0.5
    yizhong:prepareComGithubChrisbanesPhotoviewLibrary123Library - Prepare com.github.chrisbanes.photoview:library:1.2.3
    yizhong:prepareComMakeramenRoundedimageview220Library - Prepare com.makeramen:roundedimageview:2.2.0
    yizhong:prepareInSrainCubeUltraPtr1010Library - Prepare in.srain.cube:ultra-ptr:1.0.10
    yizhong:prepareMeIwfPhotopickerPhotoPicker027Library - Prepare me.iwf.photopicker:PhotoPicker:0.2.7
    yizhong:prepareOneFbpayUnspecifiedLibrary - Prepare one:fbpay:unspecified
    yizhong:prepareReleaseDependencies
    yizhong:processReleaseJavaRes
    yizhong:processReleaseManifest
    yizhong:processReleaseResources
yizhong:lintVitalRelease - Runs lint on just the fatal issues in the Release build. [fbpay:bundleRelease]
    yizhong:checkReleaseManifest
    yizhong:compileReleaseAidl
    yizhong:compileReleaseJavaWithJavac
    yizhong:compileReleaseRenderscript
    yizhong:generateReleaseAssets
    yizhong:generateReleaseBuildConfig
    yizhong:generateReleaseResValues
    yizhong:generateReleaseResources
    yizhong:generateReleaseSources
    yizhong:mergeReleaseAssets
    yizhong:mergeReleaseResources
    yizhong:preBuild  
    yizhong:preDebugBuild
    yizhong:preReleaseBuild
    yizhong:prepareComAndroidSupportAppcompatV72220Library - Prepare com.android.support:appcompat-v7:22.2.0
    yizhong:prepareComAndroidSupportDesign2220Library - Prepare com.android.support:design:22.2.0
    yizhong:prepareComAndroidSupportRecyclerviewV72220Library - Prepare com.android.support:recyclerview-v7:22.2.0
    yizhong:prepareComAndroidSupportSupportV42220Library - Prepare com.android.support:support-v4:22.2.0
    yizhong:prepareComFeiboFbcore113Library - Prepare com.feibo:fbcore:1.1.3
    yizhong:prepareComFeiboSocialComponent105Library - Prepare com.feibo:social-component:1.0.5
    yizhong:prepareComGithubChrisbanesPhotoviewLibrary123Library - Prepare com.github.chrisbanes.photoview:library:1.2.3
    yizhong:prepareComMakeramenRoundedimageview220Library - Prepare com.makeramen:roundedimageview:2.2.0
    yizhong:prepareInSrainCubeUltraPtr1010Library - Prepare in.srain.cube:ultra-ptr:1.0.10
    yizhong:prepareMeIwfPhotopickerPhotoPicker027Library - Prepare me.iwf.photopicker:PhotoPicker:0.2.7
    yizhong:prepareOneFbpayUnspecifiedLibrary - Prepare one:fbpay:unspecified
    yizhong:prepareReleaseDependencies
    yizhong:processReleaseJavaRes
    yizhong:processReleaseManifest
    yizhong:processReleaseResources
```
---
备注：
命令里加-q：能够使gradle log打印更清晰，只打印tasks的输出
对于java插件来说：
>gradle build：编译并测试代码，创建一个包含类和资源的jar文件
 gradle clean：删除build生成的目录和所有生成的文件
 gradle assemble：编译并打包代码，但不运行单元测；其他插件会加入额外的东西，War插件->生成war文件
 gradle check：编译把那个测试代码；其他插件会加入更多检查步骤，checkstyle->运行checkStyle来检查
 
 Groovy插件扩展java插件，加入编译Groovy的依赖，项目可以包含Groovy源码，java源码，甚至混合，因此是联合编译，加入该插件同时加入java插件
gradle脚本是配置脚本，当脚本执行，它就会配置一个特殊类型的对象，这个对象是这个脚本的执行者，也就是所谓的delegate object，每种脚本类型对应一类执行者类型或者叫做实例，
build script-> Project，init script->Gradle，settings script->Settings；
在对应脚本里可以使用该脚本对应执行者的方法和属性
以上脚本类型都实现了Script接口，这个接口里定义了大量的方法和属性
* **对于build script：**
它由0+个语句或者脚本块组成，语句可以包含方法调用，属性声明，局部变量定义。
build script也是Groovy script，因此在build script里可以包含在Groovy script里可以使用的元素，比如说方法定义和类的定义-->
一个脚本块是参数为闭包的方法调用。此处闭包被当做一个配置的闭包，可以配置执行闭包里的内容的对象，顶级的脚本块是：
allprojects{}
artifacts{}
buildscript{}
configurations{}
dependencies{}
repositories{}
sourceSets{}
subprojects{}
publishing{}
* **对于gradle script** 
以下是一些在该脚本里使用的核心类型：
``` 
 F:\one>gradlew properties
:properties                                                                              
                           
------------------------------------------------------------
Root project               
------------------------------------------------------------
                           
KEYALIAS: yizhong          
KEYSTORE: yizhong.keystore 
KEY_PASSWORD: fbandroid    
STORE_PASSWORD: fbandroid  
allprojects: [root project 'one', project ':fbpay', project ':yizhong']
ant: org.gradle.api.internal.project.DefaultAntBuilder@6866e740
antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@2cd5b19c
artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@7109b603
asDynamicObject: org.gradle.api.internal.ExtensibleDynamicObject@76b642aa
baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@29b5e7db
buildDir: F:\one\build     
buildFile: F:\one\build.gradle
buildScriptSource: org.gradle.groovy.scripts.UriScriptSource@286dfa20
buildscript: org.gradle.api.internal.initialization.DefaultScriptHandler@1468e880
childProjects: {fbpay=project ':fbpay', yizhong=project ':yizhong'}
class: class org.gradle.api.internal.project.DefaultProject_Decorated
classLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@508f4bb5
components: []             
configurationActions: org.gradle.configuration.project.DefaultProjectConfigurationActionContainer@5602e540
configurations: []         
convention: org.gradle.api.internal.plugins.DefaultConvention@11f9b95a
defaultTasks: []           
deferredProjectConfiguration: org.gradle.api.internal.project.DeferredProjectConfiguration@42066f0d
dependencies: org.gradle.api.internal.artifacts.dsl.dependencies.DefaultDependencyHandler_Decorated@687e561b
depth: 0                   
description: null          
ext: org.gradle.api.internal.plugins.DefaultExtraPropertiesExtension@299786b1
extensions: org.gradle.api.internal.plugins.DefaultConvention@11f9b95a
fileOperations: org.gradle.api.internal.file.DefaultFileOperations@75f8d9b0
fileResolver: org.gradle.api.internal.file.BaseDirFileResolver@4f7ae05
gradle: build 'one'        
group:                     
inheritedScope: org.gradle.api.internal.ExtensibleDynamicObject$InheritedDynamicObject@1e23ee0e
logger: org.gradle.logging.internal.slf4j.OutputEventListenerBackedLogger@b144175
logging: org.gradle.logging.internal.DefaultLoggingManager@38923cfe
market_channels:           
modelRegistry: org.gradle.model.internal.registry.DefaultModelRegistry@1ac3a6f
modelSchemaStore: org.gradle.model.internal.manage.schema.extract.DefaultModelSchemaStore@fee7ca
module: org.gradle.api.internal.artifacts.ProjectBackedModule@7ea4d397
name: one                  
parent: null               
parentIdentifier: null     
path: :                    
pluginManager: org.gradle.api.internal.plugins.DefaultPluginManager_Decorated@29c80149
plugins: [org.gradle.api.plugins.HelpTasksPlugin@601cbd8c]
processOperations: org.gradle.api.internal.file.DefaultFileOperations@75f8d9b0
proguard_on: true          
project: root project 'one'
projectDir: F:\one         
projectEvaluationBroadcaster: ProjectEvaluationListener broadcast
projectEvaluator: org.gradle.configuration.project.LifecycleProjectEvaluator@14ad42
projectRegistry: org.gradle.api.internal.project.DefaultProjectRegistry@608b906d
properties: {...}          
repositories: [org.gradle.api.internal.artifacts.repositories.DefaultMavenArtifactRepository_Decorated@173cfb01, org.gradle.api.internal.artifacts.repositories.DefaultMavenArtifactRe
pository_Decorated@7e1762e6, org.gradle.api.internal.artifacts.repositories.DefaultMavenArtifactRepository_Decorated@5bccaedb]
resources: org.gradle.api.internal.resources.DefaultResourceHandler@67784537
rootDir: F:\one            
rootProject: root project 'one'
scriptHandlerFactory: org.gradle.api.internal.initialization.DefaultScriptHandlerFactory@17ec5e2a
scriptPluginFactory: org.gradle.configuration.DefaultScriptPluginFactory@52290e63
serviceRegistryFactory: org.gradle.internal.service.scopes.ProjectScopeServices$4@6c2dd88b
services: ProjectScopeServices
standardOutputCapture: org.gradle.logging.internal.DefaultLoggingManager@38923cfe
state: project state 'EXECUTED'
status: release            
subprojects: [project ':fbpay', project ':yizhong']
tasks: [task ':properties']
version: unspecified       
                           
BUILD SUCCESSFUL
```
  [1]: http://www.open-open.com/lib/view/open1439216256770.html
  [2]: http://img.blog.csdn.net/20150905194741289?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center
  [3]: https://docs.gradle.org/current/javadoc/
  [4]: http://tech.meituan.com/mt-apk-adaptation.html%5D%28http://tech.meituan.com/mt-apk-adaptation.html
  [5]: http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0623/3097.html
  [6]: http://hugozhu.myalert.info/2014/08/03/50-use-gradle-to-customize-apk-build.html
  [7]: http://www.open-open.com/lib/view/open1439216256770.html
  




