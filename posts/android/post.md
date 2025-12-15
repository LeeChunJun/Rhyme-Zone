---
title: Android 技术栈博客文章
tags: [misc, android]
date: 2025-12-06
private: false
template: post
---

## Android 面试

✦ 基于你的Android开发经验，我来为你总结面试的核心知识点：

  Android基础知识

  1. 四大组件
   - Activity: 生命周期、启动模式、任务栈管理
   - Service: 前台服务、后台服务、绑定服务
   - BroadcastReceiver: 静态注册、动态注册、LocalBroadcastManager
   - ContentProvider: 数据共享、权限管理、ContentResolver

  2. Activity生命周期
   - onCreate() → onStart() → onResume() → onPause() → onStop() → onDestroy()
   - 异常情况下的生命周期处理（配置变更、内存不足）
   - onSaveInstanceState() 和 onRestoreInstanceState()

  3. Intent和IntentFilter
   - 显式Intent和隐式Intent
   - Intent的七大属性（Component、Action、Data、Type、Category、Extra、Flag）
   - Intent Filter匹配规则

  UI开发

  1. 布局系统
   - ConstraintLayout使用技巧
   - RecyclerView优化（ViewHolder复用、DiffUtil）
   - 自定义View和ViewGroup

  2. Material Design
   - Material Components库使用
   - 主题和样式系统
   - 深色模式适配

  3. Jetpack Compose（现代UI）
   - 声明式UI编程思想
   - State管理（remember、mutableStateOf）
   - Compose布局原则

  架构设计

  1. MVVM架构
   - ViewModel的作用和生命周期
   - LiveData和StateFlow/SharedFlow
   - DataBinding和ViewBinding

  2. 依赖注入
   - Hilt的基本使用
   - 模块化架构设计

  3. Clean Architecture
   - 分层架构原则（Presentation、Domain、Data）
   - Repository模式实现

  数据存储

  1. 本地存储
   - SharedPreferences和DataStore
   - SQLite数据库和Room
   - 文件存储

  2. 数据库优化
   - 索引优化
   - 事务处理
   - 数据库升级策略

  网络编程

  1. 网络库
   - Retrofit使用和配置
   - OkHttp拦截器
   - 网络请求优化

  2. 数据解析
   - JSON解析（Gson、Moshi）
   - XML解析
   - Protocol Buffers

  并发编程

  1. 线程管理
   - Handler、Looper、MessageQueue
   - AsyncTask的替代方案
   - 线程池和Executor

  2. Kotlin协程
   - CoroutineScope和CoroutineContext
   - 协程的异常处理
   - Flow的使用场景

  性能优化

  1. 内存优化
   - 内存泄漏检测和预防
   - OOM问题解决
   - 内存分析工具使用

  2. 启动优化
   - 冷启动和热启动优化
   - Application初始化优化
   - 启动时间监控

  3. 渲染优化
   - 布局优化（减少层级、避免过度绘制）
   - GPU渲染分析
   - 卡顿问题排查

  安全性

  1. 数据安全
   - 加密算法使用
   - KeyStore系统
   - 网络传输安全（HTTPS、证书校验）

  2. 权限管理
   - 运行时权限申请
   - 权限最佳实践
   - 隐私保护

  测试

  1. 单元测试
   - JUnit使用
   - Mockito框架
   - 测试覆盖率

  2. UI测试
   - Espresso框架
   - UI Automator
   - 测试策略制定

  开发工具

  1. Android Studio
   - 调试技巧
   - 性能分析工具
   - 代码生成和重构

  2. 构建系统
   - Gradle配置
   - 多渠道打包
   - ProGuard/R8混淆

  新技术趋势

  1. 跨平台开发
   - Flutter vs React Native对比
   - Kotlin Multiplatform Mobile
   - 跨平台技术选型

  2. 现代Android开发
   - Kotlin优先（不再推荐Java）
   - Jetpack全家桶
   - 声明式UI（Compose）

  面试准备建议

   1. 准备项目亮点: 选择2-3个最有代表性的项目，准备技术难点和解决方案
   2. 算法和数据结构: 刷一些常见的面试题（数组、链表、树、排序等）
   3. 系统设计: 了解高并发、缓存、消息队列等概念
   4. 新技术学习: 关注Android最新动态，了解Jetpack Compose等新技术

## Android 触摸事件传递机制

✦ Android触摸事件传递机制

  1. 事件序列
  触摸事件以MotionEvent对象形式传递，包含三个主要阶段：
   - ACTION_DOWN: 手指首次接触屏幕，事件序列的开始
   - ACTION_MOVE: 手指在屏幕上移动，可能包含多个MOVE事件
   - ACTION_UP: 手指离开屏幕，事件序列的结束

  2. 传递顺序
  事件传递遵循严格的层级顺序：
   1 Activity → PhoneWindow → DecorView → ViewGroup → View

  3. 三个核心方法

  dispatchTouchEvent(MotionEvent event)
   - 作用: 事件的分发入口，所有事件都经过此方法
   - 返回值:
     - true: 事件被消费，不再继续传递
     - false: 事件未消费，向上传递给父容器
     - super.dispatchTouchEvent(): 调用默认的分发逻辑

  onInterceptTouchEvent(MotionEvent event)
   - 作用: ViewGroup特有，决定是否拦截事件
   - 返回值:
     - true: 拦截事件，事件不再传递给子View
     - false: 不拦截，继续传递给子View
     - 注意: 对ACTION_DOWN事件返回false是常规做法

  onTouchEvent(MotionEvent event)
   - 作用: 事件的处理方法，决定是否消费事件
   - 返回值:
     - true: 消费事件，后续事件仍会传递到此View
     - false: 不消费，事件回传给父View

  4. 传递规则详解

  事件分发流程
  ```
   1. Activity.dispatchTouchEvent() →
   2. Window.dispatchTouchEvent() →
   3. DecorView.dispatchTouchEvent() →
   4. ViewGroup.dispatchTouchEvent() →
   5. ViewGroup.onInterceptTouchEvent() →
   6. 子View.dispatchTouchEvent() →
   7. 子View.onTouchEvent()
  ```

  拦截机制
   - ViewGroup默认不拦截ACTION_DOWN事件
   - 一旦拦截了某个事件序列中的后续事件，同一序列的其他事件也会被拦截
   - onInterceptTouchEvent()只对ACTION_DOWN返回false，后续事件才有可能被拦截

  事件回传
  当子View不消费事件时，事件会向上回传：
   1 子View.onTouchEvent() → 父ViewGroup.onTouchEvent() → 父ViewGroup的父容器.onTouchEvent()

  5. 关键源码分析

  ViewGroup.dispatchTouchEvent()核心逻辑
  ```java
    1 // 检查是否需要拦截
    2 if (actionMasked == MotionEvent.ACTION_DOWN) {
    3     // 重置状态
    4     cancelAndClearTouchTargets(ev);
    5     resetTouchState();
    6 }
    7
    8 // 检查拦截
    9 final boolean intercepted;
   10 if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
   11     final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
   12     if (!disallowIntercept) {
   13         intercepted = onInterceptTouchEvent(ev);
   14     } else {
   15         intercepted = false;
   16     }
   17 }
  ```

  6. 常见场景处理

  滑动冲突解决
   - 外部拦截法: 在父ViewGroup的onInterceptTouchEvent()中处理
   - 内部拦截法: 在子View中通过requestDisallowInterceptTouchEvent()控制

  点击事件与滑动事件区分
  ```java
    1 @Override
    2 public boolean onTouchEvent(MotionEvent event) {
    3     switch (event.getAction()) {
    4         case MotionEvent.ACTION_DOWN:
    5             // 记录按下位置和时间
    6             mLastX = event.getX();
    7             mLastY = event.getY();
    8             mDownTime = System.currentTimeMillis();
    9             break;
   10
   11         case MotionEvent.ACTION_MOVE:
   12             // 计算滑动距离
   13             float dx = Math.abs(event.getX() - mLastX);
   14             float dy = Math.abs(event.getY() - mLastY);
   15
   16             // 判断是否为滑动
   17             if (dx > mTouchSlop || dy > mTouchSlop) {
   18                 // 处理滑动逻辑
   19                 return true;
   20             }
   21             break;
   22     }
   23     return super.onTouchEvent(event);
   24 }
  ```

  7. 重要注意事项

   1. 事件序列一致性: 一旦View决定处理某个事件序列，后续事件都会传递给它
   2. ACTION_DOWN的特殊性: 每个事件序列都以ACTION_DOWN开始
   3. requestDisallowInterceptTouchEvent(): 子View可以阻止父View拦截事件
   4. 消费后的传递: 事件被消费后，同一序列的后续事件仍会传递给该View

  8. 实际应用案例

  自定义滑动View
  ```java
    1 public class CustomScrollView extends ViewGroup {
    2     @Override
    3     public boolean onInterceptTouchEvent(MotionEvent ev) {
    4         switch (ev.getAction()) {
    5             case MotionEvent.ACTION_DOWN:
    6                 // 不拦截DOWN事件，让子View有机会处理
    7                 return false;
    8             case MotionEvent.ACTION_MOVE:
    9                 // 根据滑动距离决定是否拦截
   10                 return shouldIntercept(ev);
   11             default:
   12                 return super.onInterceptTouchEvent(ev);
   13         }
   14     }
   15 }
  ```

✦ 这个机制是Android开发的核心知识点，理解它对于解决复杂的触摸交互问题至关重要。