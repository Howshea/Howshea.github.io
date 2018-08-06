---
title: SwipeRefreshLayout 配合 RecyclerView 实现瀑布流上拉下拉刷新
date: 2016-10-17 21:36
tags: Android
categories: 技术
---
谷歌良心地推出了SwipeRefreshLayout来实现下拉刷新，然而没有做上拉刷新地功能，所以想要上拉刷新就要自己实现，由于瀑布流是好几列，一时间没有思路，就先在网上查了一下，但是发现别人写的实现RecyclerView的上拉刷新的代码都很繁琐，于是自己研究了一下RecyclerView的一些方法，其实还蛮好实现的。
<!--more-->
## 首先是一些基本用法
1. 布局的代码：
	
	```xml
	<android.support.v4.widget.SwipeRefreshLayout
    	xmlns:android="http://schemas.android.com/apk/res/android"
    	xmlns:app="http://schemas.android.com/apk/res-auto"
    	android:id="@+id/refresh_layout"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent"
    	>
    	<android.support.v7.widget.RecyclerView
        	android:id="@+id/recycler_view"
        	android:layout_width="match_parent"
        	android:layout_height="match_parent"
        />
    </android.support.v4.widget.SwipeRefreshLayout>
	```
2. 下拉刷新的实现，很简单，实现`SwipeRefreshLayout`的方法`onRefresh()`，记得刷新操作完成之后要关掉刷新的圈圈

	```java
	mRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                //这里实现一些网络请求等操作
                //这是关闭刷新，不过最好不要写在这，而是写在异步完成之后的地方
				mRefreshLayout.setRefreshing(false);
            }
        });
	```
附一张效果图：
![](http://ww2.sinaimg.cn/large/8127619agw1f8vmuw8j9yg20g00sgkjn.gif)

## 实现RecyclerView的上拉刷新
`RecyclerView`有个`OnScrollListener`，其中有两个方法，`onScrollStateChanged(···)`和`onScrolled(···)`,分析一下上拉的动作，是手指在屏幕上往上滑到列表结束，停了下来，即滑动的状态发生了改变，所以我们只要实现`onScrollStateChanged(···)`即可
代码如下：  

```java
mRecyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                //判断滑动状态是否为停止状态
                if (newState == RecyclerView.SCROLL_STATE_IDLE) {
                    int lastIndex = 0;
                    int[] ints = mLayoutManager.findLastVisibleItemPositions(null);
                    lastIndex = ints[0] > ints[1] ? ints[0] : ints[1];
                    if (lastIndex + 1 == mAdapter.getItemCount()) {
                        if (!mRefreshLayout.isRefreshing()) {
                            mRefreshLayout.setRefreshing(true);
                        }
                        //这个位置写上要实现的业务
                    }

                }

            }
        });
```
* 对于多列的RecyclerView瀑布流，`StaggeredGridLayoutManager`提供了一个`findLastVisibleItemPosition()`方法，参数是一个数组，用来存储得到的数据，可为空，空的话就返回一个新的数组，这个数组的长度就是瀑布流的列数，上面的代码以两列为例，数组的第一位是第一列最下面的视图索引值，数组的第二位即是第二列最下面的视图的索引值，接着比较出最大的那个，记得要加一，因为是从0开始数的，然后跟已有的总数比较，如果相等，说明已经拉到底了，这个时候就可以刷新数据了，这个时候把`SwipeRefreshLayout`的刷新的效果打开，获得更好的交互体验。