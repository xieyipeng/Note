## 一、首先是Listview的属性设置

设置滑动到顶部和底部的背景或颜色：

```
android:overScrollFooter="@android:color/transparent"
android:overScrollHeader="@android:color/transparent"12
```

设置滑动到边缘时无效果模式：

```
android:overScrollMode="never"1
```

设置滚动条不显示：

```
android:scrollbars="none"1
```

以下是整体设置（overScrollHeader和overScrollFooter可不写，此处写了是引用的透明色）

```
<ListView
    android:id="@+id/lv_type"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:overScrollFooter="@android:color/transparent"
    android:overScrollHeader="@android:color/transparent"
    android:overScrollMode="never"
    android:scrollbars="none">12345678
```

## 二、RecyclerView的属性设置

以下是整体设置：

```
<android.support.v7.widget.RecyclerView
    android:id="@+id/rv_search_one"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:overScrollMode="never"
    android:scrollbars="none" />
```