### 1. onTrimMemory\(\)

[Activity的onTrimMemory\(\)方法](http://blog.csdn.net/wzhidev/article/details/51786372)

```java
@Override
public void onTrimMemory(int level) {
    super.onTrimMemory(level);
    switch (level) {
    case TRIM_MEMORY_UI_HIDDEN:
        // 进行资源释放操作
        break;
    }
}
```

 TRIM\_MEMORY\_UI\_HIDDEN：表示app所有相关界面已经不可见了。该回调只有当我们程序中的所有UI组件全部不可见的时候才会触发，这和onStop\(\)方法还是有很大区别的，因为onStop\(\)方法只是当一个Activity完全不可见的时候就会调用，比如说用户打开了我们程序中的另一个Activity。因此，我们可以在onStop\(\)方法中去释放一些Activity相关的资源，比如说取消网络连接或者注销广播接收器等，但是像UI相关的资源应该一直要等到onTrimMemory\(TRIM\_MEMORY\_UI\_HIDDEN\)这个回调之后才去释放，这样可以保证如果用户只是从我们程序的一个Activity回到了另外一个Activity，界面相关的资源都不需要重新加载，从而提升响应速度。

以我们的app为例，里面有这么一段使用

```java
    @Override
    public void onTrimMemory(int level) {
        if (level == TRIM_MEMORY_UI_HIDDEN && !mIsInBackground) {
            Log.e("onTrimMemory", "JIUYAN_app_hide");
            mIsInBackground = true;
            sendMessage(true);
            mTimestamp = System.nanoTime();
//            LogRecorder.instance().recordWidthTime("go to background");
        }
    }
```

 意思是在app界面完全不可见时，将一个全局静态变量mIsInBackground设置为true。（该变量在BaseActivity的onResume中会判断并设置为false）。

```java
 if (mIsInBackground || BaseApplication.getInstance().isScreenLocked()) {
            mIsInBackground = false;
            ……
 }
```

其中锁屏的判断可以在自己app的自定义Application中注册广播接收器来监测。

```java
 protected final BroadcastReceiver mScreenReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            if (intent != null) {
                String action = intent.getAction();
                if (Intent.ACTION_SCREEN_ON.equals(action)) {
                    // 开屏
                    isScreenLocked = false;
                } else if (Intent.ACTION_SCREEN_OFF.equals(action)) {
                    // 锁屏
                    isScreenLocked = true;
                    mTimestamp = System.nanoTime();
                } else if (Intent.ACTION_USER_PRESENT.equals(action)) {
                    // 解锁
                    isScreenLocked = false;
                }
            }
        }
    };
```

回到那个level的判定，这里的逻辑是发送了一个message。其实是具体的业务逻辑，当app界面完全不可见时，通过EventBus发送了一个sendMsgEvent，携带状态参数和要接受的目标，在需要处理的Activity中接收到该事件后，启动相应的处理，本处是向长连接的MsgService服务发送了一个状态改变的消息。通过Messenger实现和依附于app的独立进程的remote service之间的通信，（IPC的另一种形式），实现msg传递以及接收后逻辑的处理。

几个相关知识点：

1. IPC的有几种方式，Messenger的使用
2. Service的保活。

在MsgService中注册了一个整分广播，这是系统每隔一分钟定时发的广播，然后在广播接收中哦按段service是否还活着，否则就重启。应该属于进程拉活吧？相关代码：

定义系统广播接收器

```java
public class TimeChangeReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        ServiceHelper.startIfNotAlive(context);
    }
}
```

在MsgService的onCreate\(\)中注册该广播：

```java
    @Override
    public void onCreate() {
        super.onCreate();
        isImRunning = false;
        forceForeground();
        registerAllReceiver();
    }
```

```java
    private void registerAllReceiver() {
        //整分广播
        timeChangeReceiver = new TimeChangeReceiver();
        registerReceiver(timeChangeReceiver, new IntentFilter(Intent.ACTION_TIME_TICK));
    }
```

其中，Intent.ACTION\_TIME\_TICK表示系统没间隔一分钟发送该广播。这里注意在onDestroy\(\)中不要忘了注销它。

1. 使用了一种手段使Service保持前台状态，这样不容易被杀死。

```java
  private void forceForeground() {
        if (Build.VERSION.SDK_INT < 18) {
            startForeground(GRAY_SERVICE_ID, new Notification());//API < 18 ，此方法能有效隐藏Notification上的图标
        } else {
            Intent innerIntent = new Intent(this, GrayInnerService.class);
            startService(innerIntent);
            startForeground(GRAY_SERVICE_ID, new Notification());
        }
    }
```

```java
/**
     * 给 API >= 18 的平台上用的灰色保活手段
     */
    public static class GrayInnerService extends Service {

        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            startForeground(GRAY_SERVICE_ID, new Notification());
            stopForeground(true);
            stopSelf();
            return super.onStartCommand(intent, flags, startId);
        }

        @Nullable
        @Override
        public IBinder onBind(Intent intent) {
            return null;
        }

    }
```

这是MsgService的一个内部Service类，它们的声明在：

```java
        <service
            android:name=".service.persistentconnection.MsgService"
            android:enabled="true"
            android:exported="true"
            android:process=":msg" >
            <intent-filter >
                <action android:name="com.jiuyan.infashion.msgservice" />
            </intent-filter>
        </service>
        <service
            android:name=".service.persistentconnection.MsgService$GrayInnerService" />
```

所以想到了被问过的一个问题，见过带$的函数吗？这就是一种情况。

这其中的处理方式，进程保活 参照：[Android保证service不被杀掉-增强版: 进程保活\(根据用户需求慎用\)](http://blog.csdn.net/u011622280/article/details/52311344)

