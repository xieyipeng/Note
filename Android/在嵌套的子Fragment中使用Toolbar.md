前面已经介绍过, Fragment可以嵌套使用: [Android Fragment使用(二) 嵌套Fragments (Nested Fragments) 的使用及常见错误](http://www.cnblogs.com/mengdd/p/5552721.html).
那么在前面的Fragment中再显示一个子Fragment, 并且又带有一个不一样的Toolbar, 还需要哪些处理呢?
首先, java代码中还是需要有:

```
setHasOptionsMenu(true)
((AppCompatActivity) getActivity()).setSupportActionBar(toolbar);
```

然后根据是否需要菜单按钮, 覆写onCreateOptionsMenu()方法来inflate自己的menu文件即可.
感觉和在普通的Fragment中使用Toolbar作为ActionBar并没有什么区别.
但是如果你的多个Fragment有不同的Toolbar菜单选项, 如果你没有懂得其中的原理, 可能就会出现一些混乱.
下面来解说一下相关的方法.

### onCreateOptionsMenu()方法的调用

一旦调用

```
((AppCompatActivity) getActivity()).setSupportActionBar(toolbar);
```

就会导致Activity`onCreateOptionsMenu()`方法的调用, 而Activity会根据其中Fragment是否设置了setHasOptionsMenu(true)来调用Fragment的
`onCreateOptionsMenu()`方法, 调用顺序是树形的, 按层级调用, 中间如果有false则跳过.

假设当前Activity, Parent Fragment和Child Fragment中都设置了自己的Toolbar为ActionBar.
在打开Child fragment的时候, `onCreateOptionsMenu()`的调用顺序是.
`Activity -> Parent -> Child.` 此时parent和child fragment都设置了setHasOptionsMenu(true).

关于这个, 还有以下几种情况:

```
- 如果Parent的`setHasOptionsMenu(false)`, Child为true, 则Parent的`onCreateOptionsMenu()`不会调用, 打开Child的时候Activity -> Child.
- 如果Child的`setHasOptionsMenu(false)`, Parent为true, 则打开Child的时候仍然会调用Activity和Parent的onCreateOptionsMenu()方法.
- 如果Parent和Child都置为false, 打开Parent和Child Fragment的时候都会调用Activity的onCreateOptionsMenu()方法.
```

**仅仅是child Fragment的show() hide()的切换, activity和parent Fragment的onCreateOptionsMenu()也会重新进入.**
这一点我还没有想明白, 是项目中遇到的, 初步推测可能是menu的显隐变化invalidate了menu, 改天有空再试试.

上面的机制常常是导致Toolbar上面的按钮混淆错乱的原因.
举个例子:
如果我们现在Activity和Parent Fragment有不同的Toolbar按钮, 但是Child只有文字, 没有按钮.
很显然我们不需要给child写menu文件, 也不需要覆写child里的`onCreateOptionsMenu()`方法.
但是此时不管怎样, parent的`onCreateOptionsMenu()`方法都会被调用, 这样我们打开child的时候, toolbar上就神奇地出现了parent里的按钮.
这种情况如何解决呢?
可以在parent中加一个条件, 当没有child fragment的时候才做inflate的工作:

```
@Override
public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
    Log.e(TAG, "onCreateOptionsMenu()");
    menu.clear();
    if (getChildFragmentManager().getBackStackEntryCount() == 0) {
        inflater.inflate(R.menu.menu_parent_fragment, menu);
    }
}
```

另外, 除了`setSupportActionBar()`之外, 如果我们想**主动触发** `onCreateOptionsMenu()`方法的调用, 可以用
`invalidateOptionsMenu()`方法.

### onOptionsItemSelected()方法的调用

在Activity和其中的Fragment都有options menu的时候, 需要注意menu item的id不要重复.
以为点击事件的分发也是从Activity开始分发下去的, 如果child fragment中有个选项的id和Activity中一个选项的id重复了, 则在Activity中就会将其处理, 不会继续分发.

### 有嵌套Fragment时 Back键处理

之前没有嵌套Fragment的情况下, 只要将Fragment加入到Back Stack中, 那么按下Back键的时候pop动作是系统自动做好的.
虽然在添加child fragment的时候将其加入到back stack中, 但是按back键的时候仍然是将parent fragment弹出, 只剩下Activity.
这是因为back键只检查第一层Fragment的back stack, 对于child fragment, 需要在其parent中自己处理.
比如这样处理:

在Activity中

```
@Override
public void onBackPressed() {
    Fragment fragment = getSupportFragmentManager().findFragmentById(android.R.id.content);
    if (fragment instanceof ToolbarFragment) {
        if (((ToolbarFragment) fragment).onBackPressed()) {
            return;
        }
    }
    super.onBackPressed();
}
```

其中ToolbarFragment是直接加在Activity中作为parent fragment的.
在parent fragment中(即ToolbarFragment中):

```
public boolean onBackPressed() {
    return getChildFragmentManager().popBackStackImmediate();
}
```