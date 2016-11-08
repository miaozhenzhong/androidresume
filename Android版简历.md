    Android版简历（一）
    简介
    本人是二本应届毕业生，由于近来找工作，有的公司比较挑学校，好几次都没有笔试的机会。所以我决定弄点让人眼前一亮的东西，因为我个人比较喜欢Android开发（个人觉得我的J2EE比较好，Android只是能开发的程度，深入理解的还不够），所以就萌生了做一个Android版的简历的念头。
    预览
    因为我本来就是要应聘的，所以信息什么的我也就不隐藏了。另外说一句，我不是设计师，不是美工，所以界面如果丑了点，希望大家也不要见怪。
      
     

    涉及技术
    Menu创建、调用短信和电话、布局、处理图片
    第一步：创建主页面
    这篇文章不是教大家怎么入门Android，所以我就假设大家已经有了一定的Android水平了。我们首先创建一个Activity -- MainActivity，开始对其布局。
    主界面就是预览中的第一张带有我照片的图片，我们要达到这样的效果：文字保持在左下角，图片保持在中间靠上的部分。
    这个布局很容易实现，最外层使用RelativeLayout，将最后一排文字TextView放在整个布局的最下面最左边，其他的文字一条条的设置在这排文字的上面；头像则让ImageView居中，并且marginTop设置大概40dp就行了。
    这样我们的布局就出来了。至于文字的样式就由大家自己去调节，我觉得白色就挺好，个人比较喜欢简单的颜色，呵呵。
    这个布局文件是activity_main.xml，大家可以去查看具体代码。
    第二步：圆形照片
    接下来我们要处理我们的照片了。为了保证照片的质量，我们的照片不使用拉伸，在面对不同分辨率的设备时，我们使用不同的照片这样就行了。话不多说，来处理照片吧。
    我先来讲一下怎么处理成为圆形的照片，原理很简单，就是用一个Canvas导入一个Bitmap，在Canvas中截取一个圆形，再将我们的照片放置到里面。
    首先我们需要对照片的高和宽进行一定的处理，并且要找到照片的中心，截取一个最大的圆形，再用圆形与照片取交集就能获得圆形照片了。
    [java] view plain copy
    在CODE上查看代码片派生到我的代码片

        //将Bitmap处理为圆形的图片  
            private Bitmap circlePic(Bitmap bitmap){  
                int width = bitmap.getWidth();  
                int height = bitmap.getHeight();  
                int r = width < height ? width/2:height/2; //圆的半径，取宽和高中较小的，以便于显示没有空白  
                Bitmap outBitmap = Bitmap.createBitmap(r*2, r*2, Bitmap.Config.ARGB_8888); //创建一个刚好2r大小的Bitmap  
                Canvas canvas = new Canvas(outBitmap); // 这里的这个Canvas就是我们用来绘制照片的Canvas  
                final int color =0xff424242;  
                final Paint paint = new Paint();  
                /** 
                 * 截取图像的中心的一个正方形,用于在原图中截取 
                 * 坐标如下： 
                 * 1.如果 w < h , 左上坐标(0, (h-w)/2) , 右上坐标(w, (h+w)/2) 
                 * 2.如果 w > h , 左上坐标((w-h)/2, 0) , 右上坐标((w+h)/2, h) 
                 */  
                final Rect rect = new Rect( width < height ? 0 : (width-height)/2, width < height ? (height-width)/2 : 0,  
                        width < height ? width : (width+height)/2, (width < height ? (height+width)/2 : height));  
                //创建一个直径大小的正方形，用于设置canvas的显示与设置画布截取  
                final Rect rect2 = new Rect( 0, 0, r*2, r*2);  
                //提高精度，用于消除锯齿  
                final RectF rectF = new RectF(rect2);  
                //下面是设置画笔和canvas  
                paint.setAntiAlias(true);  
                canvas.drawARGB(0,0,0,0);  
                paint.setColor(color);  
                //设置圆角，半径都为r,大小为rect2,这样绘制出来的就是一个圆形了  
                canvas.drawRoundRect(rectF, r, r, paint);  
                //设置图像重叠时的显示方式,这个设置很重要,设置了这个属性之后我们画照片进去就会以交集的形式截取,这样就会绘制出圆形的图片了  
                paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));  
                //绘制图像到canvas  
                canvas.drawBitmap(bitmap, rect, rect2, paint);  
                return outBitmap;  
            }  

    这样圆形的图片就处理好了。大笑
    我们只需要将处理好了的图片设置到我们定义的控件中就可以了，具体代码如下：
    [java] view plain copy
    在CODE上查看代码片派生到我的代码片

        ImageView myPhoto = (ImageView) this.findViewById(R.id.myPhoto);  
        //my_photo.setImageResource(R.drawable.ic_launcher);  
        //得到Resources对象  
        Resources r = getResources();  
        WindowManager manage=getWindowManager();  
        Display display=manage.getDefaultDisplay();  
        int screenWidth=display.getWidth();  
        //以数据流的方式读取资源  
        InputStream is = null;  
        if(screenWidth > 480) {//这段代码是我适应不同的屏幕大小来显示了不同的照片，其实也有不同的处理方式，不过我觉得这样也能满足我的需求  
            is = r.openRawResource(R.drawable.my_photo);  
        } else {  
            is = r.openRawResource(R.drawable.my_photo1);  
        }  
        //获取到BitmapDrawable对象  
        BitmapDrawable  bmpDraw = new BitmapDrawable(is);  
        //获得Bitmap  
        Bitmap bmp = bmpDraw.getBitmap();  
        myPhoto.setImageBitmap(circlePic(bmp));//绘制好的圆形照片设置到ImageView中  
        myPhoto.setOnClickListener(new PhotoClickListener());//设置一个点击事件，后面再说  

    好了，我们已经绘制了圆形的图片和整体的布局了。也就是，我们做到了这一个画面：
    结尾
    下一篇，我们将会创建菜单和下一个界面。
    关于这个项目，我已经托管到了Github上，大家可以去下载浏览源码，我也会继续完善这个项目。
    托管地址：https://github.com/xjyaikj/androidresume
