---
layout: post
title:  "Android 嵌套滑动"
date:   2016-06-18 00:29:40 +0800
categories: Android
---

Android 5.0版本增加了嵌套滑动机制，用来简化嵌套滑动需要处理的手势事件，来实现更复杂的滑动效果和控件。在5.0版本之前如果要嵌套滑动，由于嵌套滑动的滑动事件冲突问题，需要在手势相关的回调方法（dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent）里小心翼翼地进行逻辑判断，同时由于Android的事件传递机制，如果事件在一个View上被消费了，那它的子控件和父控件都不会再有机会处理该事件，比如在ScrollView里嵌套了ListView，当ListView滑动到底部的时候，ScrollView不能接着ListView的滑动速度继续向下滑。新增加的嵌套滑动机制就是用来解决这些事件消费问题的，当一个View滑动开始或结束的时候，他的父控件会得到通知，可以决定是否配合子控件进行滑动。这些是如何实现的呢？先看两个重要的类：

## NestedScrollingChild

NestedScrollingChild是一个接口，规定了嵌套滑动中滑动子控件需要实现的行为：

```java
public interface NestedScrollingChild {
    public void setNestedScrollingEnabled(boolean enabled);
    public boolean isNestedScrollingEnabled();
    public boolean startNestedScroll(int axes);
    public void stopNestedScroll();
    public boolean hasNestedScrollingParent();
    public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, int[] offsetInWindow);
    public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[] offsetInWindow);
    public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed);
    public boolean dispatchNestedPreFling(float velocityX, float velocityY);
}
```

整体的调用流程是，当View准备滑动的时候调用`startNestedScroll`来通知父控件做好滑动准备；滑动开始之前调用`dispatchNestedPreScroll`，父控件得到通知，决定是否先于子控件滑动一段距离，子控件根据返回的距离余量自己再进行滑动；如果子控件滑动结束后还有余量，就调用`dispatchNestedScroll`，让父控件决定是否消费剩下的余量；滑动结束后调用`stopNestedScroll`，通知父控件滑动结束。Fling 事件和 Scroll 事件有相同的逻辑。

有了调用流程，这些通知逻辑如何实现呢？Android 提供了这些逻辑的实现类`NestedScrollingChildHelper`，子控件只要调用`NestedScrollingChildHelper`中对应的方法就可以了。

```java
	@Override
    public void setNestedScrollingEnabled(boolean enabled) {
        mChildHelper.setNestedScrollingEnabled(enabled);
    }

    @Override
    public boolean isNestedScrollingEnabled() {
        return mChildHelper.isNestedScrollingEnabled();
    }

    @Override
    public boolean startNestedScroll(int axes) {
        return mChildHelper.startNestedScroll(axes);
    }

    @Override
    public void stopNestedScroll() {
        mChildHelper.stopNestedScroll();
    }

    @Override
    public boolean hasNestedScrollingParent() {
        return mChildHelper.hasNestedScrollingParent();
    }

    @Override
    public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed,
            int dyUnconsumed, int[] offsetInWindow) {
        return mChildHelper.dispatchNestedScroll(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed,
                offsetInWindow);
    }

    @Override
    public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[] offsetInWindow) {
        return mChildHelper.dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow);
    }

    @Override
    public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed) {
        return mChildHelper.dispatchNestedFling(velocityX, velocityY, consumed);
    }

    @Override
    public boolean dispatchNestedPreFling(float velocityX, float velocityY) {
        return mChildHelper.dispatchNestedPreFling(velocityX, velocityY);
    }

```

## NestedScrollingParent

`NestedScrollingParent`同样是一个接口，规定了嵌套滑动中滑动父控件需要实现的行为：

```java
public interface NestedScrollingParent {
	public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes);
	public void onNestedScrollAccepted(View child, View target, int nestedScrollAxes);
    public void onStopNestedScroll(View target);
    public void onNestedScroll(View target, int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed);
    public void onNestedPreScroll(View target, int dx, int dy, int[] consumed);
    public boolean onNestedFling(View target, float velocityX, float velocityY, boolean consumed);
    public boolean onNestedPreFling(View target, float velocityX, float velocityY);
    public int getNestedScrollAxes();
}

```

NestedScrollParent 主要用于配合 NestedScrollChild 来进行滑动操作，child 调用
 startNestedScroll 会回调到 Parent 中的 onStartNestedScroll ，这个方法用于当前的 parent 声明是否要配合 child 滑动，如果返回 false，那 child 会继续向上一级查找；如果返回 true，child 就会把这个 parent 记下来，同时回调 onNestedScrollAccept 让 parent 进行一些配置。

```java
public boolean startNestedScroll(int axes) {
        if (hasNestedScrollingParent()) {
            // Already in progress
            return true;
        }
        if (isNestedScrollingEnabled()) {
            ViewParent p = mView.getParent();
            View child = mView;
            while (p != null) {
                if (ViewParentCompat.onStartNestedScroll(p, child, mView, axes)) {
                    mNestedScrollingParent = p;
                    ViewParentCompat.onNestedScrollAccepted(p, child, mView, axes);
                    return true;
                }
                if (p instanceof View) {
                    child = (View) p;
                }
                p = p.getParent();
            }
        }
        return false;
}

```
onNestedPreScroll 对应 child 的 dispatchNestedPreScroll；  
onNestedScroll 对应 child 的 dispatchNestedScroll；  
onNestedPreFling 对应 child 的 dispatchNestedFling；  
onNestedFling 对应 child 的 dispatchNestedFling；

同样 Android 也提供了 NestedScrollingParentHelper 类，不过里面的方法很简单，具体逻辑需要 Parent 实现。

看一下 NestedScrollView 中对这些方法的实现：

```java
@Override
    public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes) {
        return (nestedScrollAxes & ViewCompat.SCROLL_AXIS_VERTICAL) != 0;
    }

    @Override
    public void onNestedScrollAccepted(View child, View target, int nestedScrollAxes) {
        mParentHelper.onNestedScrollAccepted(child, target, nestedScrollAxes);
        startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL);
    }

    @Override
    public void onStopNestedScroll(View target) {
        mParentHelper.onStopNestedScroll(target);
        stopNestedScroll();
    }

    @Override
    public void onNestedScroll(View target, int dxConsumed, int dyConsumed, int dxUnconsumed,
            int dyUnconsumed) {
        final int oldScrollY = getScrollY();
        scrollBy(0, dyUnconsumed);
        final int myConsumed = getScrollY() - oldScrollY;
        final int myUnconsumed = dyUnconsumed - myConsumed;
        dispatchNestedScroll(0, myConsumed, 0, myUnconsumed, null);
    }

    @Override
    public void onNestedPreScroll(View target, int dx, int dy, int[] consumed) {
        dispatchNestedPreScroll(dx, dy, consumed, null);
    }

    @Override
    public boolean onNestedFling(View target, float velocityX, float velocityY, boolean consumed) {
        if (!consumed) {
            flingWithNestedDispatch((int) velocityY);
            return true;
        }
        return false;
    }

    @Override
    public boolean onNestedPreFling(View target, float velocityX, float velocityY) {
        return dispatchNestedPreFling(velocityX, velocityY);
    }

    @Override
    public int getNestedScrollAxes() {
        return mParentHelper.getNestedScrollAxes();
    }

```

## 应用

了解这些基本概念有助于我们更好地处理嵌套滑动，一般来说我们不需要去实现这些方法，因为最新的 support 包和 design 包，NestedScrollView、RecyclerView、SwipeRefreshLayout 和 CoordinatorLayout 这些控件已经实现了 NestedScrollChild 和 NestedScrollParent，用这些控件可以满足大部分开发需求，比如在一个 NestedScrollView 中嵌套一个 RecyclerView。如果我们有一个可滑动的自定义 View ，需要嵌套其他滑动控价，就可以通过实现 NestedScrollChild 或者 NestedScrollParent 来进行嵌套滑动。

## 其他

目前的 NestedScrollView 和 RecyclerView 配合滑动的有问题，NestedScrollView 对嵌套的 fling 事件处理有 bug，导致滑动不流畅，有人给出了解决方法，[https://code.google.com/p/android/issues/detail?id=195800](https://code.google.com/p/android/issues/detail?id=195800)


## Reference

[https://segmentfault.com/a/1190000002873657](https://segmentfault.com/a/1190000002873657)

[http://www.race604.com/android-nested-scrolling/](http://www.race604.com/android-nested-scrolling/)
