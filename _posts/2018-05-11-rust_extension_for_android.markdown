---
layout: post
title:  "Rust Extension for Android"
date:   2018-05-10 15:31:07 +0800
categories: rust
tags: rust java android
---

### This example shows how to write rust extension for android based on starcore_for_rust.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example shows how to write rust extension for android based on starcore_for_rust.

### 1. Install

* Using Ubuntu, and [cle for android](https://github.com/srplab/starcore_for_android) should download and uncompressed.

* Create package

  ```sh
  cargo new --lib rustsharelib
  ```
  
* Add dependencies to Cargo.toml

  {% highlight rust %}
[package]
name = "rustsharelib"
version = "0.1.0"
authors = ["srplab"]

[lib]
crate-type = ["cdylib"]
path = "src/lib.rs"

[dependencies]
starcore_for_rust = {git="https://github.com/srplab/starcore_for_rust",features = ["star-sharelib","android_arm"]}
  {% endhighlight %}{: style="color:gray;"}

<h1 align = "left"><font color="#FF9900">Create Extension</font></h1>

* Create rust extension is very simple. 

  Use macro "star_extension" to create extension body. In star_extension, create cle object, and define it's function. 

  {% highlight rust %}
#![allow(non_snake_case)]
#![allow(unused_variables)]

#[macro_use] extern crate starcore_for_rust;
use starcore_for_rust::*;
use std::os::raw::{c_void};

star_extension!(SrvGroup,Service {
  /*--create a new cle object, other script can find the object by it's name--*/
  let obj = Service.New(&[&"RustObject"]);
  /*--define function "PrintHello" of cle object--*/
  star_fn!(obj,"PrintHello",CleGroup,CleService,CleObject,a:String, b:f64 {
    println!("########{:?}",a);
    println!("########{:?}",b);
    star_ret!("return from rust");
  });     
});
  {% endhighlight %}{: style="color:gray;"}
  
* Build

  ```sh
  $ rustup target add arm-linux-androideabi
  # Create toolchain
  $ /home/xxx/Android/android-ndk-r12b/build/tools/make-standalone-toolchain.sh --platform=android-18 --toolchain=arm-linux-androideabi-4.9 --install-dir=android-18-toolchain --ndk-dir=/home/lihm/Android/android-ndk-r12b / --arch=arm
  # Create the .cargo/config file  
  $ mkdir -p .cargo
  $ echo '[build]
  target = "arm-linux-androideabi"

  [target.arm-linux-androideabi]
  linker = "/home/xxx/Desktop/rust.study/android-18-toolchain/bin/arm-linux-androideabi-gcc"' > .cargo/config  
  cargo build
  ```

  The share library "librustsharelib.so" will be generated.
  
  ![](/images/rust_extension_for_android_pic1.png){:width="320px"}

<h1 align = "left"><font color="#FF9900">Using Extension in Android App</font></h1>  

Using "DoFile" of Service Class to load the extension
  

### 1. Create Project, Add CLE and librustsharelib.so

  ![](/images/rust_extension_for_android_pic2.png){:width="320px"}
  
### 2. The Java Code  

{% highlight java %}
package com.example.srplab.testrustlib;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

import com.srplab.www.starcore.*;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        /*----init starcore----*/
        StarCoreFactoryPath.StarCoreCoreLibraryPath = this.getApplicationInfo().nativeLibraryDir;
        StarCoreFactoryPath.StarCoreShareLibraryPath = this.getApplicationInfo().nativeLibraryDir;
        StarCoreFactoryPath.StarCoreOperationPath = "/data/data/"+getPackageName()+"/files";

        final StarCoreFactory starcore= StarCoreFactory.GetFactory();
        StarServiceClass Service=starcore._InitSimple("test","123",0,0);
        starcore._RegMsgCallBack_P(new StarMsgCallBackInterface() {
            public Object Invoke(int ServiceGroupID, int uMes, Object wParam, Object lParam) {
                if (uMes == starcore._GetInt("MSG_VSDISPMSG") || uMes == starcore._GetInt("MSG_VSDISPLUAMSG")) {
                    System.out.println((String) wParam);
                }
                if (uMes == starcore._GetInt("MSG_DISPMSG") || uMes == starcore._GetInt("MSG_DISPLUAMSG")) {
                    System.out.println("++++++++++++++++" + (String) wParam);
                }
                return null;
            }
        });
        StarSrvGroupClass SrvGroup = (StarSrvGroupClass)Service._Get("_ServiceGroup");
        Service._CheckPassword(false);

        Object[] result = Service._DoFile("",this.getApplicationInfo().nativeLibraryDir+"/librustsharelib.so","");
        System.out.println(result);

        System.out.println(Service._Get("RustObject"));
        StarObjectClass RustObject = (StarObjectClass)Service._GetObject("RustObject");
        System.out.println(RustObject);
        System.out.println(RustObject._Call("PrintHello","------------1",234.56));
    }
}
{% endhighlight %}{: style="color:gray;"}

Run 

```sh
05-11 20:28:28.434 22785-22785/com.example.srplab.testrustlib I/skeletonproc_module,31657: create service group[0], free core version[2.5.2.259]
05-11 20:28:28.435 22785-22785/com.example.srplab.testrustlib I/skeletonproc_module,31661: install date[2018/5/11],please register starcore [http://www.srplab.com]
05-11 20:28:28.435 22785-22785/com.example.srplab.testrustlib I/skeletonproc_module,31700: lua engine[Lua 5.3.4  Copyright (C) 1994-2017 Lua.org, PUC-Rio]
05-11 20:28:28.455 22785-22790/com.example.srplab.testrustlib I/zygote: Do partial code cache collection, code=62KB, data=57KB
05-11 20:28:28.455 22785-22790/com.example.srplab.testrustlib I/zygote: After code cache collection, code=58KB, data=53KB
05-11 20:28:28.455 22785-22790/com.example.srplab.testrustlib I/zygote: Increasing code cache capacity to 256KB
05-11 20:28:28.461 22785-22785/com.example.srplab.testrustlib I/skeletonproc_module,1582: create service success[test]
05-11 20:28:36.498 22785-22785/com.example.srplab.testrustlib I/System.out: load share library [/data/app/com.example.srplab.testrustlib-1o6SWxDBNiXakvfL29JMwg==/lib/arm/librustsharelib.so]
05-11 20:28:36.500 22785-22785/com.example.srplab.testrustlib I/skeletonproc_module,17372: load share library [/data/app/com.example.srplab.testrustlib-1o6SWxDBNiXakvfL29JMwg==/lib/arm/librustsharelib.so]
05-11 20:28:37.277 22785-22785/com.example.srplab.testrustlib I/System.out: [Ljava.lang.Object;@ac3bebc
05-11 20:28:38.226 22785-22785/com.example.srplab.testrustlib I/System.out: RustObject
05-11 20:28:39.633 22785-22785/com.example.srplab.testrustlib I/System.out: RustObject
05-11 20:28:40.334 22785-22785/com.example.srplab.testrustlib I/System.out: return from rust
```

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[rust_extension_for_android.zip](/datas/rust_extension_for_android.zip  "rust_extension_for_android")


