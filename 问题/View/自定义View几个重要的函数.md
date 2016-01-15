# 自定义View几个重要的函数

简单讲解自定义View相关的几个重要的函数。

### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

作为一个<b>有(hui)追(zhuang)求(B)</b>的程序员，肯定想做一些让人眼前一亮的程序效果，但是系统提供的那些一般很难满足，为了<b>梦(zhuang)想(B)</b>就必须要学习一些自定义View。下面我们就了解一些自定义View相关的东西。

## 一.自定义View分类

### 我将自定义View分为了两类：
  
###  1.自定义ViewGroup
  
  自定义ViewGroup是利用现有的组件根据特定的布局方式来组成新的组件，一般继承自ViewGroup或各种Layout。
  
  例如：一个应用内的底部导航条中的条目，一般都是上面为图标，下面是文字，那么这两个就可以用自定义ViewGroup组合成为一个Veiw，提供两个属性分别用来设置文字和图片即可，这样使用起来会方便很多。
    
###  2.自定义View
  
  在没有现成的View，需要自己实现的时候，就使用自定义View，一般继承自View，SurfaceView或其他的View。
  
  例如：定义一个支持自动加载网络图片的ImageView，或制作一种特殊的动画效果。

  <b>一般来说，自定义View在大多数情况下都有替代方案，利用图片或者组合动画来实现，但是使用后者可能会面临内存耗费过大，制作麻烦更诸多问题。</b>
  
##  二.几个重要的函数

### 1.构造函数
  View的构造函数有四种重载分别如下
``` java
  public void SloopView(Context context) {}
  public void SloopView(Context context, AttributeSet attrs) {}
  public void SloopView(Context context, AttributeSet attrs, int defStyleAttr) {}
  public void SloopView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {}
```
  可以看出，关于View构造函数的参数有多有少，先排除几个不常用的，留下常用的再研究。
  
  <b>有四个参数的构造函数在API21的时候才添加上，我一般不使用，暂不考虑。</b>
  
有三个参数的构造函数中第三个参数是默认的Style，这里的默认的Style是指它在当前Application或Activity所用的Theme中的默认Style，且只有在明确调用的时候才会生效，以系统中的ImageButton为例说明：
``` java
    public ImageButton(Context context, AttributeSet attrs) {
        //调用了三个参数的构造函数，明确指定第三个参数
        this(context, attrs, com.android.internal.R.attr.imageButtonStyle);
    }

    public ImageButton(Context context, AttributeSet attrs, int defStyleAttr) {
        //此处调了四个参数的构造函数，无视即可
        this(context, attrs, defStyleAttr, 0); 
    }
```
<b>注意：即使你在View中使用了Style这个属性也不会调用三个参数的构造函数，所调用的依旧是两个参数的构造函数。</b>  

<b>由于三个参数的构造函数第三个参数一般不用，暂不考虑，第三个参数的具体用法会在以后用到的时候详细介绍。</b>

排除了两个之后，只剩下一个参数和两个参数的构造函数，他们的详情如下：
``` java
  //一般在直接New一个View的时候调用。
  public void SloopView(Context context) {}
  
  //一般在layout文件中使用的时候会调用，关于它的所有属性(包括自定义属性)都会包含在attrs中传递进来。
  public void SloopView(Context context, AttributeSet attrs) {}
```
以下方法调用的是<b>一个参数</b>的构造函数：
``` java
  //在Avtivity中
  SloopView view = new SloopView(this);
```
以下方法调用的是<b>两个参数</b>的构造函数：
``` xml
  //在layout文件中 - 格式为： 包名.View名
  <com.sloop.study.SloopView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```
关于构造函数先讲这么多，关于如何自定义属性和使用attrs中的内容，在后面会详细讲解，目前只需要知道这两个构造函数在何时调用即可。

关于构造函数，如果你想了解更多，可以参考以下文章：<br/>
[Android中自定义样式与View的构造函数中的第三个参数defStyle的意义](http://www.cnblogs.com/angeldevil/p/3479431.html) <br/>
[android view构造函数研究](http://blog.csdn.net/z103594643/article/details/6755017)<br/>
[Android View构造方法第三参数使用方法详解](http://blog.csdn.net/mybeta/article/details/39993449)<br/>

### 2.测量View大小
测量View大小使用的是onMeasure函数，函数中有两个参数，我们可以从这两个参数取出宽高的相关数据，如下：
``` java
   @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int widthsize = MeasureSpec.getSize(widthMeasureSpec);      //取出宽度的确切数值
        int widthmode = MeasureSpec.getMode(widthMeasureSpec);      //取出宽度的测量模式
        
        int heightsize = MeasureSpec.getSize(heightMeasureSpec);    //取出高度的确切数值
        int heightmode = MeasureSpec.getMode(heightMeasureSpec);    //取出高度的测量模式
    }
```
使用MeasureSpec.getSize()获取的是一个精确数值，在用指定了大小或者使用match_parent, fill_parent的时候可以直接获取数值。
