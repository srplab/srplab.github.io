---
layout: post
title:  "Flutter & Python"
date:   2018-07-29 20:10:07 +0800
categories: flutter
tags: python flutter android ios
---

### This example shows how to run python script from flutter.

This example is based on [starcore_for_flutter](https://github.com/srplab/starcore_for_flutter), which enables flutter calls other scripts.

<h1 align = "left"><font color="#FF9900">Step by Step</font></h1>

### 1. Create flutter project

```sh
$ flutter create flutter_python
```

### 2. Add dependency package "starflut" and assets folder

Edit "pubspec.yaml"

```
dependencies:
  flutter:
    sdk: flutter

  starflut: any
  
  ...
  
  # To add assets to your application, add an assets section, like this:
  assets:
    - starfiles/  
```

### 3. Create "starfiles" folder at project root, and add file "python3.6.zip"

### 4. Create "assets" and "jniLibs", and add python share libraries for android

![](/images/flutter_and_python_android_sharelibrary.png){:width="320px"}

### 5. dart code

* Initialize starcore

```dart
StarCoreFactory starcore = await Starflut.getFactory();
StarServiceClass Service = await starcore.initSimple("test", "123", 0, 0, []);
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

In callback function, we capture the output from python, and show it in text window.

```dart
  void ShowOutput(String Info) async{
    if( Info == null || Info.length == 0)
      return;
    _outputString = _outputString + "\n" + Info;
    setState((){

    });
  }
 ```
 
* Initialize python

for android, before run python, share libraries must be copied to app's local folder.

```dart
    bool isAndroid = await Starflut.isAndroid();
    if( isAndroid == true ){
      String libraryDir = await Starflut.getNativeLibraryDir();
      String docPath = await Starflut.getDocumentPath();
      if( libraryDir.indexOf("arm64") > 0 ){
        Starflut.unzipFromAssets("lib-dynload-arm64.zip", docPath, true);
      }else if( libraryDir.indexOf("x86_64") > 0 ){
        Starflut.unzipFromAssets("lib-dynload-x86_64.zip", docPath, true);
      }else if( libraryDir.indexOf("arm") > 0 ){
        Starflut.unzipFromAssets("lib-dynload-armeabi.zip", docPath, true);
      }else{  //x86
        Starflut.unzipFromAssets("lib-dynload-x86.zip", docPath, true);
      }
      await Starflut.copyFileFromAssets("python3.6.zip", "flutter_assets/starfiles",null);  //desRelatePath must be null 
    }
```

Initialize python

```dart    
    if( await srvGroup.initRaw("python36", Service) == true ){
      _outputString = "init starcore and python 3.6 successfully";
      _isButtonDisabled = false;
    }else{
      _outputString = "init starcore and python 3.6 failed";
    }
```

* Input and run python scrript

```dart
  void runScriptCode() async{
    if( myController.text.length == 0 )
      return;
    await srvGroup.runScript("python", myController.text, null);

    setState((){

    });
  }
```

### 6. screenshot

![](/images/flutter_and_python_ios_screenshot.png){:width="320px"}

![](/images/flutter_and_python_android_screenshot.jpg){:width="320px"}


<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[flutter_and_python_sample.zip](/datas/flutter_and_python_sample.zip  "flutter_and_python_sample")


