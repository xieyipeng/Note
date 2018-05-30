1.首先项目gradle中添加:

```
compile 'com.android.support:appcompat-v7:23.4.0'
```

2.确保Activity继承`AppCompatActivity`
3.在application设置中使用NoActionBar的主题:

```
<application
    android:theme="@style/Theme.AppCompat.Light.NoActionBar"
    />
```

4.把Toolbar写在布局中

```
<android.support.v7.widget.Toolbar
   android:id="@+id/my_toolbar"
   android:layout_width="match_parent"
   android:layout_height="?attr/actionBarSize"
   android:background="?attr/colorPrimary"
   android:elevation="4dp"
   android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
   app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
```

5.在Activity里面把Toolbar设置成为ActionBar
首先把Toolbar find出来, 然后调用[setSupportActionBar方法](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html#setSupportActionBar(android.support.v7.widget.Toolbar))
把Toolbar设置为自己的ActionBar即可.

```
public class ToolbarDemoActivity extends AppCompatActivity {

    @BindView(R.id.toolbar)
    Toolbar toolbar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_toolbar_demo);
        ButterKnife.bind(this);
        setSupportActionBar(toolbar);
    }
}
```

然后就可以随意使用啦, 用[getSupportActionBar](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html#getSupportActionBar())可以获取ActionBar类型的对象, 从而使用[ActionBar](https://developer.android.com/reference/android/support/v7/app/ActionBar.html)的方法.

### 添加Action Buttons

定义menu:

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_android"
        android:icon="@drawable/ic_android_black_24dp"
        android:title="@string/action_android"
        app:showAsAction="always" />
    <item
        android:id="@+id/action_favourite"
        android:icon="@drawable/ic_favorite_black_24dp"
        android:title="@string/action_favourite"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/action_settings"
        android:title="@string/action_settings"
        app:showAsAction="never" />
</menu>
```

然后在代码中inflate和处理它的点击事件:

```
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    Log.i(TAG, "onCreateOptionsMenu()");
    getMenuInflater().inflate(R.menu.menu_activity_main, menu);
    return super.onCreateOptionsMenu(menu);
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.action_android:
            Log.i(TAG, "action android selected");
            return true;
        case R.id.action_favourite:
            Log.i(TAG, "action favourite selected");
            return true;
        case R.id.action_settings:
            Log.i(TAG, "action settings selected");
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

### 添加向上返回的action

添加向上返回parent的action:

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_toolbar_demo);
    ButterKnife.bind(this);
    setSupportActionBar(toolbar);

    // add a left arrow to back to parent activity,
    // no need to handle action selected event, this is handled by super
    getSupportActionBar().setDisplayHomeAsUpEnabled(true);
}
```

然后只需要在manifest中指定parent:

```
<activity
    android:name=".toolbar.ToolbarDemoActivity"
    android:parentActivityName=".MainActivity"></activity>
```