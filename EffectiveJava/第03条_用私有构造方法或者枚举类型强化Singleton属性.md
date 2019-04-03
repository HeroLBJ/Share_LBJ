## 单例模式最佳写法1 - 双重校验锁

```
public class Singleton {

    private static volatile Singleton INSTANCE;

    private Singleton(){}

    public static Singleton getInstance(){
        if(INSTANCE == null){
            synchronized (Singleton.class){
                if(INSTANCE == null){
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

<br>

## 单例模式最佳写法2 - 静态内部类

```
public class Singleton {

    private Singleton() {
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```


