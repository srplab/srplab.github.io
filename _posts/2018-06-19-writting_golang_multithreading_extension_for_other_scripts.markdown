---
layout: post
title:  "Writting Go Lang Multi-Threading Extension for Other Scripts"
date:   2018-06-19 15:31:07 +0800
categories: go
tags: go
---

### This example shows how to write go lang multi-threading extension for other scripts based on starcore_for_go.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example shows how to write go lang multi-threading extension for other scripts based on starcore_for_go.

### 1. Install

* For windows, python and [cle for win32](https://github.com/srplab/starcore_for_windows) must be installed.

* For linux, python and [cle for linux](https://github.com/srplab/starcore_for_linux) must be installed.

```sh
go get github.com/srplab/starcore_for_go/stargo
```

### 2. Access cle objects in threads.

After cle is initialized, the global lock is locked. The function or method that the thread calls the cle object needs to be locked first with stargo.SRPLock. After the end of the call, unlock with stargo.SRPUnLock. The caller needs to call stargo.SRPDispatch or SRPUnLock to allow the go thread to run.

### 3. Create go extension. 

  Create go source file main.go, define "init" function.
  Register CallBack function "RegScriptInitCallBack_P", in this callback function, create cle object and define it's function
  
  The code is relatively simple and does not require much explanation

  {% highlight go %}
// main
package main

import (
	"fmt"
	"time"

	"github.com/srplab/starcore_for_go/stargo"
)

func main() {
	fmt.Println("Hello World!")
}

func print_time(LuaObj *stargo.StarObject, threadName string, delay float64) {
	var count int
	count = 0
	stargo.Println(threadName)
	stargo.Println(delay)
	for {
		if count > 100000 {
			break
		}
		time.Sleep(time.Duration(delay) * time.Second)
		stargo.Println(threadName)
		stargo.Println(delay)
		count = count + 1
		stargo.SRPLock()
		stargo.Println(LuaObj)
		LuaObj.Call("Print")
		stargo.SRPUnLock()
	}
}

func init() {
	stargo.RegScriptTermCallBack_P(func() {
		stargo.Println("go script engine exit...")
	})

	stargo.RegScriptInitCallBack_P(func(SrvGroup *stargo.StarSrvGroup, Service *stargo.StarService) {
		stargo.Println("go script engine init...")

		/*--testgo can be called from other script */
		s := Service.New("testgo")
		s.RegScriptProc_P("tt", func(CleGroup *stargo.StarSrvGroup, CleService *stargo.StarService, CleObject *stargo.StarObject, Paras []interface{}) interface{} {
			stargo.Println(Paras)
			LuaObj := CleService.GetObject("LuaObj")
			stargo.Println(LuaObj)
			go print_time(LuaObj, "Thread-1", 1.0)
			go print_time(LuaObj, "Thread-2", 2.0)

			return []interface{}{666, 777}
		})
		s.RegScriptProc_P("tt1", func(CleGroup *stargo.StarSrvGroup, CleService *stargo.StarService, CleObject *stargo.StarObject, Paras []interface{}) interface{} {
			stargo.Println(Paras)
			return []interface{}{666, 777}
		})
	})
}
  {% endhighlight %}{: style="color:gray;"}
  
### 4. Build and use the extension in lua app. 

* Build

```sh
go build -buildmode=c-shared -o testgo.dll
```

* lua app

  {% highlight lua %}
Service=libstarcore._InitSimple("test","123",0,0,nil);
SrvGroup = Service._ServiceGroup;
Service:_CheckPassword(false)

result = Service:_DoFile("","./testgo.dll","");
print(result);

LuaObj = Service:_New("LuaObj")
Python = Service.testgo
print(Python)

function LuaObj:Print()
    print("hello from lua")
    print(Python)
    libstarcore._SRPUnLock()
    libstarcore._SRPLock()    
end    

Python:tt(222,333)

function aa()
  -- libstarcore._SRPUnLock()
  i = 0
  j = 0
  while( true ) do
      libstarcore._SRPDispatch(true)
      if i == 10 then
          print("----------------------------------------")
          Python:tt1("123",j);
          j = j + 1
          i = 0
      end
      i = i + 1
  end  
end

aa()


libstarcore._SRPUnLock()

while( true ) do
  
end


SrvGroup:_ClearService()
libstarcore._ModuleExit()
  {% endhighlight %}{: style="color:gray;"}
  
* Run

```sh
"C:\Program Files\srplab\starcore\starapp" --print -e call_go.lua
```

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[writting_golang_multithreading_extension_for_other_scripts.zip](/datas/writting_golang_multithreading_extension_for_other_scripts.zip  "writting_golang_multithreading_extension_for_other_scripts")


