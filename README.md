## Android：启动页--最佳实践

[TOC]

### 一、前言

> Android 开发过程中启动页是必不可少的，但是我们经常会看到启动打开后是先白屏或者黑屏，然后才会显示出启动页的图片，本文会解析此现象的原因，以及给出解决方案

#### 1.1 启动白屏或黑屏的原因

`AndroidManifest.xml`中的`application`标签中设置了**theme**，当设置的theme是`Light`类型时启动**白屏**，当设置theme是`Dark`类型时启动**黑屏**；

#### 1.2 进一步分析

当系统启动一个app时，**zygote**进程会fork一个app子进程，进程创建后在启动activity时就会创建一个`window`，这个window会使用`theme`中设置的**windowBackground**来显示背景颜色或者图片，当使用`Light`或`Dark`时跟进代码就能看到默认设置的`windowBackground`就是白色和黑色。

###二、启动页--常规做法

#### 2.1 创建Activity：SplashActivity

```kotlin
class SplashActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_splash)
        Handler().postDelayed({
            startActivity(Intent(this,MainActivity::class.java))
            finish()
        },3000)
    }
}
```



#### 2.2 给启动页创建background：splash_background.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@android:color/white"/>
    <item>
        <bitmap android:gravity="center"
                android:src="@drawable/splash_screen"/>
    </item>
</layer-list>
```

下面重点说下**bitmap**的属性功能：

* android:antialias

  *布尔值*。启用或停用抗锯齿。

* android:dither

  *布尔值*。当位图的像素配置与屏幕不同时（例如：ARGB 8888 位图和 RGB 565 屏幕），启用或停用位图抖动。

* android:filter

  *布尔值*。启用或停用位图过滤。当位图收缩或拉伸以使其外观平滑时使用过滤。

* **android:gravity**

  定义位图的重力。重力指示当位图小于容器时，可绘制对象在其容器中放置的位置。

  必须是以下一个或多个（用 '|' 分隔）常量值：

  | 值                  | 说明                                                         |
  | ------------------- | ------------------------------------------------------------ |
  | `top`               | 将对象放在其容器顶部，不改变其大小。                         |
  | `bottom`            | 将对象放在其容器底部，不改变其大小。                         |
  | `left`              | 将对象放在其容器左边缘，不改变其大小。                       |
  | `right`             | 将对象放在其容器右边缘，不改变其大小。                       |
  | `center_vertical`   | 将对象放在其容器的垂直中心，不改变其大小。                   |
  | `fill_vertical`     | 按需要扩展对象的垂直大小，使其完全适应其容器。               |
  | `center_horizontal` | 将对象放在其容器的水平中心，不改变其大小。                   |
  | `fill_horizontal`   | 按需要扩展对象的水平大小，使其完全适应其容器。               |
  | `center`            | 将对象放在其容器的水平和垂直轴中心，不改变其大小。           |
  | `fill`              | 按需要扩展对象的垂直大小，使其完全适应其容器。这是默认值。   |
  | `clip_vertical`     | 可设置为让子元素的上边缘和/或下边缘裁剪至其容器边界的附加选项。裁剪基于垂直重力：顶部重力裁剪上边缘，底部重力裁剪下边缘，任一重力不会同时裁剪两边。 |
  | `clip_horizontal`   | 可设置为让子元素的左边和/或右边裁剪至其容器边界的附加选项。裁剪基于水平重力：左边重力裁剪右边缘，右边重力裁剪左边缘，任一重力不会同时裁剪两边。 |

* android:mipMap

  *布尔值*。启用或停用 mipmap 提示。如需了解详细信息，请参阅 `setHasMipMap()`。默认值为 false。

* **android:tileMode**

  定义平铺模式。当平铺模式启用时，位图会重复。重力在平铺模式启用时将被忽略。

  必须是以下常量值之一：

  | 值         | 说明                                                         |
  | ---------- | ------------------------------------------------------------ |
  | `disabled` | 不平铺位图。这是默认值。                                     |
  | `clamp`    | 当着色器绘制范围超出其原边界时复制边缘颜色                   |
  | `repeat`   | 水平和垂直重复着色器的图像。                                 |
  | `mirror`   | 水平和垂直重复着色器的图像，交替镜像图像以使相邻图像始终相接。 |



#### 2.3 给启动页创建style：SplashTheme

```xml
<style name="SplashTheme" parent="AppTheme">
  <item name="windowNoTitle">true</item><!--无标题-->
  <item name="android:windowFullscreen">true</item><!--全屏-->
  <item name="android:windowIsTranslucent">true</item><!--半透明-->
  <item name="android:windowBackground">@drawable/splash_background</item>
</style>
```

#### 2.4 在AndroidManifest.xml中给SplashActivity设置style

```xml
<activity android:name=".SplashActivity"
          android:theme="@style/SplashTheme">
  <intent-filter>
    <action android:name="android.intent.action.MAIN"/>

    <category android:name="android.intent.category.LAUNCHER"/>
  </intent-filter>
</activity>
```

通过上面的4步设置就能够解决打开app白屏或黑屏的问题了。

### 三、最佳实践

#### 3.1 场景1：只有屏幕中间有一张图片或者是图片+文字

通过上面的方式就能实现，注意bitmap使用`gravity`为`center`让图片和文字`居中`显示

#### 3.2 场景2：屏幕中部和底部都有图片或者文字

> 这种上下布局的，例如微博，可以把图片切成上下两个，或者切割图片上下左右使图片整体能够居中显示

方式一：将图片切成屏幕中央显示的图片和底部图片

splash_background.xml中就需要换种写法了

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 整体的背景颜色 -->
    <item>
        <color android:color="@color/white"/>
    </item>
    <!-- 顶部 -->
    <item>
        <bitmap android:gravity="center"
                android:src="@drawable/splash_center"/>
    </item>
    <!-- 底部 -->
    <item>
        <bitmap android:gravity="bottom|center_horizontal"
                android:src="@drawable/splash_bottom"/>
    </item>
</layer-list>
```

当手机**底部**有**虚拟导航栏**时，底部图片会被遮挡，使导航栏**透明**即可：

```kotlin
class SplashActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION)
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_splash)
        Handler().postDelayed({
            startActivity(Intent(this,MainActivity::class.java))
            finish()
        },3000)
    }
}
```

这句**window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION)**是关键



方式二：将图片设置成background而不是windowBackground

* style设置：

```xml
<style name="SplashTheme" parent="AppTheme">
  <item name="windowNoTitle">true</item>
  <item name="android:windowFullscreen">true</item>
  <item name="android:windowIsTranslucent">true</item>
  <item name="android:windowBackground">@android:color/white</item>
  <item name="android:background">@drawable/splash_background</item>
</style>
```

用background来设置背景图片

* splash_background.xml设置：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@android:color/white"/>
    <item>
        <bitmap android:gravity="clip_vertical|clip_horizontal"
                android:src="@drawable/splash_screen"/>
    </item>
</layer-list>
```

利用bitmap的gravity属性，用**clip_vertical|clip_horizontal**来裁剪图片

**注意：利用第二种方式时不能使用window.addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION)，这个会导致图片被拉伸**









