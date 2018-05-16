这几天写了自己人生中的第一个项目。。。虽然是和别人一起完成的，但也是自己的心血了，在上传git的时候，显示报错：

```java
Counting objects: 1397, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (1149/1149), done.
Writing objects: 100% (1397/1397), 203.41 MiB | 771.00 KiB/s, done.
Total 1397 (delta 446), reused 0 (delta 0)
remote: Resolving deltas: 100% (446/446), completed with 13 local objects.
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
remote: error: Trace: 63c7ef5b5f5816c144ba83799f306f67
remote: error: See http://git.io/iEPt8g for more information.
remote: error: File The computer application competition/Englishplay.zip is 104.45 MB; this exceeds GitHub's file size limit of 100.00 MB
To https://github.com/xieyipeng/Android.git
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'https://github.com/xieyipeng/Android.git'
```

很明显发生了错误：

```java
统计对象:1397年,完成了。

使用最多4个线程的压缩。

压缩对象:100%(1149/1149)，完成。

写作对象:100% (1397/1397)，203.41 MiB |771.00 KiB/s，完成。

总1397 (delta 446)，再利用0 (0)

远程:解决deltas: 100%(446/446)，完成13个本地对象。

远程:错误:GH001:检测到大文件。您可能想尝试Git大文件存储——https://gitlfs.github.com。

远程:错误:跟踪:63 c7ef5b5f5816c144ba83799f306f67

远程:错误:见http://git。io / iEPt8g获得更多信息。

远程:错误:文件计算机应用程序比赛/英语游戏。zip是104.45 MB;这超过了GitHub的文件大小限制为100.00 MB。

到https://github.com/xieyipeng/Android.git

![远程拒绝]master -> master(预收钩子下降)

错误:未能将一些refs推入'https://github.com/xieyipeng/Android.git'
```

问题原因： 
问题原因是http.postBuffer默认上限为1M所致。在git的配置里将http.postBuffer变量改大一些即可，比如将上限设为500M： 

```java
git config --global http.postBuffer 524288000
```

在哪里执行以上命令呢？ 

打开git bash命令行工具。 

注意要加上--global。网上很多资料都没加这个参数。不加执行的话会报以下错误的： 

```java
error:could not lock config file .git/config: no such file or directory.
```

