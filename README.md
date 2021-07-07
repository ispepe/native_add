# native_add

原生调用示例项目

## 搭建开发环境

本示例通过Flutter开发框架一次性获取所需lib，[安装和环境配置](https://flutter.cn/docs/get-started/install)

## 构建lib步骤

### 添加 C/C++ 源码

- 注意：将源代码添加到 ios 文件夹，因为 CocoaPods 不允许源码处于比 podspec 文件更高的目录层级，但是 Gradle 允许您指向 ios 文件夹。
- 创建一个 C++ 文件，路径为：ios/Classes/native_add.cpp（已创建）

### Android 添加构建动态库配置

- 在 Android 中，您需要创建一个 android/CMakeLists.txt 文件用来定义如何编译源文件，同时告诉 Gradle 如何去定位它们
```
cmake_minimum_required(VERSION 3.4.1)  # for example

add_library( native_add
     # Sets the library as a shared library.
     SHARED

     # 指定待编译的源文件
     ../ios/Classes/native_add.cpp )
```
- 添加一个 externalNativeBuild 到您的 android/build.gradle 文件中
```
android {
//////////
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
/////////
}
```

### IOS 添加构建动态库配置 

- ios/native_add.podspec 中已配置`s.source_files = 'Classes/**/*'`表示从ios/Classes目录自动获取源文件

### 添加统一调用接口

- 在lib/native_add.dart 添加调用接口
```dart
import 'dart:ffi'; // For FFI
import 'dart:io'; // For Platform.isX

/// 链接动态库
/// ios自动生成[native_add.framework]
/// android按照CMakeLists.txt文件的`add_library( native_add)`生成[libnative_add.so]，lib前缀会自动添加
final DynamicLibrary nativeAddLib = Platform.isAndroid
    ? DynamicLibrary.open("libnative_add.so")
    : DynamicLibrary.process();

/// 导出一个nativeAdd接口
final int Function(int x, int y) nativeAdd = nativeAddLib
    .lookup<NativeFunction<Int32 Function(Int32, Int32)>>("native_add")
    .asFunction();
```

- 在`example/lib/main.dart`调用接口
```dart
import 'package:native_add/native_add.dart';

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Plugin example app'),
        ),
        /// 调用nativeAdd并显示
        body: Center(
          child: Text('1 + 2 = ${nativeAdd(1, 2)}'),
        ),
      ),
    );
  }
```

### 获取ios/android动态库

- 在Android Studio中点击运行按钮编译和运行
- 在`example/build/native_add/intermediates/cmake/debug/obj/`获取对应安卓平台的动态库
- 在`example/build/ios/iphonesimulator/Runner.app/Frameworks/`获取ios的库文件`native_add.framework`

### 注意事项⚠️⚠️

- ios的库实际是framework文件，它实际是一个目录文件，上架App Store时交由Apple生成不同版本的IOS库文件
- example是APP项目，不用修改
- 本项目`native_add`是package项目，`lib/native_add.dart`文件用于包装接口，供APP项目使用，由我来完善测试用
- 主要编写`ios/Classes/native_add.cpp`，并编译获取lib文件