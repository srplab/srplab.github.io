---
layout: post
title:  "Rust Extension for Other Scripts"
date:   2018-05-10 15:31:07 +0800
categories: rust
tags: rust python java csharp ruby go
---

### This example shows how to write rust extension for other scripts based on starcore_for_rust.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example shows how to write rust extension for other scripts based on starcore_for_rust.

### 1. Install

* For windows, python and [cle for win32](https://github.com/srplab/starcore_for_windows) must be installed.

* For linux, python and [cle for linux](https://github.com/srplab/starcore_for_linux) must be installed.

* Create package

  ```sh
  cargo new --lib rustsharelib
  ```
  
* Add dependencies to Cargo.toml

  {% highlight rust %}
[package]
name = "rustsharelib"
version = "0.1.0"
authors = ["srplab <srplab.cn@hotmail.com>"]

[lib]
crate-type = ["cdylib"]
path = "src/lib.rs"

[dependencies]
starcore_for_rust = {git="https://github.com/srplab/starcore_for_rust",features = ["star-sharelib"]}
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
    star_ret!(star_parapkg!(CleGroup,"return from go",345.4));
  });     
});
  {% endhighlight %}{: style="color:gray;"}
  
* Build

  ```sh
  cargo build
  ```

<h1 align = "left"><font color="#FF9900">Using Extension from Other Scripts</font></h1>  

Using "DoFile" of Service Class to load the extension
  

### 1. python

{% highlight python %}
import libstarpy
Service=libstarpy._InitSimple("test","123",0,0);
SrvGroup = Service._ServiceGroup;
Service._CheckPassword(False)

result = Service._DoFile("","./target/debug/rustsharelib.dll","");
print(result)
print(Service.RustObject)
print(Service.RustObject.PrintHello("------------1",234.56))

SrvGroup._ClearService()
libstarpy._ModuleExit()
{% endhighlight %}{: style="color:gray;"}

Run 

```sh
PS E:\rustsharelib.test\rustsharelib> python call_rust.py
(True, '')
RustObject
########"------------1"
########234.56
parapkg
```

### 2. ruby

{% highlight ruby %}
if (defined? Libstar_ruby) == nil
   require "C:\\srplab\\libs64\\libstar_ruby.so"
end  
$Service=$starruby._InitSimple("test","123",0,0,nil);
$SrvGroup = $Service._ServiceGroup;
$Service._CheckPassword(false)

result = $Service._DoFile("",$SrvGroup._GetCurrentPath() + "/target/debug/rustsharelib.dll","");
puts(result)
puts($Service.RustObject)
puts($Service.RustObject.PrintHello("------------1",234.56))

$SrvGroup._ClearService()
$starruby._ModuleExit()

#[warn(1):(ruby:32745):tm(11:40:26)]can not load ruby core share library[C:\Ruby25-x64\bin\x64-msvcrt-ruby250.dll][error code = 126
#add "C:\Ruby25-x64\bin\ruby_builtin_dlls" path for environment
{% endhighlight %}{: style="color:gray;"}


Run 

```sh
PS E:\rustsharelib.test\rustsharelib> ruby call_rust.rb
call_rust.rb:12: warning: encountered \r in middle of line, treated as a mere space
true

RustObject
########"------------1"
########234.56
parapkg
```

### 3. java

{% highlight java %}
import com.srplab.www.starcore.*;

public class call_rust{ 
  public static void main(String[] args){
    StarCoreFactory starcore= StarCoreFactory.GetFactory();
    StarServiceClass Service=starcore._InitSimple("test","123",0,0);
    StarSrvGroupClass SrvGroup = (StarSrvGroupClass)Service._Get("_ServiceGroup");
    Service._CheckPassword(false);

    Object result = Service._DoFile("",SrvGroup._GetCurrentPath() + "/target/debug/rustsharelib.dll","");
    System.out.println(result);
    System.out.println(Service._GetObject("RustObject"));
    System.out.println(Service._GetObject("RustObject")._Call("PrintHello","------------1",234.56));
		
    SrvGroup._ClearService();
    starcore._ModuleExit();
  }
}		
{% endhighlight %}{: style="color:gray;"}

Run 

```sh
javac -classpath .;c:\srplab\libs64\starcore.jar call_rust.java
java -classpath .;c:\srplab\libs64\starcore.jar call_rust
[Ljava.lang.Object;@1b28cdfa
RustObject
########"------------1"
########234.56
parapkg
```

### 4. c#

{% highlight c# %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Star_csharp;

namespace call_rust
{
    class Program
    {
        static void Main(string[] args)
        {
            StarCoreFactory starcore= StarCoreFactory.GetFactory();
            StarServiceClass Service=starcore._InitSimple("test","123",0,0);
            StarSrvGroupClass SrvGroup = (StarSrvGroupClass)Service._Get("_ServiceGroup");
            Service._CheckPassword(false);

            object result = Service._DoFile("",SrvGroup._GetCurrentPath() + "/target/debug/rustsharelib.dll","");
            Console.WriteLine(result);
            Console.WriteLine(Service._GetObject("RustObject"));
            Console.WriteLine(Service._GetObject("RustObject")._Call("PrintHello","------------1",234.56));
		
            SrvGroup._ClearService();
            starcore._ModuleExit();
        }
    }
}		
{% endhighlight %}{: style="color:gray;"}

Run 

```sh
csc /reference:c:\srplab\libs64\star_csharp462.dll /platform:x64 call_rust.cs
```

### 5. go

{% highlight go %}
// test project main.go
package main

import (
	"fmt"
	"github.com/srplab/starcore_for_go/stargo"
)

func main() {
	Service := stargo.InitSimple("test", "123", 0, 0)
	SrvGroup := Service.Get("_ServiceGroup").(*stargo.StarSrvGroup)
	Service.CheckPassword(false)

	result := Service.DoFile("",SrvGroup.GetCurrentPath() + "/target/debug/rustsharelib.dll","")
	fmt.Println(result);
	fmt.Println(Service.GetObject("RustObject"));
	fmt.Println(Service.GetObject("RustObject").Call("PrintHello","------------1",234.56));

	SrvGroup.ClearService();
	stargo.ModuleExit();
}
{% endhighlight %}{: style="color:gray;"}

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[rust_extension_for_other_scripts.zip](/datas/rust_extension_for_other_scripts.zip  "rust_extension_for_other_scripts")


