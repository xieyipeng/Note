git操作的过程中突然显示：

```java
Another git process semms to be running in this repository, e.g. an editor opened by ‘git commit’. Please make sure all processes are terminated then try again. If it still fails, a git process remove the file manually to continue… 
```

翻译过来就是git被另外一个程序占用，重启机器也不能够解决。

原因在于Git在使用过程中遭遇了奔溃，部分被上锁资源没有被释放导致的。

解决方案：

进入项目文件夹下的 .git文件中（显示隐藏文件夹或rm .git/index.lock）删除index.lock文件即可。