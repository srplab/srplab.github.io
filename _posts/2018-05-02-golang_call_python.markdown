---
layout: post
title:  "GoLang Call Python"
date:   2018-05-02 15:31:07 +0800
categories: go
tags: go python
---

### This example shows how to use starcore_for_go, which enable go language to call python.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example shows how to use starcore_for_go, which enable go language to call python, including calling python functions, creating instances of python classes, getting and setting Python variables, capturing python printouts, and setting python callbacks, etc.

### 1. Install

* For windows, python and [cle for win32](https://github.com/srplab/starcore_for_windows) must be installed.

* For linux, python and [cle for linux](https://github.com/srplab/starcore_for_linux) must be installed.

* Install starfore_for_go

  ```sh
  go get github.com/srplab/starcore_for_go/stargo
  ```

<h1 align = "left"><font color="#FF9900">Using the code</font></h1>

### 1. Init cle using starcore_for_go

  {% highlight go %}
    import (
      	"fmt"
       	"github.com/srplab/starcore_for_go/stargo"
    )

    func main() {
       	Service := stargo.InitSimple("test", "123", 0, 0)
       	SrvGroup := Service.Get("_ServiceGroup").(*stargo.StarSrvGroup)
       	Service.CheckPassword(false)
    }
  {% endhighlight %}{: style="color:gray;"}

### 2. Loading and initializing python

{% highlight go %}
    SrvGroup.InitRaw("python", Service)
{% endhighlight %}{: style="color:gray;"}
    
* Load python2.7 by default, you can use python35, python36, etc. to load different versions

* Get python global context, with it, app can get or set python global variables, classes or functions.

{% highlight go %}
    python := Service.ImportRawContext("python", "", false, "")
{% endhighlight %}{: style="color:gray;"}    

### 3. Set callbacks to capture output from python

* Callback function

{% highlight go %}
    func MsgCallBack(ServiceGroupID uint32, uMsg uint32, wParam interface{}, lParam interface{}) (IsProcessed bool, Result interface{}) {
    	if uMsg == stargo.MSG_VSDISPMSG || uMsg == stargo.MSG_VSDISPLUAMSG || uMsg == stargo.MSG_DISPMSG || uMsg == stargo.MSG_DISPLUAMSG {
		    fmt.Println(wParam)
    	} else {
		    fmt.Println(ServiceGroupID, uMsg, wParam, lParam)
    	}
    	return false, 0
    }
{% endhighlight %}{: style="color:gray;"}   

* Set Callback function

{% highlight go %}
    func main() {
    	Service := stargo.InitSimple("test", "123", 0, 0)
    	SrvGroup := Service.Get("_ServiceGroup").(*stargo.StarSrvGroup)
    	Service.CheckPassword(false)

    	stargo.RegMsgCallBack_P(MsgCallBack)

        ...
    }
{% endhighlight %}{: style="color:gray;"}   


### 4. Interoperation with python

* Run python file

{% highlight go %}
    Service.DoFile("python", "testpy.py", "")
{% endhighlight %}{: style="color:gray;"} 

* Get python global variable

{% highlight go %}
    fmt.Println(python.Get("g1"))
{% endhighlight %}{: style="color:gray;"} 

* Python class related operations

{% highlight go %}
    //Get python class
    Multiply := Service.ImportRawContext("python", "Multiply", true, "")
    //Create instance of python class
    multiply := Multiply.New("", "", 33, 44)
    //Invoke function of python instance
    fmt.Printf("instance multiply = %d\n", multiply.Call("multiply", 11, 22))
{% endhighlight %}{: style="color:gray;"} 
    
### 5. Set callback to python

{% highlight go %}
    Service.DoFile("python", " testcallback.py", "")

    CallBackObj := Service.New()
    python.Set("CallBackObj", CallBackObj)

    CallBackObj.RegScriptProc_P("PrintHello", func(CleGroup *stargo.StarSrvGroup, CleService *stargo.StarService, CleObject *stargo.StarObject, Paras []interface{}) interface{} {
        fmt.Println(Paras)
        return []interface{}{"return from go", 345.4}
    })

    retobj := python.Call("TestCallBack", "hello ", "world").(*stargo.StarObject)
    fmt.Println(retobj)
{% endhighlight %}{: style="color:gray;"} 

### 6. Run

  ```sh
  go build
  
  load library error[126], try[C:\srplab/libs64/star_python36.dll]
  load library error[126], try[C:\WINDOWS\system32/star_python36.dll]
  load library error[126], try[C:\srplab/libs64/star_python36.pyd]
  load library [C:\srplab/libs64/star_python36.pyd] success....
  basic script [python] for version [36] is registered, you can use [python] to interact with the script
  123
  multiply.... <__main__.Multiply object at 0x0000000006D69320> 11 22
  instance multiply = 242
  hello  world
  [--------------- 1234]
  return from go
  345.4
  Name60E15194[python36:tuple](666, 777)  
  ```

* python code of "testpy.py"

{% highlight python %}
def tt(a,b) :
    print(a,b)
    return 666,777
g1 = 123
def yy(a,b,z) :
    print(a,b,z)
    return {'jack': 4098, 'sape': 4139}

class Multiply :
    def __init__(self,x,y) :
        self.a = x
        self.b = y
    
    def multiply(self,a,b):
        print("multiply....",self,a,b)
        return a * b
{% endhighlight %}{: style="color:gray;"}

* python code of "testcallback.py"

{% highlight python %}
def TestCallBack(a,b) :
    print(a,b)
    ParaPkg = CallBackObj.PrintHello("---------------",1234)
    print(ParaPkg[0])
    print(ParaPkg[1])
    return 666,777

{% endhighlight %}{: style="color:gray;"}

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[go_call_python.zip](/datas/go_call_python.zip  "go_call_python")


