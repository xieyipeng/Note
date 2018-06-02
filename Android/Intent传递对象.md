Activity_A:

```java
Intent intent = new Intent(view.getContext(), SearchResultActivity.class);
                        intent.putExtra("result", resutlt);
                        startActivity(intent);
```

Activity_B:

```java
result=getIntent().getStringExtra("result");//获得result
```

