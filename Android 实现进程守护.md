# Android 实现进程守护

标签（空格分隔）： 常驻进程 进程守护

---

###Native保活

[相关博客](http://blog.csdn.net/marswin89/article/details/50917098)

###相关库的使用

[github](https://github.com/Morcal/MarsDaemon.git)

###将库文件打包到jcenter

[参考示例](https://github.com/openproject/LessCode/blob/master/lesscode-core/build.gradle)

####在项目的build.gradle中添加classpath

```
classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.0'
classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3'
```

####在库module的build.gradle中添加插件

```
// 根节点添加
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'
```
####定义版本

> version = "0.1"

####定义相关网站

```
def siteUrl = 'https://github.com/Morcal/MarsDaemon'                        // #CONFIG# // project homepage
def gitUrl = 'https://github.com/Morcal/MarsDaemon.git'  

####定义Group,引用时会显示

> group = "com.morcal.marsdaemon"

####定义pom并打包aar

上传到jcenter至少需要四个文件，除了打包的aar之外，还需要pom和javadoc，source，否则是通不过jcenter审核的。不过不用紧张，这些我们都可以用脚本生成。

```
install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'
                name 'Mars daemon For Android'                                   // #CONFIG# // project title
                url siteUrl
                // Set your license
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id 'morcal'                                           // #CONFIG# // your user id (you can write your nickname)
                        name 'lyqdhgo'                                       // #CONFIG# // your user name
                        email 'lyqdhgo@163.com'                               // #CONFIG# // your email
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}


task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives javadocJar
    archives sourcesJar
}
```

####上传到JCenter仓库

上传到jcenter的网站BinTray，需要用户验证，需要2个值：它们属于个人隐私，一般保存在项目的local.properties配置文件中

>bintray.user=lyqdhgo
bintray.apikey=7b5207c0dc75ddbbf99985d8e88cbe5c4773df35

####在bulid.gradle中通过Properties来获取local.properties中的配置信息

```
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")
    configurations = ['archives']
    pkg {
        repo = "maven"
        name = "marsdaemon"                                                 // #CONFIG# project name in jcenter
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
}
```

####执行命令行控制

> gradlew javadocJar
  gradlew sourcesJar
  gradlew install
  gradlew bintrayUpload
  
####打包到远程

* 相关教程

[教程一](http://www.cnblogs.com/qianxudetianxia/p/4322331.html)
[教程二](http://www.voidcn.com/blog/qqyanjiang/article/p-5021401.html)  

* 相关错误

1.javaDoc

>What went wrong:
Execution failed for task ':MaterialDesign:javadoc'.
> Javadoc generation failed. Generated Javadoc options file (useful for troubleshooting): 'D:\360\test\MaterialDesignLibrary-master\MaterialDesignLibrary\MaterialDesign\build\tmp\javadoc\javadoc.options'

 解决方案：删除代码中的所有html标记，编码不可为GBK

 2.gradle版本问题

 解决方案：在gradle.wrapper改变
 
 >distributionUrl=https\://services.gradle.org/distributions/gradle-2.2.1-all.zip






