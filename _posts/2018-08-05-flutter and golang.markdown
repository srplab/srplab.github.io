---
layout: post
title:  "Flutter & golang"
date:   2018-08-05 20:10:07 +0800
categories: flutter
tags: go flutter android ios
---

### This example shows how to call golang modules from flutter.

This example is based on [starcore_for_flutter](https://github.com/srplab/starcore_for_flutter), which enables flutter calls other programming languages.
The golang module to be call is taken from the post [Writting Go Lang Multi-Threading Extension for Other Scripts](http://www.srplab.info/go/2018/06/19/writting_golang_multithreading_extension_for_other_scripts.html)
The code also packed in the example of this post.

<h1 align = "left"><font color="#FF9900">Step by Step</font></h1>

### 1. Create flutter project

```sh
$ flutter create flutter_golang
```

### 2. Add dependency package "starflut"

Edit "pubspec.yaml"

```
dev_dependencies:
  flutter_test:
    sdk: flutter

  starflut: ^0.5.2
```

### 3. Create "jniLibs" for golang share libraries of android

![](/images/flutter_and_golang_android_sharelibrary.png){:width="320px"}

The "libstar_go.so" is compiled from golang module, as step 4.

### 4. Compiling golang module for android and ios

* Android

```sh
# compiled with android-ndk-r12b

GO="/usr/local/go/bin/go"

arch=("x86" "x86_64" "arm64" "arm");

for s in ${arch[@]}; do
  echo "build for $s"

  cd writting_golang_multithreading_extension_for_other_scripts

  export GOOS=android
  export CGO_ENABLED=1

  if [ "$s" == "arm" ]; then
    export NDK_TOOLCHAIN=/Users/srplab/Desktop/ndk_toolchain/android-toolchain.arm
    export CC=$NDK_TOOLCHAIN/bin/arm-linux-androideabi-gcc
    export GOARCH=arm
    export GOARM=7
  elif [ "$s" == "x86" ]; then
    export NDK_TOOLCHAIN=/Users/srplab/Desktop/ndk_toolchain/android-toolchain.x86
    export CC=$NDK_TOOLCHAIN/bin/i686-linux-android-gcc
    export GOARCH=386
  elif [ "$s" == "x86_64" ]; then
    export NDK_TOOLCHAIN=/Users/srplab/Desktop/ndk_toolchain/android-toolchain.x86_64
    export CC=$NDK_TOOLCHAIN/bin/x86_64-linux-android-gcc
    export GOARCH=amd64
  elif [ "$s" == "arm64" ]; then
    export NDK_TOOLCHAIN=/Users/srplab/Desktop/ndk_toolchain/android-toolchain.arm64
    export CC=$NDK_TOOLCHAIN/bin/aarch64-linux-android-gcc
    export GOARCH=arm64
  fi

  $GO build -buildmode=c-shared -o libstar_go.so
  
  cd ..
  if [ "$s" == "arm" ]; then
    cp writting_golang_multithreading_extension_for_other_scripts/libstar_go.so flutter_golang/android/app/src/main/jniLibs/armeabi-v7a
  elif [ "$s" == "x86" ]; then
    cp writting_golang_multithreading_extension_for_other_scripts/libstar_go.so flutter_golang/android/app/src/main/jniLibs/x86
  elif [ "$s" == "x86_64" ]; then
    cp writting_golang_multithreading_extension_for_other_scripts/libstar_go.so flutter_golang/android/app/src/main/jniLibs/x86_64
  elif [ "$s" == "arm64" ]; then
    cp writting_golang_multithreading_extension_for_other_scripts/libstar_go.so flutter_golang/android/app/src/main/jniLibs/arm64-v8a
  fi

  rm -rf writting_golang_multithreading_extension_for_other_scripts/libstar_go.so
 
done
```

* ios

```sh
rm -rf libstar_go.a

export PATH="/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/:/usr/local/bin:/usr/bin:/bin:$PATH"  
export SRCPATH="writting_golang_multithreading_extension_for_other_scripts"

arch=("i386" "x86_64" "arm64" "armv7");

for s in ${arch[@]}; do
  echo "build for $s"

  cd $SRCPATH

  export GOOS=darwin
  export CGO_ENABLED=1
  if [ "$s" == "armv7" ]; then
    export CFLAGS="-arch $s -miphoneos-version-min=6.0 -isysroot "$(xcrun -sdk iphoneos --show-sdk-path)
    export CC=/usr/local/go/misc/ios/clangwrap.sh
    export GOARCH=arm
    export GOARM=7
  elif [ "$s" == "i386" ]; then
    export CFLAGS="-arch $s -miphoneos-version-min=6.0 -isysroot "$(xcrun -sdk iphonesimulator --show-sdk-path)
    export CC="clang $CFLAGS"
    export GOARCH=386
  elif [ "$s" == "x86_64" ]; then
    export CFLAGS="-arch $s -miphoneos-version-min=6.0 -isysroot "$(xcrun -sdk iphonesimulator --show-sdk-path)
    export CC="clang $CFLAGS"
    export GOARCH=amd64
  elif [ "$s" == "arm64" ]; then
    export CFLAGS="-arch $s -miphoneos-version-min=6.0 -isysroot "$(xcrun -sdk iphoneos --show-sdk-path)
    export CC="clang $CFLAGS"
    export GOARCH=arm64
  fi

  go build -tags='ios' -buildmode=c-archive -o libstar_go.a
  
  cd ..
  mkdir $s
  cp $SRCPATH/libstar_go.a $s
  rm -rf $SRCPATH/libstar_go.a
 
done

lipo -create ./i386/libstar_go.a ./x86_64/libstar_go.a ./armv7/libstar_go.a ./arm64/libstar_go.a -o libstar_go.a

rm -rf ./i386
rm -rf ./x86_64
rm -rf ./armv7
rm -rf ./arm64

lipo -info libstar_go.a
```

### 5. dart code

* Initialize starcore

```dart
StarCoreFactory starcore = await Starflut.getFactory();
StarServiceClass Service = await starcore.initSimple("test", "123", 0, 0, []);
await Service.checkPassword(false);
```

* Register callback function

```dart
    await starcore.regMsgCallBackP(
        (int serviceGroupID, int uMsg, Object wParam, Object lParam) async{
          if( uMsg == Starflut.MSG_DISPMSG || uMsg == Starflut.MSG_DISPLUAMSG ){
            ShowOutput(wParam);
          }
      print("$serviceGroupID  $uMsg   $wParam   $lParam");
      return null;
    });
```

In callback function, we capture the output from golang, and show it in text window.

```dart
  void ShowOutput(String Info) async{
    if( Info == null || Info.length == 0)
      return;
    _outputString = _outputString + "\n" + Info;
    setState((){

    });
  }
 ```
 
* Load and execute golang module

```dart
    bool isAndroid = await Starflut.isAndroid();
    if( isAndroid == true ){
      String path = await Starflut.getNativeLibraryDir();
      List result = await Service.doFile("",path + "/libstar_go.so","");
      print(result);
    }else{
      List result = await Service.doFile("","libstar_go.dylib","");
      print(result);
    }
```

* Get starcore object created by golang module

```dart    
    StarObjectClass goObject = await Service.getObject("testgo");
    print("$goObject");
```

* Create starcore object, which will be called from golang

```dart    
    StarObjectClass callBackObj = await Service.newObject(["LuaObj"]);
    await callBackObj.regScriptProcP("Print", (StarObjectClass cleObject, List paras ) async {
			    print("hello from lua");
			    return null;
		      });
```

* Call golang function

```dart
    dynamic ttResult = await goObject.call("tt",[222,333]);
    print("$ttResult");
```

### 6. compile for ios

```sh
$ flutter clean
$ export STARCORE_PATH='/Users/srplab/Desktop'
$ export STARCORE_GOLIBRARYPATH='/Users/srplab/Desktop/flutter.study.example/flutter.golang'
$ flutter build ios --no-codesign
```

### 7. screenshot

![](/images/flutter_and_golang_ios_screenshot.png){:width="320px"}

![](/images/flutter_and_golang_android_screenshot.jpg){:width="320px"}


<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[flutter_and_golang_sample.zip](/datas/flutter_and_golang_sample.zip  "flutter_and_golang_sample")


