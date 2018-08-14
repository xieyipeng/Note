1、获取权限

<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

2、点击按钮来提示选择图库还是相机

private String[]mCustomItems=new String[]{"本地相册","相机拍照"};

//显示选择相机，图库对话框

```java
      private void showDialogCustom(){
        //创建对话框
        AlertDialog.Builder builder=new AlertDialog.Builder(MainActivity.this);
        builder.setTitle("请选择：");
        builder.setItems(mCustomItems, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                if(which==0) {
                    //相册
 
                }else if(which==1){
                    //照相机
                    
                }
            }
        });
        builder.create().show();
    }

```

3、相机拍照

//拍照后照片的Uri

private Uri imageUri;

```
//返回码，相机
private static final int RESULT_CAMERA=200;
```

 

```java
                     //先验证手机是否有sdcard 
                    String status=Environment.getExternalStorageState();
                    if(status.equals(Environment.MEDIA_MOUNTED))
                    {
                        //创建File对象，用于存储拍照后的照片
                        File outputImage=new File(getExternalCacheDir(),"out_image.jpg");//SD卡的应用关联缓存目录
                        try {
                            if(outputImage.exists()){
                                outputImage.delete();
                            }
                            outputImage.createNewFile();
                            Intent intent=new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
                            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
                                imageUri=FileProvider.getUriForFile(MainActivity.this,
                                        "com.hanrui.android.fileprovider",outputImage);//添加这一句表示对目标应用临时授权该Uri所代表的文件
                            }else{
                                imageUri=Uri.fromFile(outputImage);
                            }
                            //启动相机程序
                            intent.putExtra(MediaStore.Images.Media.ORIENTATION, 0);
                            intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
                            startActivityForResult(intent,RESULT_CAMERA);
                        } catch (Exception e) {
                            // TODO Auto-generated catch block 
                            Toast.makeText(MainActivity.this, "没有找到储存目录",Toast.LENGTH_LONG).show();
                        }
                    }else{
                        Toast.makeText(MainActivity.this, "没有储存卡",Toast.LENGTH_LONG).show();
                    }
                    dialog.dismiss();
```

 

应用关联缓存目录指SD卡中专门用于存放当前应用缓存数据的位置，调用getExternalCacheDir()方法可以得到这个目录，具体路径是/sdcard/Android/data/<package name>/cache。为什么要存放这个目录呢？因为从Android6.0系统开始，读写SD卡被列为危险权限，若将图片存放在SD卡的任何其他目录，都要进行权限处理才行，使用应用关联目录可以跳过这一步。

接着进行一个判断，若设备版本低于Android7.0，就调用Uri的fromFile()方法将File对象转换成Uri对象，这个Uri对象标识图片的本地真实路径。否则，就调用FileProvider的getUriForFile()方法将File对象转换成一个封装过的Uri对象。getUriForFile()方法接收3个参数，第一个要求传入Context对象，第二个可以是任意字符串，第三个是我们刚刚创建的File对象。进行这样一层转换，因为从Android7.0开始，直接使用本地真实路径的Uri被认为是不安全的，会抛出一个FileUriExposedException异常。而FileProvider则是一种特殊的内容提供器，它使用了和内容提供器类似的机制来对数据进行保护，可以选择性的将封装过的Uri共享给外部，从而提高了应用的安全性。

在AndroidManifest.xml中对内容提供器进行注册

```java
<manifest

...>

<application

.....>

<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.example.android.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths">
    </meta-data>
</provider>
....

</application>

</manifest>
```

android:name属性值是固定的，android:authorities属性的值必须要和刚才FileProvider.getUriForFile()方法中的第二个参数一致。另外，在<provider>标签的内部使用<mata-data>来指定Uri的共享路径，并引用了一个@xml/file_paths资源，下面创建这个资源：

在res目录下创建一个xml目录，在xml目录下创建一个file_paths.xml文件，内容如下：

```
<?xml version="1.0" encoding="utf-8"?>
<path xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path=""/>
</path>
```

 

external-path就是用来指定Uri共享的，name属性的值可以随便填，path属性的值表示共享的具体路径，设置空值表示将整个SD卡进行共享。当然也可以仅共享我们存放照片的路径。

回调方法对图片进行裁剪

```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
switch (requestCode){
 case RESULT_CAMERA:
                if(resultCode==RESULT_OK){
                    //进行裁剪
                    startPhotoZoom(imageUri);
                }
        break;
	}
}
```

 

 

4、调用图库

```
//返回码，本地图库
private static final int RESULT_IMAGE=100;
```

 

```java
//相册
                    if(ContextCompat.checkSelfPermission(MainActivity.this,Manifest.permission.WRITE_EXTERNAL_STORAGE)!=
                            PackageManager.PERMISSION_GRANTED){
                        ActivityCompat.requestPermissions(MainActivity.this,new String[]{
                                Manifest.permission.WRITE_EXTERNAL_STORAGE},1);
                    }else{
                        openAlbum();
                    }

```

打开相册

 

```java
private void openAlbum(){
        Intent intent=new Intent("android.intent.action.GET_CONTENT");
        intent.setType("image/*");
        startActivityForResult(intent,RESULT_IMAGE);//打开相册
    }
```

 

```java
@Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        switch (requestCode){
            case 1:
                if(grantResults.length>0&&grantResults[0]==PackageManager.PERMISSION_GRANTED){
                    openAlbum();
                }else{
                    Toast.makeText(this,"你没有开启权限",Toast.LENGTH_SHORT).show();
                }
                break;
            default:
                break;
        }
    }
```

 

调用相册时先进行一个运行时权限处理，动态申请WRITE_EXTERNAL_STORAGE这个危险权限。因为相册照片存放在SD卡中，从SD卡读取照片需要申请这个权限。WRITE_EXTERNAL_STORAGE表示同时授予程序对SD卡读和写的能力。

回调方法

```java
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
case RESULT_IMAGE:
                if(resultCode==RESULT_OK&&data!=null){
                    //判断手机系统版本号
                    if(Build.VERSION.SDK_INT>=19){
                        //4.4及以上系统使用这个方法处理图片
                        handlerImageOnKitKat(data);
                    }else{
                        //4.4以下系统使用这个方法处理图片
                        handlerImageBeforeKitKat(data);
                    }
                }
                break;
}
```

 

Android系统从4.4版本开始，选取相册中图片不再返回图片真实的Uri了，而是一个封装过的Uri，隐因此4.4版本以上要对这个Uri进行解析。

```java
private void handlerImageOnKitKat(Intent data){
        String imagePath=null;
        Uri uri=data.getData();
        if(DocumentsContract.isDocumentUri(this,uri)){
            //如果是document类型的Uri,则通过document id处理
            String docId=DocumentsContract.getDocumentId(uri);
            if("com.android.providers.media.documents".equals(uri.getAuthority())){
                String id=docId.split(":")[1];//解析出数字格式的id
                String selection=MediaStore.Images.Media._ID+"="+id;
                imagePath=getImagePath(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,selection);
            }else if("com.android.providers.downloads.documents".equals(uri.getAuthority())){
                Uri contentUri= ContentUris.withAppendedId(Uri.parse("content://downloads/public_downloads"),Long.valueOf(docId));
                imagePath=getImagePath(contentUri,null);
            }
        }else if("content".equalsIgnoreCase(uri.getScheme())){
            //如果是content类型的URI，则使用普通方式处理
            imagePath=getImagePath(uri,null);
        }else if("file".equalsIgnoreCase(uri.getScheme())){
            //如果是file类型的Uri,直接获取图片路径即可
            imagePath=uri.getPath();
        }
        startPhotoZoom(uri);
    }
```

```java
private String getImagePath(Uri uri, String selection) {
        String path = null;
        //通过Uri和selection来获取真实的图片路径
        Cursor cursor = getContentResolver().query(uri, null, selection, null, null);
        if (cursor != null) {
            if (cursor.moveToFirst()) {
                path = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA));
            }
            cursor.close();
        }
        return path;
    }
```

 

 

如果返回的Uri是document类型的话，就取出doculent_id进行处理，若不是，就是用普通方法处理。另外，若Uri的authority是media格式的话，documentid还需要进行一次解析，要通过字符串分割的方式取出后半部分才能得到真正的数字id。取出的id用于构建新的Uri和条件语句，然后把这些值作为参数传入到getImagePath()方法中，就可以获取图片的真实路径了。

```java
private void handlerImageBeforeKitKat(Intent data){
        Uri cropUri=data.getData();
        startPhotoZoom(cropUri);
    }
```

直接取出Uri进行裁剪。

5、图片裁剪的方法

```
private static final int CROP_PICTURE = 2;//裁剪后图片返回码
//裁剪图片存放地址的Uri
private Uri cropImageUri;
```

 

```java
public void startPhotoZoom(Uri uri) {
        File CropPhoto=new File(getExternalCacheDir(),"crop_image.jpg");
        try{
            if(CropPhoto.exists()){
                CropPhoto.delete();
            }
            CropPhoto.createNewFile();
        }catch(IOException e){
            e.printStackTrace();
        }
        cropImageUri=Uri.fromFile(CropPhoto);
        Intent intent = new Intent("com.android.camera.action.CROP");
        intent.setDataAndType(uri, "image/*");
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION); //添加这一句表示对目标应用临时授权该Uri所代表的文件
        }
        // 下面这个crop=true是设置在开启的Intent中设置显示的VIEW可裁剪
        intent.putExtra("crop", "true");
        intent.putExtra("scale", true);
 
        intent.putExtra("aspectX", 1);
        intent.putExtra("aspectY", 1);
 
        intent.putExtra("outputX", 300);
        intent.putExtra("outputY", 300);
 
        intent.putExtra("return-data", false);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, cropImageUri); 
        intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
        intent.putExtra("noFaceDetection", true); // no face detection
        startActivityForResult(intent, CROP_PICTURE);
    }
```

 

| 附加选项                           | 数据类型   | 描述                         |
| ---------------------------------- | ---------- | ---------------------------- |
| crop                               | String     | 发送裁剪信号                 |
| aspectX                            | int        | X方向上的比例                |
| aspectY                            | int        | Y方向上的比例                |
| outputX                            | int        | 裁剪区的宽                   |
| outputY                            | int        | 裁剪区的高                   |
| scale                              | boolean    | 是否保留比例                 |
| return-data                        | boolean    | 是否将数据保留在Bitmap中返回 |
| data                               | Parcelable | 相应的Bitmap数据             |
| circleCrop                         | String     | 圆形裁剪区域？               |
| MediaStore.EXTRA_OUTPUT ("output") | URI        | 将URI指向相应的file:///...   |

 

```java
 @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
case CROP_PICTURE: // 取得裁剪后的图片
                if(resultCode==RESULT_OK) {
                    try {
                        Bitmap headShot = BitmapFactory.decodeStream(getContentResolver().openInputStream(cropImageUri));
                       .....
} catch (FileNotFoundException e) {
                        e.printStackTrace();
                    }
                }
 
                break;
            default:
                break;
        }
    }
```

 

 

到此，获得裁剪图片结束。

还有一个问题：三星手机自带的相机会转屏而引起当前activity销毁，重建。

我们可以在当前Activity的manifest文件里添加这一行代码：

android:configChanges="orientation|keyboardHidden| screenSize"

关键就是添加screenSize

然后在Activityl里添加以下代码：

```java
@Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
    }
```

好了，到此结束。

 