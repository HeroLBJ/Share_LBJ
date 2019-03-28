## 开始实现
### 1、实现Thread.UncaughtExceptionHandler接口
```
public class ExceptionHelper implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        
    }
}
```
### 2、单例模式
```
private static volatile ExceptionHelper INSTANCE;

private ExceptionHelper() {}

public static ExceptionHelper getInstance() {
    if (INSTANCE == null) {
        synchronized (ExceptionHelper.class) {
            if (INSTANCE == null) {
                synchronized (ExceptionHelper.class) {
                    INSTANCE = new ExceptionHelper();
                }
            }
        }
    }
    return INSTANCE;
}
```
### 3、初始化异常信息
```
private Thread.UncaughtExceptionHandler mDefaultHandler;

public void init() {
    // 获取默认异常处理器
    mDefaultHandler = Thread.getDefaultUncaughtExceptionHandler();
    // 将当前类设为默认异常处理器
    Thread.setDefaultUncaughtExceptionHandler(this);
}
```
### 4、捕获异常
```
@Override
public void uncaughtException(Thread t, Throwable e) {
    if (handlerException(e)) {
        // 未处理异常，调用系统方法来进行处理，弹出错误弹框或闪退
        if (mDefaultHandler != null) {
            mDefaultHandler.uncaughtException(t, e);
        }
    } else {
        // 拦截错误，可进行重启APP等后续操作
    }
}

private boolean handleException(Throwable e) {
    if (e == null) {
        return false;
    }
    Writer writer = new StringWriter();
    PrintWriter pw = new PrintWriter(writer);
    e.printStackTrace(pw);
    pw.close();
    String result = writer.toString();
    Log.e("TAG", result);
    return true;
}
```
### 5、在App中初始化
```
public class App extends Application {

    private static Context mContext;

    @Override
    public void onCreate() {
        super.onCreate();
        mContext = getApplicationContext();
        // 初始化ExceptionHelper
        ExceptionHelper.getInstance().init();
    }

    public static Context getContext() {
        return mContext;
    }
}
```

<br>

## 测试
> 在启动Activity中调用 ```int a = 1 / 0```。APP没有闪退，也没有弹出错误提示。收集到的错误日志如下：
```
E/TAG:java.lang.ArithmeticException: divide by zero
    at com.lbj.mvpflower.mvp.ui.activity.UserActivity.onUser(UserActivity.java:36)
    at java.lang.reflect.Method.invoke(Native Method) 
    at android.view.View$DeclaredOnClickListener.onClick(View.java:4702) 
    at android.view.View.performClick(View.java:5619) 
    at android.view.View$PerformClick.run(View.java:22298) 
    at android.os.Handler.handleCallback(Handler.java:754) 
    at android.os.Handler.dispatchMessage(Handler.java:95) 
    at android.os.Looper.loop(Looper.java:165) 
    at android.app.ActivityThread.main(ActivityThread.java:6365) 
```
<br>

## 完整代码
```
public class ExceptionHelper implements Thread.UncaughtExceptionHandler {

    private static volatile ExceptionHelper INSTANCE;

    private ExceptionHelper() {
    }

    public static ExceptionHelper getInstance() {
        if (INSTANCE == null) {
            synchronized (ExceptionHelper.class) {
                if (INSTANCE == null) {
                    synchronized (ExceptionHelper.class) {
                        INSTANCE = new ExceptionHelper();
                    }
                }
            }
        }
        return INSTANCE;
    }

    private Thread.UncaughtExceptionHandler mDefaultHandler;

    /**
     * 初始化默认异常捕获
     */
    public void init() {
        // 获取默认异常处理器
        mDefaultHandler = Thread.getDefaultUncaughtExceptionHandler();
        // 将当前类设为默认异常处理器
        Thread.setDefaultUncaughtExceptionHandler(this);
    }

    @Override
    public void uncaughtException(Thread t, Throwable e) {
        if (handleException(e)) {
            // 已经处理,APP重启
            restartApp();
        } else {
            // 如果不处理,则调用系统默认处理异常,弹出系统强制关闭的对话框
            if (mDefaultHandler != null) {
                mDefaultHandler.uncaughtException(t, e);
            }
        }
    }

    private boolean handleException(Throwable e) {
        if (e == null) {
            return false;
        }

        Writer writer = new StringWriter();
        PrintWriter pw = new PrintWriter(writer);
        e.printStackTrace(pw);
        pw.close();
        String result = writer.toString();
        // 打印出错误日志
        Log.e("TAG", result);
        return true;
    }

    /**
     * 1s后让APP重启
     */
    private void restartApp() {
        Intent intent = App.getContext().getPackageManager()
                .getLaunchIntentForPackage(App.getContext().getPackageName());
        PendingIntent restartIntent = PendingIntent.getActivity(App.getContext(), 0, intent, 0);
        AlarmManager mgr = (AlarmManager) App.getContext().getSystemService(Context.ALARM_SERVICE);
        // 1秒钟后重启应用
        mgr.set(AlarmManager.RTC, System.currentTimeMillis() + 1000, restartIntent);
        System.exit(0);
    }
}
```