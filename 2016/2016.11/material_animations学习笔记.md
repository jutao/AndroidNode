# 前言 #
关于转场动画之类的动画效果平时了解的不多，最近在 Github 上 看到了一个开源项目：[Material-Animations](https://github.com/lgvalle/Material-Animations "Material-Animations") 做的非常好，忍不住照着写了一遍，项目不大，但是完全弄懂需要花不少时间，这篇笔记记录了一些我从该项目中学到的知识点。

# 概述 #
Android Transition Framework 主要有三种效果：

1.不同 Activity 之间切换时,Activityc 的内容 (contentView) 转场动画。

2.不同 Activity 之间切换时，如果使用了 Shared Element 动画，也可以使用 Transition FrameWork 来实现不同的过渡动画效果。

3.同一个 Activity 内 View 变化的过渡动画 (Scene)。

下面我们分模块来讨论这几种效果。

# Activity 转场动画 #
首先要进行 ThemeStyle 设置:

	<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar"> 
	  ...
	  <!-- 允许使用transitions -->  
	  <item name="android:windowContentTransitions">true</item>  
	  <!--是否覆盖执行，其实可以理解成是否同步执行还是顺序执行-->  
	  <item name="android:windowAllowEnterTransitionOverlap">false</item>  
	  <item name="android:windowAllowReturnTransitionOverlap">false</item>
	</style>

# DataBinding #
Material-Animations 用到了 DataBinding，基本配置方式如下：
> 环境搭建：
> 
> Android 的 Gradle 插件版本不低于 1.5.0-alpha1：
> 
> classpath 'com.android.tools.build:gradle:1.5.0'
> 
> 然后修改对应模块（Module）的 build.grade：
> 
> 
> 	android {
> 	    ....
> 	    dataBinding {
> 	        enabled = true
> 	    }
> 	}
> 注意：Android stuido 的版本一定要大于1.3，而且Android Studio目前对binding对象没有自动代码提示，只会在编译时进行检查。
> 
> 就是这么简单，但是1.3及以前的版本，对于环境的搭建，可能就会麻烦一点（没事1.3的环境搭建方法，网上多得是）。


# RecyclerView 优化技巧 #
## 使用 setHasFixedSize ##
    //如果item的内容不改变view布局大小，那使用这个设置可以提高RecyclerView的效率
    //这里我们的item是不会改变的，所以用这个属性来优化自身效率
    recyclerView.setHasFixedSize(true);

# 零散的知识点 #
## Gravity.START 和 Gravity.LEFT 的区别  ##
left/right是代表一种绝对的对齐

start/end表示基于阅读顺序的对齐

目前存在的主要阅读方式： 从左向右(LTR)和从右向左(RTL)；

当使用left/right的时候，无论是LTR还是RTL，总是左/右对齐的；而使用start/end，在LTR中是左/右对齐，而在RTL中则是右/左对齐。 
注： left/right属于绝对对齐，而start/end会根据不同国家习惯改变。如阅读顺序是从左到右(LTR)的国家，start在左边，在阅读顺序是从右到左(RTL)的国家，start在右边。 

## Toolbar 去原生 title ##
    //不显示自带title
    if (getSupportActionBar() != null)
      getSupportActionBar().setDisplayShowTitleEnabled(false);