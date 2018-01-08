---
layout: post
title:  "UWP C# Call Ruby Kramdown"
date:   2018-01-06 20:26:07 +0800
categories: win10_uwp_csharp
tags: csharp ruby uwp
---

### The example shows how c# calls kramdown to convert markdown file to html in application written for UWP(Universal Windows Platform).

{% include codeformat.html %}

<h1 align = "left"><font color="#FF9900">Introduction</font></h1>

kramdown can convert markdown file to html, which is written in pure ruby.

The example shows how c# calls kramdown to convert markdown file to html in application written for UWP(Universal Windows Platform).

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example is based on CLE. CLE supports multiple platforms, ruby language, and interaction between ruby and c#.

The sample is developed with c#, ruby 2.2, and kramdown-1.16.2.

### 1. Install CLE

*   Download and unzip starcore for uwp

    [starcore_for_winuap](https://github.com/srplab/starcore_for_windows10_uwp  "starcore_for_winuap")
    
    ![](/images/uwp_csharp_call_ruby_kramdown_cle.png){:width="640px"}

### 2. Create new Windows Universal project
    
*   create new project
    
    ![](/images/uwp_csharp_call_ruby_kramdown_create_project.png){:width="640px"}
    
### 3. Set project parameters

*   Add reference, Star_csharp and Libstarcore
    
    ![](/images/uwp_csharp_call_ruby_kramdown_add_reference.png){:width="640px"}

*   Add ruby share libraries
    
    ![](/images/uwp_csharp_call_ruby_kramdown_add_ruby_share_library.png){:width="640px"}      
    
    ![](/images/uwp_csharp_call_ruby_kramdown_set_ruby_library_property.png){:width="640px"}   
    
*   Add ruby libraries and kramdown files
  
    ![](/images/uwp_csharp_call_ruby_kramdown_add_ruby_file_karmdown.png){:width="640px"}     
    
<h1 align = "left"><font color="#FF9900">Using the code</font></h1>
    

### 1. Initialize CLE

*   initialize CLE

    {% highlight c# %}
StarCoreFactoryInit.Init(this);

StarCoreFactory starcore = StarCoreFactory.GetFactory();
StarServiceClass Service = (StarServiceClass)starcore._InitSimple("test", "123", 0, 0, null);
StarSrvGroupClass SrvGroup = (StarSrvGroupClass)Service._Get("_ServiceGroup");
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}  
    
*   define callback function

    {% highlight c# %}
starcore._RegMsgCallBack_P(new StarMsgCallBackInterface(delegate (int ServiceGroupID, int uMes, object wParam, object lParam) {
    if (uMes == starcore._Getint("MSG_VSDISPMSG") || uMes == starcore._Getint("MSG_VSDISPLUAMSG") || uMes == starcore._Getint("MSG_DISPMSG") || uMes == starcore._Getint("MSG_DISPLUAMSG"))
    {
        Debug.WriteLine((string)wParam);
    }
    return null;
}));
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}     
    
### 2. Unzip ruby libraries and kramdown files to local folder

*  Unzip ruby libraries

    {% highlight c# %}
try
{
    ZipFile.ExtractToDirectory(Package.Current.InstalledLocation.Path + "\\ruby.zip", ApplicationData.Current.LocalFolder.Path);
}
catch(Exception)
{

}
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 

*  Unzip kramdown files

    {% highlight c# %}
try
{
    ZipFile.ExtractToDirectory(Package.Current.InstalledLocation.Path + "\\kramdown-1.16.2.zip", ApplicationData.Current.LocalFolder.Path);
}
catch(Exception)
{

}
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 

### 3. Initialize ruby and set ruby libraries searching path

*  initialize ruby

{% highlight c# %}
bool InitRawFlag = SrvGroup._InitRaw("ruby", Service);
dynamic ruby = Service._ImportRawContext("ruby", "", false, "");
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 
     
*  ser ruby library search path

{% highlight c# %}
dynamic LOAD_PATH = ruby.LOAD_PATH;
LOAD_PATH.unshift(Package.Current.InstalledLocation.Path);
LOAD_PATH.unshift(ApplicationData.Current.LocalFolder.Path+ "\\ruby\\2.2.0");
LOAD_PATH.unshift(ApplicationData.Current.LocalFolder.Path + "\\kramdown-1.16.2\\lib");
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 
     
### 4. Call kramdown

{% highlight c# %}
ruby.require("kramdown");
dynamic KramdownDocument = ruby.eval("Kramdown::Document");
dynamic resobj = KramdownDocument._New("", "", "# aaaaaa");
string res = resobj.method_missing("to_html");
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

*   download from here

    [uwp_csharp_call_ruby_kramdown.zip](/datas/uwp_csharp_call_ruby_kramdown.zip  "uwp_csharp_call_ruby_kramdown")

<h1 align = "left"><font color="#FF9900">History</font></h1>


