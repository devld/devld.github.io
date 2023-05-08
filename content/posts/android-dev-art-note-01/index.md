---
title: "《Android 开发艺术探索》笔记 - Activity - 01"
date: "2017-10-24T17:56:00+08:00"
categories:
- Android
tags:
- android
- notes
---
### Activity 生命周期


#### 正常情况下 Activity 的生命周期

1. **onCreate**: 表示 Activity 正在被创建, 主要在这个方法中做一些初始化.

2. **onRestart**: 表示 Activity 正在重新启动, 一般情况下, 当 Activity 从不可见变为可见时会被调用.

3. **onStart**: 表示 Activity 正在启动, 但还无法与用户交互, 此时 Activity 还在后台.

4. **onResume**: 表示 Activity 已经可见了, 和 onStart 的区别是现在 Activity 处于前台.

5. **onPause**: 当其他 Activity 进入前台时会被调用, 这个方法里不能执行耗时的操作, 否则会影响新 Activity 的启动, 因为新 Activity 的 onResume 会在这个 Activity 的 onPause 执行完成后才会被调用.

6. **onStop**: 表示 Activity 即将停止, 可以做一些回收工作, 但也不能太耗时.

7. **onDestroy**: 表示 Activity 即将被销毁, 这是最后一个回调.

<!-- more -->

![Activity Life Cycle][1]

上图来源于: https://developer.android.google.cn/reference/android/app/Activity.html

Activity 的生命周期回调方法分为一下几种情况:

1. 首次启动, `onCreate -> onStart -> onResume`.

2. 当用户切换回桌面或其他 Activity 时, `onPause -> onStop`. 特殊情况: 如果新的 Activity 使用了透明主题, 那么当前 Activity 的 `onStop` 不会被调用.

3. 用户再次回到这个 Activity 时, `onRestart -> onStart -> onResume`.

4. 当用户按下 `返回键` 时, 默认情况下 Activity 会被销毁, `onPause -> onStop -> onDestroy`.

5. `onCreate` 和 `onStop` 对应着 Activity 的 `创建` 与 `销毁`；`onStart` 和 `onStop` 对应着`可见`与`不可见`；`onResume` 和 `onPause` 对应着 `前台` 和 `后台`.


#### 异常情况下 Activity 的生命周期

1. 系统资源发生改变

  比如屏幕方向发生变化时. 在默认情况下, Activity 会被销毁, 正常的生命周期方法会被调用, 并且 `onSaveInstanceState` 和 `onRestoreInstanceState` 会被额外调用, 用于保存和恢复当前状态.
  `onSaveInstanceState`会在`onStop`之前被调用,  `onRestoreInstanceState` 会在 `onStart` 之后被调用.
  并且, Activity 会自动帮我们保存一些数据, 比如用户在文本框中输入的文本等等.
  View 也有 `onSaveInstanceState` 和 `onRestoreInstanceState` 两个方法, 当 Activity 意外停止时, 他们会被调用.

2. 系统资源不足导致 Activity 被销毁

  Activity 的优先级如下:
  1. 前台 Activity, 即正在和用户交互的 Activity.
  2. 可见, 但是在后台的 Activity.
  3. 在后台的 Activity, 并且不可见.

  前台 Activity 最不容易被杀死.

  在这种情况下, `onSaveInstanceState` 和 `onRestoreInstanceState` 也会被调用.

### Activity 的启动模式 (Launch Mode)

1. **standard**: 标准模式, 也是默认的模式. 可以有多个实例, 在哪个 Activity 栈中启动, 就会被至于那个栈中.

2. **singleTop**: 这种模式下, 如果这个 Activtiy 处于栈顶的话, 再次启动这个 Activity 并不会启动一个新的实例, 而这个 Activity 的 `onNewIntent` 会被调用, 可以在这个方法里处理新的请求.

    > “standard”和“singleTop”Activity 为一类, 使用“standard”或“singleTop”启动模式的 Activity 可多次实例化.  实例可归属任何任务, 并且可以位于 Activity 堆栈中的任何位置.

3. **singleInstance**: 该启动模式的 Activity 启动后会创建一个新的任务栈, 并且这个任务栈中始终只会有这一个实例, 如果再次启动, 不会创建新实例, 而 `onNewIntent` 会被调用.

4. **singleTask**: 该模式也是一种单实例模式, 如果当前任务栈中不存在这个 Activity , 则会创建这个 Activity, 否则会将已存在的实例调到栈顶, 并且会清除掉原来所有的在该 Activity 之上的 Activity, 如下图:

  ![SingleTask Mode][2]

  > “singleTask”和“singleInstance”模式同样只在一个方面有差异: “singleTask”Activity 允许其他 Activity 成为其任务的组成部分.  它始终位于其任务的根位置, 但其他 Activity（必然是“standard”和“singleTop”Activity）可以启动到该任务中.  相反, “singleInstance”Activity 则不允许其他 Activity 成为其任务的组成部分. 


  另外, 在 `singleTop`, `singleTask`, `singleInstance` 在栈顶时, 再次启动该 Activity 时, 生命周期调用顺序为: `onPause -> onNewIntent -> onResume`.


#### Activity Flags

1. FLAG_ACTIVITY_NEW_TASK

  这个 flag 效果与在 xml 中指定 `singleTask` 效果相同.

2. FLAG_ACTIVITY_SINGLE_TOP

  这个 flag 效果与在 xml 中指定 `singleTop` 效果相同.

3. FLAG_ACTIVITY_CLEAR_TOP

  具有此标记为的 Activity 在启动时, 在同一个任务栈中的所有位于它上面的 Activity 都要出栈.
  一般这个 flag 与 FLAG_ACTIVITY_NEW_TASK 配合使用.
  如果被启动的 Activity 是 `standard` 启动模式, 那么连同它自己以及它之上的 Activity 都要出栈.
  系统会重新创建该 Activity 的实例, 并放入栈顶.

4. FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

  这个 flag 可以使启动的 Activity 不在最近任务列表中显示.


### IntentFilter 匹配规则

Activity 的启动分为 显式启动 和 隐式启动.

显式启动通过直接指定包名组件名直接启动对应的组件.

隐式启动不需要指定特定的包名或组件名, 可以通过某些信息匹配某些 Activity.

intent-filter 中有三种过滤信息, action, category, data, 而在 startActivity 时,
必须匹配这三个信息, 否则会匹配失败.
一个组件中可以有多个 intent-filter, 只要匹配到其中任意一个即可.

- action

  action 的匹配要求 Intent 中的 action 字符串与过滤器中声明的相同, 并且区分大小写.
  在单个 intent-filter 中可以定义多个 action, 而 Intent 中仅需要匹配到任意一个即可.

- category

  Intent 可以添加多个 category, Intent 中如果含有 category,
  那么 Intent 中添加的 category 必须都在 intent-filter 中声明, 否则会匹配失败.

  另外, 在 intent-filter 中必须添加一个 android.intent.category.DEFAULT, 否则它不会被任何 Intent 匹配到,
  因为
  > Android 会自动将 CATEGORY_DEFAULT 类别应用于传递给 startActivity() 和 startActivityForResult() 的所有隐式 Intent. 因此, 如需 Activity 接收隐式 Intent, 则必须将 "android.intent.category.DEFAULT" 的类别包括在其 Intent 过滤器中（如上文的 <intent-filter> 示例所示）.

- data

  > 使用一个或多个指定数据 URI 各个方面（scheme、host、port、path 等）和 MIME 类型的属性, 声明接受的数据类型.

  语法如下:

  ```xml
  <data android:scheme="string"
        android:host="string"
        android:port="string"
        android:path="string"
        android:pathPattern="string"
        android:pathPrefix="string"
        android:mimeType="string" />
  ```

  + scheme

    URI 的模式, 比如 http, ftp, file, content 等, 这个是必选项, 如果不指定这个参数, 那么这个 URI 就是无效的.

  + host

    URI 的主机名, 比如 www.google.com, host 也是必须的.

  + port

    主机的端口号, 比如 80.

  + path | pathPattern | pathPrefix

    表示 URI 的路径, path 表示完整的路径, pathPattern 中可以包含通配符, pathPrefix 表示路径的前缀

  + mimeType

    表示文件的类型, 比如 `image/png`, `text/html` 等等.


  在被启动的 Activity 中可以通过 `Intent#getData` 获取到匹配到的 URI.

  在 Intent 中可以通过 `Intent#setData`, `Intent#setType` 设置 data 或 type.
  如果需要同时设置 data 和 type 则需要调用 `Intent#setDataAndType`, 因为上面两个方法都会清空另一个的值.

  如果 `intents-filters` 仅设置了 `mimeType`, 而没有设置 `scheme`, `scheme` 默认会是 `file` 或 `content`.


在隐式启动过程中, 如果没有匹配到任何组件, 那么程序会报错停止运行.
通常需要在隐式启动时, 判断是否有 Activity 可以匹配到.

有两种方法:

1. 通过 `PackageManager` 的一系列 `resolve...()`  来查询, 如果可以匹配, 则会返回响应 Intent 的最佳组件, 否则就会返回 null.
2. `PackageManager` 该提供了一系列的 `query...()` 的方法, 它会返回所有可以匹配的 Activity.

> 两种方法都不会直接激活组件, 而只是列出可以响应的组件.

```java
public abstract List<ResolveInfo> queryIntentActivities(Intent intent, int flags);
public abstract ResolveInfo resolveActivity(Intent intent, int flags);
```

注意第二个参数, 最好使用 MATCH_DEFAULT_ONLY
```java
/**
 * Resolution and querying flag: if set, only filters that support the
 * {@link android.content.Intent#CATEGORY_DEFAULT} will be considered for
 * matching.  This is a synonym for including the CATEGORY_DEFAULT in your
 * supplied Intent.
 */
public static final int MATCH_DEFAULT_ONLY  = 0x00010000;
```

这个标记的含义是仅仅匹配那些在 intents-filters 中声明了 android.intent.category.DEFAULT 的 Activity,
如果 Activity 没有声明这个 category, 使用 startActivity 将会报错.

参考资料:

https://developer.android.google.cn/guide/topics/manifest/activity-element.html

https://inthecheesefactory.com/blog/understand-android-activity-launchmode/en

http://blog.csdn.net/javazejian/article/details/52072131

https://developer.android.com/guide/components/intents-filters.html

[1]: activity-lifecycle.png
[2]: mode-singleTask.gif
