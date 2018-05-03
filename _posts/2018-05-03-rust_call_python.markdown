---
layout: post
title:  "Rust Call Python"
date:   2018-05-03 15:31:07 +0800
categories: rust
tags: rust python
---

### This example shows how to use starcore_for_rust, which enable rust language to call python.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example shows how to use starcore_for_rust, which enable rust language to call python, including calling python functions, creating instances of python classes, getting and setting Python variables, capturing python printouts, and setting python callbacks, etc.

### 1. Install

* For windows, python and [cle for win32](https://github.com/srplab/starcore_for_windows) must be installed.

* For linux, python and [cle for linux](https://github.com/srplab/starcore_for_linux) must be installed.

* Create package

  ```sh
  cargo new rustcallpython
  ```
  
* Add dependencies  

  ```sh
  [dependencies]
  starcore_for_rust = {git="https://github.com/srplab/starcore_for_rust"}
  ```

<h1 align = "left"><font color="#FF9900">Using the code</font></h1>

### 1. Init cle with starcore_for_rust

  {% highlight rust %}
#![allow(unused_variables)]
#![allow(non_snake_case)]
#![allow(unused_imports)]
#![allow(dead_code)]

extern crate starcore_for_rust;
use starcore_for_rust::*;

fn main() {
	let Service = starrust::InitSimple(&"test",&"123", 0, 0,&[]);
	let SrvGroup = Service.Get(&"_ServiceGroup").ToSrvGroup();
    Service.CheckPassword(false);    
}
  {% endhighlight %}{: style="color:gray;"}

### 2. Loading and initializing python

{% highlight rust %}
let initResult = SrvGroup.InitRaw(&"python36", &Service);
{% endhighlight %}{: style="color:gray;"}
    
* Load python2.7 by default, you can use python35, python36, etc. to load different versions

* Get python global context, with it, app can get or set python global variables, classes or functions.

{% highlight rust %}
let python = Service.ImportRawContext(&"python", &"", false, &"");
{% endhighlight %}{: style="color:gray;"}    

### 3. Set callbacks to capture output from python

* Callback function

{% highlight rust %}
fn MsgCallBack(ServiceGroupID: u32, uMsg: u32, wParam: &Any, lParam: &Any) -> (bool, Box<Any>)
{
	if uMsg == MSG_VSDISPMSG || uMsg == MSG_VSDISPLUAMSG || uMsg == MSG_DISPMSG || uMsg == MSG_DISPLUAMSG {
		println!("{}",starrust::ToString(wParam));
	} else {
	}
	return (false, Box::new(&0));
}
{% endhighlight %}{: style="color:gray;"}   

* Set Callback function

{% highlight rust %}
fn main() {
	let Service = starrust::InitSimple(&"test",&"123", 0, 0,&[]);
	let SrvGroup = Service.Get(&"_ServiceGroup").ToSrvGroup();
    Service.CheckPassword(false);

    starrust::RegMsgCallBack_P(MsgCallBack);
    ...
}
{% endhighlight %}{: style="color:gray;"}   


### 4. Interoperation with python

* Run python file

{% highlight rust %}
Service.DoFile(&"python", &"testpy.py", &"");
{% endhighlight %}{: style="color:gray;"} 

* Get python global variable

{% highlight rust %}
println!("{}",python.GetInt(&"g1"));
{% endhighlight %}{: style="color:gray;"} 

* Python class related operations

{% highlight rust %}
//Get python class
let Multiply = Service.ImportRawContext(&"python", &"Multiply", true, &"");
//Create instance of python class
let multiply = Multiply.New(&[&"",&"",&33,&44]);
//Invoke function of python instance
println!("instance multiply = {:?}", multiply.CallInt(&"multiply", &[&11, &22]));
{% endhighlight %}{: style="color:gray;"} 
    
### 5. Set callback to python

{% highlight rust %}
fn rustcallback(CleGroup:&STARSRVGROUP,CleService:&STARSERVICE,CleObject:&starrust::STAROBJECT,Paras: &[starrust::STARRESULT]) -> starrust::STARRESULT {
    starrust::println(format!("{}",Paras[0].ToString()));
    starrust::println(format!("{}",Paras[1].ToInt()));
	return Some(Box::new(CleGroup.NewParaPkg(&[&"return from go", &345.4])));
}
{% endhighlight %}{: style="color:gray;"} 

{% highlight rust %}
Service.DoFile(&"python", &"testcallback.py", &"");
let CallBackObj = Service.New(&[]);
python.Set(&"CallBackObj", &CallBackObj);
CallBackObj.RegScriptProc_P(&"PrintHello",rustcallback);
let retobj = python.Call(&"TestCallBack", &[&"hello ", &"world"]);
println!("{:?}",retobj.Type());
println!("{:?}",retobj.ToObject().GetInt(&0));
println!("{:?}",retobj.ToObject().GetInt(&1));
{% endhighlight %}{: style="color:gray;"} 

### 6. Run

```sh
cargo run
  
load library error[126], try[C:\srplab/libs64/star_python36.dll]
load library error[126], try[C:\WINDOWS\system32/star_python36.dll]
load library error[126], try[C:\srplab/libs64/star_python36.pyd]
load library [C:\srplab/libs64/star_python36.pyd] success....
basic script [python] for version [36] is registered, you can use [python] to interact with the script
123
multiply.... <__main__.Multiply object at 0x000000000650A6A0> 11 22
instance multiply = 242
hello  world
---------------
1234
return from go
345.4
STAROBJECT
666
777 
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

[rust_call_python.zip](/datas/rust_call_python.zip  "rust_call_python")


