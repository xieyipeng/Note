```java
android.support.v7.app.ActionBar actionbar=((AppCompatActivity)getActivity()).getSupportActionBar();
if(actionbar==null)
    Toast.makeText(getActivity(), "NULL", Toast.LENGTH_SHORT).show();
else
    Toast.makeText(getActivity(), "Not NULL", Toast.LENGTH_SHORT).show();
```

