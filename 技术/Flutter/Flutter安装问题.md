# Flutter安装常见问题

## 问题1：

正常下载flutter sdk后，设置flutter/bin环境变量后，执行flutter doctor可能会出现“pub upgrade error”，此时我们需要设置两个全局环境变量：

```bash
PUB_HOSTED_URL: https://pub.flutter-io.cn
FLUTTER_STORAGE_BASE_URL: https://storage.flutter-io.cn
```

## 问题2：

执行flutter doctor时，可能会出现显示的android studio版本与你安装的版本不一样，所以即使你安装的版本已经安装了flutter与dart插件，flutter doctor还是会提示你没有安装。其实，如果确认插件已经安装完成，这个可以不用在意，根据安装的android studio打开直接创建flutter项目即可。

## 问题3：

调试Flutter项目的时候可能会出现“Running Gradle task 'assembleDebug'...”然后一直卡在这里不动。

也许你会认为翻墙可以解决，但很遗憾解决不了。

正确的解决办法：

第一步：打开flutter项目中\android\gradle\wrapper\gradle-wrapper.properties文件，内容可能如下：

```bash
#Fri Jun 23 08:50:38 CEST 2017
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-5.6.2-all.zip
```

通过distributionUrl的值，下载gradle对应的版本，如上：gradle-5.6.2-all.zip到本地目录，如：M:\learning\flutter\gradle-5.6.2-all.zip，如果发现无法下载，此时可以考虑翻墙。

下载完后，修改上面文件内容，如下：

```bash
#Fri Jun 23 08:50:38 CEST 2017
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=file\:/M\:/learning/flutter/gradle-5.6.2-all.zip
```

即，修改distributionUrl的值指向本地下载好的gradle。

第二步：修改项目中/android/build.gradle 文件，如下：

```bash
buildscript {
    ext.kotlin_version = '1.3.50'
    repositories {
        // google()
        // jcenter()
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/jcenter' }
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public' }
        maven { url 'http://download.flutter.io' }
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.0'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

allprojects {
    repositories {
        // google()
        // jcenter()
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/jcenter' }
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public' }
        maven { url 'http://download.flutter.io' }
    }
}

rootProject.buildDir = '../build'
subprojects {
    project.buildDir = "${rootProject.buildDir}/${project.name}"
}
subprojects {
    project.evaluationDependsOn(':app')
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

第三步：修改flutter sdk的目录中packages\flutter_tools\gradle\flutter.gradle文件，如下：

```bash
buildscript {
    repositories {
        // google()
        // jcenter()
		maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/jcenter' }
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public' }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.0'
    }
}

android {
    compileOptions {
        sourceCompatibility 1.8
        targetCompatibility 1.8
    }
}

apply plugin: FlutterPlugin

class FlutterPlugin implements Plugin<Project> {
    private static final String DEFAULT_MAVEN_HOST = "https://storage.flutter-io.cn/download.flutter.io";
```

第四步：修改flutter sdk的目录中packages/flutter_tools/gradle/resolve_dependencies.gradle文件，如下：

```bash
repositories {
    // google()
    // jcenter()
	maven { url 'https://maven.aliyun.com/repository/google' }
    maven { url 'https://maven.aliyun.com/repository/jcenter' }
    maven { url 'http://maven.aliyun.com/nexus/content/groups/public' }
    maven {
        url "https://storage.flutter-io.cn/download.flutter.io"
    }
}
```

至此问题解决，运行项目时你会看到如下控制台输出：

```bash
Launching lib\main.dart on sdk gphone x86 arm in debug mode...
Running Gradle task 'assembleDebug'...
Checking the license for package Android SDK Build-Tools 28.0.3 in C:\Users\wyin\AppData\Local\Android\sdk\licenses
License for package Android SDK Build-Tools 28.0.3 accepted.
Preparing "Install Android SDK Build-Tools 28.0.3 (revision: 28.0.3)".
"Install Android SDK Build-Tools 28.0.3 (revision: 28.0.3)" ready.
Installing Android SDK Build-Tools 28.0.3 in C:\Users\wyin\AppData\Local\Android\sdk\build-tools\28.0.3
"Install Android SDK Build-Tools 28.0.3 (revision: 28.0.3)" complete.
"Install Android SDK Build-Tools 28.0.3 (revision: 28.0.3)" finished.
Checking the license for package Android SDK Platform 29 in C:\Users\wyin\AppData\Local\Android\sdk\licenses
License for package Android SDK Platform 29 accepted.
Preparing "Install Android SDK Platform 29 (revision: 5)".
"Install Android SDK Platform 29 (revision: 5)" ready.
Installing Android SDK Platform 29 in C:\Users\wyin\AppData\Local\Android\sdk\platforms\android-29
"Install Android SDK Platform 29 (revision: 5)" complete.
"Install Android SDK Platform 29 (revision: 5)" finished.
√ Built build\app\outputs\flutter-apk\app-debug.apk.
Installing build\app\outputs\flutter-apk\app.apk...
Waiting for sdk gphone x86 arm to report its views...
Debug service listening on ws://127.0.0.1:24807/ctpqdgCAklY=/ws
Syncing files to device sdk gphone x86 arm...
```