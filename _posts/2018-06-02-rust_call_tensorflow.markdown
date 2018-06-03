---
layout: post
title:  "Rust Call Tensorflow"
date:   2018-06-02 12:10:07 +0800
categories: rust
tags: python rust
---

### This example shows how to call python tensorflow with rust language.

This example is based on [star_for_rust](https://github.com/srplab/starcore_for_rust), which enables rust calls other scripts.
Here, we call tensorflow python code with rust language.
The code is simple.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

### Install

* For windows, python and [cle for win32](https://github.com/srplab/starcore_for_windows) must be installed.

* For linux, python and [cle for linux](https://github.com/srplab/starcore_for_linux) must be installed.

<h1 align = "left"><font color="#FF9900">Step by Step</font></h1>

### 1. Python code to be called


  {% highlight py %}   
import tensorflow as tf
a = tf.constant(5,name="input_a")
b = tf.constant(3,name="input_b")
c = tf.multiply(a,b,name="input_c")
d = tf.add(a,b,name="input_d")
e = tf.add(c,d,name="input_c_d")
sess = tf.Session()
r = sess.run(e)
print(r)     
  {% endhighlight %}{: style="color:gray;"}

### 2. Rust code

* Cargo.toml
  {% highlight rust %}
[package]
name = "rustcalltensorflow"
version = "0.1.0"
authors = ["srplab"]

[dependencies]
starcore_for_rust = {git="https://github.com/srplab/starcore_for_rust"}
  {% endhighlight %}{: style="color:gray;"}
  
* Initialize CLE  

  {% highlight rust %}
let Service = starrust::InitSimple(&"test",&"123", 0, 0,&[]);
starrust::RegMsgCallBack_P(MsgCallBack);
let SrvGroup = Service.Get(&"_ServiceGroup").ToSrvGroup();
Service.CheckPassword(false);
  {% endhighlight %}{: style="color:gray;"}
  
* Initialize python  

  {% highlight rust %}
let initResult = SrvGroup.InitRaw(&"python", &Service);
let python = Service.ImportRawContext(&"python", &"", false, &"");
  {% endhighlight %}{: style="color:gray;"}  
  
* Call tensorflow  

  {% highlight rust %}
star_call!(python,"import","tensorflow");
let tf = python.GetObject(&"tensorflow");    

let a = star_callobject!(tf,"constant",5,star_dict!(SrvGroup,"name"=>"input_a")); 
let b = star_callobject!(tf,"constant",3,star_dict!(SrvGroup,"name"=>"input_b"));

let c = star_callobject!(tf,"multiply",a, b,star_dict!(SrvGroup,"name"=>"mul_c"));  
let d = star_callobject!(tf,"add", a, b,star_dict!(SrvGroup,"name"=>"add_d"));

let e = star_callobject!(tf,"add",c, d, star_dict!(SrvGroup,"name"=>"add_e_d"));

let sess = star_callobject!(tf,"Session",);
let r = star_callobject!(sess,"run", e);

println!("{}",r.ToString());
  {% endhighlight %}{: style="color:gray;"}    

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[rustcalltensorflow.zip](/datas/rustcalltensorflow.zip  "rustcalltensorflow")


