## 如何编译：携程 - DynamicAPK

### 如何使用

#### 编译

>   gradle clean

>   gradle afterAssembleRelease

>   gradle bundleRelease

>   gradle repackAll

#### 编译并生成依赖图

>    gradle clean

>   gradle afterAssembleRelease

>dot ./build-outputs/reports/visteg.dot -T png -o ./build-outputs/reports/visteg_afterAssembleRelease.dot.png

>   gradle bundleRelease

>dot ./build-outputs/reports/visteg.dot -T png -o ./build-outputs/reports/visteg_bundleRelease.dot.png

>   gradle repackAll

>   dot ./build-outputs/reports/visteg.dot -T png -o ./build-outputs/reports/visteg_repackAll.dot.png

## Task说明

#### 宿主：sample

assembleRelease

```
打包apk
```

afterAssembleRelease (sample/build.gradle)

```
输入：
	/sample/build/outputs/apk/release/sample-release.apk
	rename 'sample-release.apk', 'demo-base-release.apk'

	/sample/build/intermediates/manifests/full/release/AndroidManifest.xml

	/sample/build/outputs/mapping/release/mapping.txt
	rename 'mapping.txt', 'demo-base-mapping.txt'

输出：
	/build-outputs/demo-base-release.apk 宿主代码 + 宿主资源 + 宿主资源id
	/build-outputs/AndroidManifest.xml
	/build-outputs/demo-base-mapping.txt
```

#### 插件：demo1

init

```
输出：
	/build-outputs
	/demo1/build/gen/r
   	/demo1/intermediates
   	/demo1/intermediates/classes
   	/demo1/intermediates/classes-obfuscated
   	/demo1/intermediates/res
   	/demo1/intermediates/dex
```

aaptRelease (‘init’) (/sub-project-build.gradle)

```
输入：
	/build-outputs/demo-base-release.apk 宿主资源(插件需要)
	/sample/build/generated/source/r/release/ctrip/android/sample/R.java 宿主资源id
	/demo1/AndroidManifest.xml"
	/demo1/res
	/demo1/assets
	sdk.androidJar
	apk_module_config.xml (插件资源id前缀配置文件)

输出：
	/demo1/build/gen/r/…/R.java                插件资源id + 宿主资源id
	/demo1/intermediates/res/resources.zip     插件资源 + 宿主资源
	/demo1/intermediates/res/aapt-rules.txt    混淆规则(后续混淆时使用)

执行：
	package
```

compileRelease ('aaptRelease')  (/sub-project-build.gradle)

```
输入：
	/demo1/src/*.java
	/demo1//libs/*.jar
	/demo1/gen/r/…/R.java
	/sample/build/intermediates/classes-proguard/release/classes.jar(没有？)
	sdk.androidJar
	sdk.apacheJar

输出：
	demo1/intermediates/classes/*.class
	demo1/dependency-cache (没有？)

执行：
    javac
```

obfuscateRelease (‘compileRelease’)  (/sub-project-build.gradle)

```
输入：
	/sub-project-proguard-rules.txt
	/demo1/build/intermediates/res/aapt-rules.txt
	/demo1/build/intermediates/classes"
	/demo1/libs/*.jar
	/sample/build/intermediates/classes-proguard/release/classes.jar
	sdk.androidJar
	/sub-project-proguard-rules.txt
	/demo1/build/intermediates/res/aapt-rules.txt

输出：
	/demo1/build/intermediates/classes-obfuscated/classes-obfuscated.jar
	/build-outputs/${apkName}-mapping.txt

执行：
	obfuscate
```

dexRelease  (/sub-project-build.gradle)

```
输入：
	/demo1/build/intermediates/classes"

输出：
	/demo1/build/intermediates/dex/demo1_dex.zip"

执行：
	dex
```

bundleRelease (‘compileRelease','aaptRelease','dexRelease’) 

```
输入：
	/demo1/build/intermediates/dex/demo1_dex.zip
	/demo1/build/intermediates/res/resources.zip

输出：
	/build-outputs/ctrip_android_demo1.so"

执行：
    zip
```

—————
reload

```
输入：
	/build-outputs/demo-base-release.apk
	/build-outputs/*.so

输出：
	/build-outputs/demo-release-reloaded.apk

执行：
	zip
```

repack

```
输入：
	/build-outputs/demo-release-reloaded.apk
输出：
	/build-outputs/demo-release-repacked.apk
操作：
	
```

resign

```
输入：
	/build-outputs/demo-release-repacked.apk
	/*.jks
输出：
	/build-outputs/demo-release-resigned.apk
操作：
	
```

realign

```
输入：
	/build-outputs/demo-release-resigned.apk
输出：
	/build-outputs/demo-release-final.apk
操作：
	
```

concatMappings

```
输入：
	/build-outputs/*mapping.txt'
输出：
	/build-outputs/demo-mapping-final.txt
```

repackAll ('reload','resign','repack','realign','concatMappings') (/sample/build.gradle)

```
输入：
	
输出：
	
```

### 参考

[Android 携程动态加载框架的打包流程分析](http://blog.csdn.net/sbsujjbcy/article/details/49848715)

[携程动态加载实践DynamicAPK项目学习](http://lilei.work/2016/01/18/%E6%90%BA%E7%A8%8B%E5%8A%A8%E6%80%81%E5%8A%A0%E8%BD%BD%E5%AE%9E%E8%B7%B5DynamicAPK%E9%A1%B9%E7%9B%AE%E5%AD%A6%E4%B9%A0/)

[Android项目中的Gradle Task流程可视化](https://www.jianshu.com/p/6599de4cdcd1)
[mac下的Graphviz安装及使用](http://blog.csdn.net/qq_36847641/article/details/78224910)

[Gradle脚本基础全攻略](http://blog.csdn.net/yanbober/article/details/49314255)

[全面理解Gradle - 定义Task](http://blog.csdn.net/singwhatiwanna/article/details/78898113?utm_source=tuicool&utm_medium=referral)

[正确运行携程DynamicAPK](http://blog.csdn.net/ujnnet/article/details/52042900)

[关于MAC下Android SDK manager 更新解决办法（无需翻墙）](http://blog.csdn.net/c_clear/article/details/50490861)

### 编译错误处理参考

[The TaskInternal.execute() method has been deprecated and is scheduled to be removed in Gradle 5.0.](https://discuss.gradle.org/t/the-taskinternal-execute-method-has-been-deprecated-and-is-scheduled-to-be-removed-in-gradle-5-0/24897/11)





