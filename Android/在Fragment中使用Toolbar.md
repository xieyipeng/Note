## 在Fragment中使用Toolbar

在Fragment中使用Toolbar的步骤和Activity差不多.
在Fragment布局中添加一个Toolbar, 然后find它, 然后调用Activity的方法来把它设置成ActionBar:

```
((AppCompatActivity) getActivity()).setSupportActionBar(toolbar);
```

注意此处有一个强转, 必须是AppCompatActivity才有这个方法.
但是此时运行到Fragment之后, 发现Toolbar上的文字和按钮全是Activity传过来的, 这是因为只有Activity的`onCreateOptionsMenu()`被调用了, 但是Fragment的并没有被调用.
在Fragment中加上这句:

```
setHasOptionsMenu(true);
```

此时Fragment的`onCreateOptionsMenu()`回调会被调到了, 但是inflate出的按钮和Activity中的actions加在一起显示出来了.
因为Activity的`onCreateOptionsMenu()`会在之前调用到.
于是Fragment中的写成这样:

```
@Override
public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
    Log.e(TAG, "onCreateOptionsMenu()");
    menu.clear();
    inflater.inflate(R.menu.menu_parent_fragment, menu);
}
```

即先clear()一下, 这样按钮就只有Fragment中设置的自己的了, 不会有Activity中的按钮.