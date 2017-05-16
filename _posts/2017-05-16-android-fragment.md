---
layout: post
title: "Android Fragment"
date: 2017-05-16 18:19:26 +0800
categories: Android
---

Android 在 3.0 版本添加了 Fragment，用来解决屏幕碎片化的问题，例如平板和手机两种尺寸差异较大的设备，通过 Fragment，可以根据屏幕尺寸选择性加载要显示的视图，业务逻辑都封装在对应的 Fragment 中，达到代码复用的目的。

## Fragment 与 View

- Fragment 依附在 FragmentActivity 上，通过 FragmentManager 进行管理，具有了一系列与 Activity 相关的生命周期；
- Fragment 与 View 最大的区别在于 Fragment 拥有更多的生命周期，Android 系统可以对他进行更细致的管理，View 除了本身的绘制逻辑（*onMeasur、onLayout、onDraw*）之外，暴露出来的只有 *onFinishInflate*、*attachToWindow* 和 *detachFromWindow* 等。同时由于 Fragment 生命周期函数增多，也带来了更多的复杂性；

## Fragment 与 Activity

- Activity 通过 ActivityManagerService、WindowManagerService 管理，跨进程，属于系统组件 
- Fragment 通过 FragmentManager 管理，FragmentManager 属于对应的 FragmentActivity，一个 FragmentActivity 可以添加任意个 Fragment，所以 Fragment 相当于 Activity 的组件；
- Fragment 可以返回一个对应的 View 用来显示，也可以不返回 View。返回 View 的时候 View 还是添加到了当前 Activity 的视图树里，也就是说用的是 Activity 的 Window，可以看作是 Activity 的一个视图组建；不返回 View 的时候相当于管理一段具有生命周期的业务逻辑。
- Fragment 之间可以互相嵌套，通过 getChildFragmentManager 实现


## Fragment 生命周期

Fragment 的生命周期总体与 Activity 相对应

1. Fragment 在 xml 中定义时

		A onCreate - F onAttach - F onCreate - F onCreateView - F onViewCreated 
		A onStart - F onActivityCreate - F onStart 
		A onResume - A onPostResume - A onResumeFragments- F onResume
		
		A 表示 Activity，F 表示 Fragment

2. Fragment 通过 FragmentTransaction 进行添加时，由于可以反复对 Fragment 进行操作，所以就涉及到 Fragment 直接的生命周期关系
	
	add 操作，不会影响已添加的 Fragment 生命周期
	
		add A -> add B
		
		A onAttach - A onCreate - A onCreateView - A onActivityCreated - A onStart - A onResume
		B onAttach - B onCreate - B onCreateView - B onActivityCreated - B onStart - B onResume
	
	replace 操作
	
		add A -> replace B
		
		A onAttach - A onCreate - A onCreateView - A onActivityCreated - A onStart - A onResume
		B onAttach - B onCreate
		A onPause - A onDestroyView - A onDestroy - A onDetach
		B onCreateView - B onActivityCreated - B onStart - B onResume
		
## FragmentTransaction 异步提交

	getSupportFragmentManager().beginTransaction()
		.replace(R.id.main_content, fragmentB).commit();
		
FragmentTransaction 的 commit 操作都是异步的，每次 beginTransaction 其实是构建了一个 BackStackRecord，所有操作都封装在这个 BackStackRecord里，之后通过 Handler 发送了一个 runnable 的消息到主线程去执行。

