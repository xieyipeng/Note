## 设置ScrollView的滚动条为隐藏的方法，scrollview滚动条

 

要实现ScrollView滚动条的隐藏，有两种方法，

一种是在XML的ScrollView布局中加入属性android:scrollbars="none"

另一种则是在代码中获取ScrollView后进行scroll.setVerticalScrollBarEnabled(false);

 

### 怎隐藏滚动条在ScrollView中

如何在Android的ScrollView中隐藏滚动条呢，其实通过两个方法就可以搞定了。在layout的xml布局文件中，比如有

```xml
<ScrollView
    android:id="@+id/sView"
```

这个代码。在Java中使用

```java
ScrollView sView = (ScrollView)findViewById(R.id.sView);
sView.setVerticalScrollBarEnabled(false); //禁用垂直滚动 
sView.setHorizontalScrollBarEnabled(false); //禁用水平滚动
```

