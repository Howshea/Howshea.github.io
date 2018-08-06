---
title: DrawerLayout 和 NavigationView 的使用细节
date: 2016-10-13 11:41
tags: Android
categories: 技术
---

昨天想弄个侧边栏，结果踩了很多坑，`DrawerLayout`和`NavigationView`配合可以很容易的实现侧边栏，我也用过不少次了，但是之前都是按照模板来写，没出过什么差错，现在我的想法是用activity来托管fragment，而所有的布局包括`Toolbar`、`DrawerLayout`都放在fragment的布局文件里，结果就踩了很多坑。
<!--more-->
## DrawLayout在fragment中使用
1. 写了一个简单的Demo，这是fragment布局文件：

	```xml  
	<android.support.v4.widget.DrawerLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    android:id="@+id/drawer_layout"
	    android:layout_width="match_parent"
	    android:clickable="true"
	    android:layout_height="match_parent">
	    <LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:orientation="vertical"
	        >
	        <android.support.v7.widget.Toolbar
	            android:id="@+id/toolbar"
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:background="@color/colorPrimary"
	            android:elevation="8dp"
	            app:title="@string/app_name">
	        </android.support.v7.widget.Toolbar>
	        <ImageView
	            android:layout_width="match_parent"
	            android:layout_height="match_parent"
	            android:src="@mipmap/ic_launcher"/>
	    </LinearLayout>
	    <android.support.design.widget.NavigationView
	        android:id="@+id/navigation_view"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:layout_gravity="start"
	        app:elevation="16dp"
	        app:menu="@menu/navigation_menu"/>
	</android.support.v4.widget.DrawerLayout>
	```
2. 在fragment中`onCreateView()`方法中设置：  

	```java
	AppCompatActivity activity = (AppCompatActivity) getActivity();
	        activity.setSupportActionBar(mToolbar);
	        activity.getSupportActionBar().setHomeButtonEnabled(true);
	        activity.getSupportActionBar().setDisplayShowHomeEnabled(true);
	        ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(activity, mDrawerLayout, 
	mToolbar, R.string.open, R.string.close);
	        toggle.syncState();
	        mDrawerLayout.addDrawerListener(toggle);
	```
3. 注意几个点：
   - drawerlayout的设置要放在fragment的`onCreate()`之后，否则会报空指针
   - NavigationView的`layout_gravity`属性设置成left的话编译有时候会报错，有时候又正常，不知道为什么，总之设置成start是没有问题的

## 关于NavigationView
1. **主界面要显示的内容要包含在DrawerLayout中**，否则会挡住NavigationView的显示
2. NavigationView必须是DrawerLayout的次一级的子布局，就是**不能在NavigationView外面在加一层布局**
3. 如果不想让侧边栏遮住Toolbar，DrawerLayout要写在Toolbar的外面


## 其它解决方案
其实完全可以把侧边栏写在Activity中，如果要响应fragment的操作，**可以由fragment丢出一个接口**，这样就可以实现了。
不过这样似乎违背了activity只用来托管的想法，还是看项目具体怎么规划
   