---
layout: post
title:  "Windows Desktop C# Call Ruby Kramdown"
date:   2017-12-30 19:53:07 +0800
categories: windows_desktop_csharp
---
{% include codeformat.html %}

<h1 align = "left"><font color="#FF9900">Introduction</font></h1>

kramdown can convert markdown file to html, which is written in pure ruby.

The example shows how to use kramdown to convert markdown file to html in apps developed with c# for windows desktop.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example is based on CLE. CLE supports windows platform, ruby language, and interaction between ruby and delphi.

The sample is developed with virsual studio 2017, ruby 2.2(Installed with RubyInstaller), and kramdown-1.16.2.

### 1. Install CLE

*   Download and install CLE for win32

    [starcore_for_windows](https://github.com/srplab/starcore_for_windows  "starcore_for_windows")
    
    ![](/images/install_starcore_win32_2_5_2.png){:width="320px"}

### 2. Create c# project
    
*   create a windows classic desktop project
    
    ![](/images/windows_csharp_create_project.png){:width="320px"}

### 3. Add Star_csharp reference

*   Add star_csharp62 to the project

    ![](/images/windows_csharp_add_star_csharp.png){:width="320px"}    

<h1 align = "left"><font color="#FF9900">Using the code</font></h1>
    

### 1. Initialize CLE

*   Using Star_csharp

    {% highlight c# %}
using Star_csharp;
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}  
    
*   Use the following code to initialize CLE

    {% highlight c# %}    
StarCoreFactory starcore = StarCoreFactory.GetFactory();
starcore._RegMsgCallBack_P((int ServiceGroupID, int uMes, Object wParam, Object lParam) => 
{
    if (uMes == starcore._Getint("MSG_DISPMSG") || uMes == starcore._Getint("MSG_DISPLUAMSG") || uMes == starcore._Getint("MSG_VSDISPMSG") || uMes == starcore._Getint("MSG_VSDISPLUAMSG"))
    {
        Debug.WriteLine((String)wParam);
    }
    return null;
});
StarServiceClass Service = starcore._InitSimple("test", "123", 0, 0, null);
StarSrvGroupClass SrvGroup = (StarSrvGroupClass)Service._Get("_ServiceGroup");
Service._CheckPassword(false);
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}      

### 2. Initialize ruby and set ruby libraries searching path

*  kramdown-1.16.2 library path must be set to ruby before load.

{% highlight c# %}
SrvGroup._InitRaw("ruby", Service);
dynamic varruby = Service._ImportRawContext("ruby", "", false, "");

dynamic LOAD_PATH = varruby.LOAD_PATH;
LOAD_PATH.unshift("C:\\Ruby22\\lib\\ruby\\gems\\2.2.0\\gems\\kramdown-1.16.2\\lib");
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 
     
### 3. Call kramdown

{% highlight c# %}
varruby.require("kramdown");
dynamic KramdownDocument= varruby.eval("Kramdown::Document");
dynamic resobj = KramdownDocument._New("","","# aaaaaa");

dynamic Result  = resobj.method_missing("to_html");
string str = Result as string;
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

*   download from here

    [windows_csharp_call_ruby_kramdown.zip](/datas/windows_csharp_call_ruby_kramdown.zip  "windows_csharp_call_ruby_kramdown")

<h1 align = "left"><font color="#FF9900">History</font></h1>


