### 1、下载插件
> 在```SDK Manager```中找到```Android SDK```选中```LLDB```、```CMake```、```NDK```后点击OK按钮。

![下载插件](https://user-gold-cdn.xitu.io/2019/3/21/169a05d8e5ba71d5?w=2090&h=1480&f=png&s=447945)

### 2、创建CMakeLists.txt文件
> 在项目中app目录下创建```CMakeLists.txt```文件，在文件中粘贴以下内容：
```
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html
 
# Sets the minimum version of CMake required to build the native library.
 
cmake_minimum_required(VERSION 3.4.1)
 
# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.
 
add_library( 
        jni_lib # 项目中自行定义的so库名字
        SHARED  # Sets the library as a shared library.
        src/main/jni/jni_lib.c)  # c文件的全路径
        
# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.
find_library( # Sets the name of the path variable.
              log-lib
              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )
# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.
target_link_libraries( # Specifies the target library.
                       jni_lib # 项目中自行定义的so库名字
                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

### 3、创建Java类,类名随意
```
public class JNIClass {

    {
        // 加载so包，加载内容就是CmakeLists.txt中定义so包的名字
        System.loadLibrary("jni_lib");
    }

    // 定义一个native方法
    public native int add(int x,int y);
}
```

### 4、创建c文件
> 在```CMakeLists.txt```中有个c文件路径```src/main/jni/jni_lib.c```,在此目录下创建```jni_lib.c```文件，类名随意，但是两者需要保持一致。


### 5、开始构建
> 5.1 在NDK项目目录上右击选中```Link C++ Project with Gradle```

![](https://user-gold-cdn.xitu.io/2019/3/21/169a071f8c696845?w=1208&h=506&f=png&s=131007)

> 5.2 在弹出的选择框中，选择```CMakeLists.txt```的存放目录

![](https://user-gold-cdn.xitu.io/2019/3/21/169a0731ce424399?w=986&h=386&f=png&s=72042)

> 5.3 点击OK之后Gradle开始构建，在项目的```build.gradle```中可以看到多了下面几行代码

```
android{
    ...
    externalNativeBuild {
        cmake {
            path file('CMakeLists.txt')
        }
    }
}
```

### 6、创建c方法
> 在方法上```alt+enter```后在c文件中自动创建方法

![](https://user-gold-cdn.xitu.io/2019/3/21/169a07c0e8252753?w=1266&h=564&f=png&s=105172)

> 创建方法如下，生成名字规则 Java+全类名+方法名
```
JNIEXPORT jint JNICALL
Java_com_lbj_mvpflower_ndk_JNIClass_add(JNIEnv *env, jobject instance, jint x, jint y) {

    // TODO

}
```

### 7、调用c方法
> 修改c方法内容，求两个数的和

```
JNIEXPORT jint JNICALL
Java_com_lbj_mvpflower_ndk_JNIClass_add(JNIEnv *env, jobject instance, jint x, jint y) {

   return x + y;

}
```

> 修改完毕后，```Make Project```项目，在```build```目录下可以看到so文件，so文件名为```lib+自己去的名字+.so```

![](https://user-gold-cdn.xitu.io/2019/3/21/169a081b29f2f282?w=652&h=1004&f=png&s=115100)

> 最后在java中调用c的方法，得出结果，大功告成

![](https://user-gold-cdn.xitu.io/2019/3/21/169a08b976acc303?w=1896&h=222&f=png&s=96902)

