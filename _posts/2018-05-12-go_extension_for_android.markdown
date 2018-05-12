---
layout: post
title:  "Go Extension for Android"
date:   2018-05-11 15:31:07 +0800
categories: go
tags: go java android
---

### This example shows how to write go extension for android based on starcore_for_go.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example shows how to write go extension for android based on starcore_for_go.

### 1. Install

* Using Ubuntu, and [cle for android](https://github.com/srplab/starcore_for_android) should download and uncompressed.

```sh
go get github.com/srplab/starcore_for_go/stargo
```

* Create go extension is simple. 

  Create go source file main.go, define "init" function.
  Register CallBack function "RegScriptInitCallBack_P", in this callback function, create cle object and define it's function

  {% highlight go %}
// main
package main

import (
	"fmt"

	"github.com/srplab/starcore_for_go/stargo"
)

func main() {
	fmt.Println("Hello World!")
}

func init() {
	stargo.RegAttachRawContextCallBack_P(func(ContextName string) interface{} {
		if ContextName == "gofunc" {
			return func(v1 string, v2 float32) string {
				stargo.Println(v1)
				stargo.Println(v2)
				return "hello fro go"
			}
		}
		return nil
	})

	stargo.RegScriptTermCallBack_P(func() {
		stargo.Println("go script engine exit...")
	})

	stargo.RegScriptInitCallBack_P(func(SrvGroup *stargo.StarSrvGroup, Service *stargo.StarService) {
		stargo.Println("go script engine init...")

		/*--GoObject can be called from other script */
		s := Service.New("GoObject")
		s.RegScriptProc_P("PrintHello", func(CleGroup *stargo.StarSrvGroup, CleService *stargo.StarService, CleObject *stargo.StarObject, Paras []interface{}) interface{} {
			stargo.Println(Paras)
			return []interface{}{"return from go", 345.4}
		})
	})
}
  {% endhighlight %}{: style="color:gray;"}
  
* Build

  Using NDK, create toolchain, and build.sh as follow,

  ```sh
  build.sh
  
  export NDK_TOOLCHAIN=/home/xxx/Desktop/android-18-toolchain
  export CC=$NDK_TOOLCHAIN/bin/arm-linux-androideabi-gcc
  export GOOS=android
  export GOARCH=arm
  export GOARM=7
  export CGO_ENABLED=1

  GO="$GOROOT/bin/go"

  $GO build -buildmode=c-shared -o libgosharelib.so
  
  chmod 0777 build.sh
  ./build.sh
  ```

  The share library "libgosharelib.so" will be generated.
  
  ![](/images/go_extension_for_android_pic1.png){:width="320px"}

<h1 align = "left"><font color="#FF9900">Using Extension in Android App</font></h1>  

Using "DoFile" of Service Class to load the extension
  

### 1. Create Project, Add CLE and librustsharelib.so

  ![](/images/go_extension_for_android_pic2.png){:width="320px"}
  
### 2. The Java Code  

{% highlight java %}
package com.example.srplab.testgosharelib;

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

        Object[] result = Service._DoFile("",this.getApplicationInfo().nativeLibraryDir+"/libgosharelib.so","");
        System.out.println(result);

        System.out.println(Service._Get("GoObject"));
        StarObjectClass GoObject = (StarObjectClass)Service._GetObject("GoObject");
        System.out.println(GoObject);
        System.out.println(GoObject._Call("PrintHello","------------1",234.56));
    }
}
{% endhighlight %}{: style="color:gray;"}

Run 

```sh
05-12 09:51:31.369 3277-3277/com.example.srplab.testgosharelib I/skeletonproc_module,31657: create service group[0], free core version[2.5.2.259]
05-12 09:51:31.370 3277-3277/com.example.srplab.testgosharelib I/skeletonproc_module,31661: install date[2018/5/12],please register starcore [http://www.srplab.com]
05-12 09:51:31.370 3277-3277/com.example.srplab.testgosharelib I/skeletonproc_module,31700: lua engine[Lua 5.3.4  Copyright (C) 1994-2017 Lua.org, PUC-Rio]
05-12 09:51:31.398 3277-3277/com.example.srplab.testgosharelib I/skeletonproc_module,1582: create service success[test]
05-12 09:51:35.219 3277-3277/com.example.srplab.testgosharelib I/System.out: ++++++++++++++++[go script engine init...]
05-12 09:51:35.222 3277-3277/com.example.srplab.testgosharelib I/System.out: ++++++++++++++++
05-12 09:51:35.223 3277-3277/com.example.srplab.testgosharelib I/go,0: [go script engine init...]
05-12 09:51:35.225 3277-3277/com.example.srplab.testgosharelib I/System.out: load share library [/data/app/com.example.srplab.testgosharelib-a5bAOE5HeLMLfpvLgEd8eQ==/lib/arm/libgosharelib.so]
05-12 09:51:35.226 3277-3277/com.example.srplab.testgosharelib I/skeletonproc_module,17372: load share library [/data/app/com.example.srplab.testgosharelib-a5bAOE5HeLMLfpvLgEd8eQ==/lib/arm/libgosharelib.so]
05-12 09:51:35.969 3277-3277/com.example.srplab.testgosharelib I/System.out: [Ljava.lang.Object;@1a35e8
05-12 09:51:36.973 3277-3277/com.example.srplab.testgosharelib I/System.out: GoObject
05-12 09:51:38.543 3277-3277/com.example.srplab.testgosharelib I/System.out: GoObject
05-12 09:51:39.123 3277-3277/com.example.srplab.testgosharelib I/System.out: ++++++++++++++++[[------------1 234.56]]
05-12 09:51:39.130 3277-3277/com.example.srplab.testgosharelib I/System.out: ++++++++++++++++
05-12 09:51:39.131 3277-3277/com.example.srplab.testgosharelib I/go,0: [[------------1 234.56]]
05-12 09:51:39.140 3277-3277/com.example.srplab.testgosharelib I/System.out: parapkg
```

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[go_extension_for_android.zip](/datas/go_extension_for_android.zip  "go_extension_for_android")


