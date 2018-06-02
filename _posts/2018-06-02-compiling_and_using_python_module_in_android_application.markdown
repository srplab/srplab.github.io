---
layout: post
title:  "Compiling and Using Python Module in Android Application"
date:   2018-06-01 21:10:07 +0800
categories: android
tags: python java android
---

### This example shows how to compile python natibe module and to use it in android application.

<h1 align = "left"><font color="#FF9900">Step by Step</font></h1>

This example is based on [cle for android](https://github.com/srplab/starcore_for_android), and test with android ndk r10d.
To illustrate how to compile, we chose a python module [aiohttp](https://github.com/aio-libs/aiohttp).


### 1. Compile aiohttp with ndk

* Download and unzip aiohttp

* Copy files about python 3.6, which can be found from starcore_for_android, to the same directory with aiphttp.

  ![](/images/compiling_and_using_python_module_in_android_pci1.png){:width="640px"}

* Setup compiling environment

```sh
export ANDROID_NDK="/usr/local/android/android-ndk-r10d"
export PATH="$ANDROID_NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64/bin/:$ANDROID_NDK:$ANDROID_NDK/tools:/usr/local/bin:/usr/bin:/bin:$PATH"
export ARCH="armeabi"
export CFLAGS="-DANDROID -mandroid -fomit-frame-pointer --sysroot=$ANDROID_NDK/platforms/android-9/arch-arm -I../python-3.6/armeabi/include/python3.6m"
export CXXFLAGS="$CFLAGS"
export CC="arm-linux-androideabi-gcc"
export LDFLAGS="-lm -L../python-3.6/armeabi"
export CXX="arm-linux-androideabi-g++"
export AR="arm-linux-androideabi-ar"
export RANLIB="arm-linux-androideabi-ranlib"
export STRIP="arm-linux-androideabi-strip --strip-unneeded"
export READELF="arm-linux-androideabi-readelf"
export MAKE="make -j4 CROSS_COMPILE_TARGET=yes"
```

* Change the setup.py of aiphttp

Add the following code to function "build_extension" of class "ve_build_ext" of setup.py

  {% highlight py %}   
        if '/usr/local/include/python3.6m' in self.compiler.include_dirs:
            self.compiler.include_dirs.remove('/usr/local/include/python3.6m')
        import os
        self.compiler.linker_so[0] = os.environ.get('CC')  
        ext.extra_link_args = ['-Wl,-soname,'+build_ext.get_ext_filename(self,ext.name).split('/')[1],]        
  {% endhighlight %}{: style="color:gray;"}

Like this,

  {% highlight py %}
class ve_build_ext(build_ext):
    # This class allows C extension building to fail.

    def run(self):   
        try:
            build_ext.run(self)
        except (DistutilsPlatformError, FileNotFoundError):
            raise BuildFailed()

    def build_extension(self, ext):  
    
        if '/usr/local/include/python3.6m' in self.compiler.include_dirs:
            self.compiler.include_dirs.remove('/usr/local/include/python3.6m')
        import os
        self.compiler.linker_so[0] = os.environ.get('CC')  
        ext.extra_link_args = ['-Wl,-soname,'+build_ext.get_ext_filename(self,ext.name).split('/')[1],]
        
  {% endhighlight %}{: style="color:gray;"}
  
The above code may need to make some adjustments based on the system environment    
  
* Compile aiphttp

```sh
python3 setup.py build
```

The result is located at "aiohttp-3.1/build/lib.linux-x86_64-3.6" 

* Compile dependent modules and compress them to "aiohttp_module.zip"

aiohttp depends other python modules, such as "multidict", "yarl", etc. If the modules contains native code, it should be compiled with ndk using the above method.
Copy these modules into a folder, and compress them to "aiohttp_module.zip"

![](/images/compiling_and_using_python_module_in_android_pci2.png){:width="640px"}

### 2. Using aiohttp in android application

* Create Project, Add CLE, python libraries and aiohttp_module.zip

  ![](/images/compiling_and_using_python_module_in_android_pci3.png){:width="320px"}
  
  ![](/images/compiling_and_using_python_module_in_android_pci4.png){:width="320px"}
  
* The Python Code in "test.py"

{% highlight Python %}
import aiohttp
import asyncio

async def fetch(client):
    async with client.get('http://www.python.org') as resp:
        assert resp.status == 200
        return await resp.text()


async def main():
    connector = aiohttp.TCPConnector(verify_ssl=False)
    async with aiohttp.ClientSession(connector=connector) as client:
        html = await fetch(client)
        print(html)

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
{% endhighlight %}{: style="color:gray;"}  
  
* The Java Code  

{% highlight java %}
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        File destDir = new File("/data/data/"+getPackageName()+"/files");
        if(!destDir.exists())
            destDir.mkdirs();

        try {
            AssetManager localassetManager = getAssets();
            InputStream dataSource = localassetManager.open("aiohttp_module.zip");
            unzip(dataSource,"/data/data/"+getPackageName()+"/files",true);

            dataSource = localassetManager.open("certs.zip");
            unzip(dataSource,"/data/data/"+getPackageName()+"/files",true);

            /*---so--*/
            String nativeLibraryDir = this.getApplicationInfo().nativeLibraryDir;
            if (nativeLibraryDir.contains("x86")) {
            } else {
                copyFile(this, "_ssl.cpython-36m.so", "armeabi/");
                copyFile(this, "unicodedata.cpython-36m.so", "armeabi/");
                copyFile(this, "zlib.cpython-36m.so", "armeabi/");
                copyFile(this, "_posixsubprocess.cpython-36m.so", "armeabi/");
                copyFile(this, "select.cpython-36m.so", "armeabi/");
                copyFile(this, "math.cpython-36m.so", "armeabi/");
                copyFile(this, "array.cpython-36m.so", "armeabi/");
                copyFile(this, "_struct.cpython-36m.so", "armeabi/");
                copyFile(this, "_socket.cpython-36m.so", "armeabi/");
                copyFile(this, "_sha512.cpython-36m.so", "armeabi/");
                copyFile(this, "_random.cpython-36m.so", "armeabi/");
                copyFile(this, "_multiprocessing.cpython-36m.so", "armeabi/");
                copyFile(this, "binascii.cpython-36m.so", "armeabi/");
                copyFile(this, "_md5.cpython-36m.so", "armeabi/");
                copyFile(this, "_sha1.cpython-36m.so", "armeabi/");
                copyFile(this, "_sha3.cpython-36m.so", "armeabi/");
                copyFile(this, "_sha256.cpython-36m.so", "armeabi/");
                copyFile(this, "_blake2.cpython-36m.so", "armeabi/");
                copyFile(this, "python3.6.zip", null);

                copyFile(this, "test.py", null);
            }
        }
        catch (Exception ex){}
        try{
            //--load python36 core library first;
            System.load(this.getApplicationInfo().nativeLibraryDir+"/libpython3.6m.so");
        }
        catch(UnsatisfiedLinkError ex)
        {
            System.out.println(ex.toString());
        }

        /*----init starcore----*/
        StarCoreFactoryPath.StarCoreCoreLibraryPath = this.getApplicationInfo().nativeLibraryDir;
        StarCoreFactoryPath.StarCoreShareLibraryPath = this.getApplicationInfo().nativeLibraryDir;
        StarCoreFactoryPath.StarCoreOperationPath = "/data/data/"+getPackageName()+"/files";

        StarCoreFactory starcore= StarCoreFactory.GetFactory();
        StarServiceClass Service=starcore._InitSimple("test","123",0,0);
        StarSrvGroupClass SrvGroup = (StarSrvGroupClass)Service._Get("_ServiceGroup");
        Service._CheckPassword(false);

        /*----run python code----*/
        SrvGroup._InitRaw("python36",Service);
        StarObjectClass python = Service._ImportRawContext("python","",false,"");
        python._Call("import", "sys");

        StarObjectClass pythonSys = python._GetObject("sys");
        StarObjectClass pythonPath = (StarObjectClass)pythonSys._Get("path");
        pythonPath._Call("insert",0,"/data/data/"+getPackageName()+"/files/python3.6.zip");
        pythonPath._Call("insert",0,this.getApplicationInfo().nativeLibraryDir);
        pythonPath._Call("insert",0,"/data/data/"+getPackageName()+"/files/aiohttp_module");
        pythonPath._Call("insert",0,"/data/data/"+getPackageName()+"/files");

        python._Call("import", "aiohttp");

        StarObjectClass aiohttp = python._GetObject("aiohttp");
        SrvGroup._DoFile("python","/data/data/"+getPackageName()+"/files/test.py");
    }
{% endhighlight %}{: style="color:gray;"}

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[test_aiohttp.zip](/datas/test_aiohttp.zip  "test_aiohttp")


