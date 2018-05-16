报错：

```java
java.lang.UnsatisfiedLinkError: Couldn't load weibosdkcore from loader dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.dk.dkweibo-2.apk"],nativeLibraryDirectories=[/data/app-lib/com.dk.dkweibo-2, /system/lib]]]: findLibrary returned null
at java.lang.Runtime.loadLibrary(Runtime.java:358)
at java.lang.System.loadLibrary(System.java:526)
at com.sina.weibo.sdk.net.HttpManager.<clinit>(HttpManager.java:83)
at com.sina.weibo.sdk.net.NetUtils.internalHttpRequest(NetUtils.java:46)
at com.sina.weibo.sdk.utils.AidTask.loadAidFromNet(AidTask.java:344)
at com.sina.weibo.sdk.utils.AidTask.access$3(AidTask.java:331)
at com.sina.weibo.sdk.utils.AidTask$2.run(AidTask.java:203)
at java.lang.Thread.run(Thread.java:841)
```

问题原因：

.so文件缺失，在引用有道词典的API(.jar文件)时需要引入.so文件。

而对于不同的CPU，.so文件是不同的，所以API提供了arm64-v8a、armeabi-v7a、x86、armeabi、mips等七个文件夹，里面都含各自的.so文件。

解决办法：

1. 在项目中的src文件夹创建jniLibs目录；

2. 把当前目录的arm64-v8a、armeabi-v7a、x86、armeabi、mips等全部文件夹拷贝到项目的jniLibs目录中；

3. 打开gradle.build文件，修改为（这一步是为了能把.so文件编译进去）

   ```java
   android{
   　　　　　　...
   　　　　　　sourceSets.main{
   　　　　　　　　jniLibs.srcDirs = ['src/jniLibs'];
   　　　　　　}
   　　　　　　...
   　　　　}
   ```