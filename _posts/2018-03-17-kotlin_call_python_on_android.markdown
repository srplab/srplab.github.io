---
layout: post
title:  "Kotlin call python on Android"
date:   2018-03-16 15:31:07 +0800
categories: android
tags: android python
---

### This is an example which shows how kotlin calls python on android platform.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example is based on CLE. CLE supports android platform, python language, and interaction between ruby and kotlin(or java).

### 1. Create project and Add the dependent files

* add starcore_android_rX.XX.jar and share libraries

![](/images/2018-03-17-kotlin_call_python_on_android_project.png){:width="320px"}

### 2. Public function, copy files to the application's installation directory from assets

* copyFile

{% highlight java %}
    private fun copyFile(c: Activity, Name: String, desPath: String?) {
        var outfile: File? = null
        outfile = File("/data/data/$packageName/files/$Name")
        outfile.createNewFile()
        val out = FileOutputStream(outfile)
        val buffer = ByteArray(1024)
        val f_in: InputStream
        var readLen = 0
        if (desPath != null)
            f_in = c.assets.open(desPath + Name)
        else
            f_in = c.assets.open(Name)
        readLen = f_in.read(buffer)
        while (readLen != -1) {
            out.write(buffer, 0, readLen)
            readLen = f_in.read(buffer)
        }
        out.flush()
        f_in.close()
        out.close()
    }
{% endhighlight %}{: style="color:gray;"}


<h1 align = "left"><font color="#FF9900">Using the code</font></h1>

### 1. copy files to app folder

{% highlight java %}
        val destDir = File("/data/data/$packageName/files")
        if (!destDir.exists())
            destDir.mkdirs()
        val python2_7_libFile = java.io.File("/data/data/$packageName/files/python3.6.zip")
        if (!python2_7_libFile.exists()) {
            try {
                copyFile(this, "python3.6.zip", null)
            } catch (e: Exception) {
            }

        }
        try {
            val nativeLibraryDir = this.getApplicationInfo().nativeLibraryDir
            if (nativeLibraryDir.contains("x86")) {
                copyFile(this, "_datetime.cpython-36m.so", "x86/")
                copyFile(this, "_socket.cpython-36m.so", "x86/")
                copyFile(this, "_struct.cpython-36m.so", "x86/")
                copyFile(this, "array.cpython-36m.so", "x86/")
                copyFile(this, "binascii.cpython-36m.so", "x86/")
                copyFile(this, "select.cpython-36m.so", "x86/")
                copyFile(this, "unicodedata.cpython-36m.so", "x86/")
                copyFile(this, "zlib.cpython-36m.so", "x86/")
            } else {
                copyFile(this, "_datetime.cpython-36m.so", "armeabi/")
                copyFile(this, "_socket.cpython-36m.so", "armeabi/")
                copyFile(this, "_struct.cpython-36m.so", "armeabi/")
                copyFile(this, "array.cpython-36m.so", "armeabi/")
                copyFile(this, "binascii.cpython-36m.so", "armeabi/")
                copyFile(this, "select.cpython-36m.so", "armeabi/")
                copyFile(this, "unicodedata.cpython-36m.so", "armeabi/")
                copyFile(this, "zlib.cpython-36m.so", "armeabi/")
            }
            copyFile(this, "test_calljava.py", "")
            copyFile(this, "testpy.py", "")
        } catch (e: Exception) {
            println(e)
        }
{% endhighlight %}{: style="color:gray;"}

### 2. Load python core share library

{% highlight java %}
        val libpath = this.applicationInfo.nativeLibraryDir
        try {
            //--load python36 core library first;
            System.load(libpath + "/libpython3.6m.so")
        } catch (ex: UnsatisfiedLinkError) {
            println(ex.toString())
        }
{% endhighlight %}{: style="color:gray;"}

### 3. Initilize CLE using kotlin

{% highlight java %}
        /*----init starcore----*/
        StarCoreFactoryPath.StarCoreCoreLibraryPath = libpath //"/data/data/"+getPackageName()+"/lib";
        StarCoreFactoryPath.StarCoreShareLibraryPath = libpath //"/data/data/"+getPackageName()+"/lib";
        StarCoreFactoryPath.StarCoreOperationPath = "/data/data/$packageName/files"

        val starcore = StarCoreFactory.GetFactory()
        val Service = starcore._InitSimple("test", "123", 0, 0)
        SrvGroup = Service._Get("_ServiceGroup") as StarSrvGroupClass
        Service._CheckPassword(false)
{% endhighlight %}{: style="color:gray;"}   

### 4. Initilize python

{% highlight java %}
        SrvGroup._InitRaw("python36", Service)
        val python = Service._ImportRawContext("python", "", false, "")
        SrvGroup._RunScript("python", "import sys", "")
{% endhighlight %}{: style="color:gray;"} 
     
### 5. Set python libraries searching path

{% highlight java %}
        val pythonSys = python._GetObject("sys")
        val pythonPath = pythonSys._Get("path") as StarObjectClass
        pythonPath._Call("insert", 0, "/data/data/$packageName/files/python3.6.zip")
        pythonPath._Call("insert", 0, "/data/data/$packageName/lib")
        pythonPath._Call("insert", 0, "/data/data/$packageName/files")
{% endhighlight %}{: style="color:gray;"} 

### 6. Run python file

{% highlight java %}
        val CorePath = "/data/data/$packageName/files"
        Service._DoFile("python", CorePath + "/testpy.py", "")

        println(python._Get("g1"));
        val retobj = python._Call("tt", "hello ", "world") as StarObjectClass
        println(" "+retobj._Get(0)+" "+retobj._Get(1)+" "+retobj._Get(2))  
{% endhighlight %}{: style="color:gray;"}

* python code of "testpy.py"

{% highlight python %}
def tt(a,b) :
    print(a,b)
    return a,777
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

### 7. Python call kotlin class

* kotlin class
{% highlight java %}
class CallBackClass(Info: String) {
    init {
        println(Info)
    }

    fun SetPythonObject(rb: Any) {
        val PythonClass = rb as StarObjectClass
        val aa = ""
        val data1 = MainActivity.Host.SrvGroup._NewParaPkg("b", 789, "c", 456, "a", 123)._AsDict(true)
        val d1 = PythonClass._Call("dumps", data1, MainActivity.Host.SrvGroup._NewParaPkg("sort_keys", true)._AsDict(true))
        println("" + d1)
        val d2 = PythonClass._Call("dumps", data1)
        println("" + d2)
        val d3 = PythonClass._Call("dumps", data1, MainActivity.Host.SrvGroup._NewParaPkg("sort_keys", true, "indent", 4)._AsDict(true))
        println("" + d3)
    }
}
{% endhighlight %}{: style="color:gray;"}

* python code of "test_calljava.py"

{% highlight python %}
import os
import json

print(JavaClass)

val = JavaClass("from python")
val.SetPythonObject(json);
print("===========end=========")
{% endhighlight %}{: style="color:gray;"}

* run python code

{% highlight java %}
python._Set("JavaClass", CallBackClass::class.java)
Service._DoFile("python", CorePath + "/test_calljava.py", "")
{% endhighlight %}{: style="color:gray;"}


<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[kotlin_call_python_on_android.zip](/datas/kotlin_call_python_on_android.zip  "kotlin_call_python_on_android")


