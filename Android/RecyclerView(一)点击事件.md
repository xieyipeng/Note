## 1、通过接口回调的方式：

直接上代码加注释 
1、首先定义点击的接口

```java
public interface OnItemClickListener {
    void onItemClick(View view, int position);
}
```

2、在recyclerview的adapter中实现点击方法

```java
public class ListRecycleViewAdapter extends RecyclerView.Adapter<ListRecycleViewAdapter.ListViewHolder>{

    private Context mContext;
    private List<String> mList;

    public ListRecycleViewAdapter(Context context, List<String> list) {
        mContext = context;
        mList = list;
    }

    @Override
    public ListViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        return new ListViewHolder(LayoutInflater.from(mContext).inflate(R.layout.item_list_recycle_view, parent,
                false));
    }

    @Override
    public void onBindViewHolder(final ListViewHolder holder, int position) {
        holder.tv.setText(mList.get(position));

        View itemView = ((RelativeLayout) holder.itemView).getChildAt(0);

        if (mOnItemClickListener != null) {
            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    int position = holder.getLayoutPosition();
                    mOnItemClickListener.onItemClick(holder.itemView, position);
                }
            });
        }
    }

    @Override
    public int getItemCount() {
        return mList.size();
    }

    private OnItemClickListener mOnItemClickListener;//声明接口

    public void setOnItemClickListener(OnItemClickListener onItemClickListener) {
        mOnItemClickListener = onItemClickListener;
    }

    class ListViewHolder extends RecyclerView.ViewHolder {
        TextView tv;

        public ListViewHolder(View itemView) {
            super(itemView);
            tv = (TextView) itemView.findViewById(R.id.tv_list_item);
        }
    }
}
```

3、在activity中使用

```java
ListRecycleViewAdapter listRecycleViewAdapter = new ListRecycleViewAdapter(SettingActivity.this, mList);
   listRecycleViewAdapter.setOnItemClickListener(new OnItemClickListener() {
            @Override
            public void onItemClick(View view, int position) {

            }
        }
```



## 2，通过recyclerview的addOnItemTouchListener实现点击事件

```java
    1、首先定义类RecyclerItemClickListener实现 RecyclerView.OnItemTouchListener接口，重写onLongPress（）方法，onInterceptTouchEvent（）方法，onTouchEvent（）方法，onRequestDisallowInterceptTouchEvent（）方法，并在本类内部定义一个点击监听的接口interface OnItemClickListener，接口内部设置两个方法onItemClick（），onLongClick（）。下面上代码：
```

```java
public static class RecyclerItemClickListener implements RecyclerView.OnItemTouchListener {
        GestureDetector mGestureDetector;
        private View childView;
        private RecyclerView touchView;

        public RecyclerItemClickListener(Context context, final HomePageFragment.RecyclerItemClickListener.OnItemClickListener mListener) {
            mGestureDetector = new GestureDetector(context, new GestureDetector.SimpleOnGestureListener() {
                @Override
                public boolean onSingleTapUp(MotionEvent ev) {
                    if (childView != null && mListener != null) {
                        mListener.onItemClick(childView, touchView.getChildPosition(childView));
                    }
                    return true;
                }

                @Override
                public void onLongPress(MotionEvent ev) {
                    if (childView != null && mListener != null) {
                        mListener.onLongClick(childView, touchView.getChildPosition(childView));
                    }
                }
            });
        }

        public interface OnItemClickListener {
            void onItemClick(View view, int position);

            void onLongClick(View view, int posotion);
        }

        @Override
        public boolean onInterceptTouchEvent(RecyclerView rv, MotionEvent e) {
            mGestureDetector.onTouchEvent(e);
            childView = rv.findChildViewUnder(e.getX(), e.getY());
            touchView = rv;
            return false;
        }

        @Override
        public void onTouchEvent(RecyclerView rv, MotionEvent e) {

        }

        @Override
        public void onRequestDisallowInterceptTouchEvent(boolean disallowIntercept) {

        }
    }
```

2、将我们的布局中定义recyclerview添加方法 addOnItemTouchListener（）

```java
rc_view.addOnItemTouchListener(new RecyclerItemClickListener(this，new RecyclerItemClickListener.OnItemClickListener() {
  public void onItemClick(View view, int position) {
           //在此处做点击之后的逻辑处理
  ｝
   @Override
            public void onLongClick(View view, int position) {
            //在此处做长按之后的逻辑处理
            //    Toast.makeText(getContext(), "长按", Toast.LENGTH_SHORT).show();
            }
 ｝
```

