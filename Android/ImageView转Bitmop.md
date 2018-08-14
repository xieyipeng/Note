imageView.getBackground()，是获取它的背景图片；

在调用getDrawingCache的时候要注意下面2点：

1. 在调用getDrawingCache()方法从ImageView对象获取图像之前，一定要调用setDrawingCacheEnabled(true)方法：

   imageview.setDrawingCacheEnabled(true);

   否则，无法从ImageView对象iv_photo中获取图像；

2. 在调用getDrawingCache()方法从ImageView对象获取图像之后，一定要调用setDrawingCacheEnabled(false)方法：

   imageview.setDrawingCacheEnabled(false);

   以清空画图缓冲区，否则，下一次从ImageView对象iv_photo中获取的图像，还是原来的图像。