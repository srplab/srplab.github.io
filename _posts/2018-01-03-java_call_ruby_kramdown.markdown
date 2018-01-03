---
layout: post
title:  "Java Call Ruby Kramdown"
date:   2018-01-03 20:09:07 +0800
categories: java
---
{% include codeformat.html %}

<h1 align = "left"><font color="#FF9900">Introduction</font></h1>

kramdown can convert markdown file to html, which is written in pure ruby.

The example shows how java calls kramdown to convert markdown file to html.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example is based on CLE. CLE supports multiple platforms, ruby language, and interaction between ruby and delphi.

The sample is developed with java, ruby 2.2(Installed with RubyInstaller), and kramdown-1.16.2.

### 1. Install CLE

*   Download and install CLE for win32

    [starcore_for_windows](https://github.com/srplab/starcore_for_windows  "starcore_for_windows")
    
    ![](/images/install_starcore_win32_2_5_2.png){:width="320px"}

### 2. Install ruby and kramdown
    
*   skip
       
    
<h1 align = "left"><font color="#FF9900">Using the code</font></h1>
    

### 1. Initialize CLE

*   initialize CLE

    {% highlight java %}
final StarCoreFactory starcore= StarCoreFactory.GetFactory();
StarServiceClass Service=starcore._InitSimple("test","123",0,0);
StarSrvGroupClass SrvGroup = (StarSrvGroupClass)Service._Get("_ServiceGroup");
Service._CheckPassword(false);
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}  
    
*   define callback function

    {% highlight java %}
starcore._RegMsgCallBack_P(new StarMsgCallBackInterface(){
    public Object Invoke(int ServiceGroupID, int uMes, Object wParam, Object lParam){
        if (uMes == starcore._Getint("MSG_DISPMSG") || uMes == starcore._Getint("MSG_DISPLUAMSG") ||
            uMes == starcore._Getint("MSG_VSDISPMSG") || uMes == starcore._Getint("MSG_VSDISPLUAMSG"))
        {
            String Str;
            Str = (String)wParam;
            System.out.println(Str);
        }
        return null;
    }
});
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}     

### 2. Initialize ruby and set ruby libraries searching path

*  initialize ruby

{% highlight java %}
starcore._SetScript("ruby","", "-v 2.2.0");
SrvGroup._InitRaw("ruby",Service);
StarObjectClass ruby = Service._ImportRawContext("ruby","",false,"");
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 
     
*  ser ruby library search path

{% highlight c++ %}
StarObjectClass LOAD_PATH = (StarObjectClass)ruby._R("LOAD_PATH");
LOAD_PATH._Call("unshift", "C:\\Ruby22\\lib\\ruby\\gems\\2.2.0\\gems\\kramdown-1.16.2\\lib");
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 
     
### 3. Call kramdown

{% highlight java %}
ruby._Call("require","kramdown");
StarObjectClass KramdownDocument = (StarObjectClass)ruby._Call("eval","Kramdown::Document");
StarObjectClass resobj = KramdownDocument._New("","","# aaaaaa");
Object res = resobj._Call("method_missing","to_html");
System.out.println(res);
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}

### 4. Compile and Run

*   compile : javac -classpath .;c:\srplab\libs\starcore.jar java_call_ruby_kramdown.java

*   run : java -classpath .;c:\srplab\libs\starcore.jar java_call_ruby_kramdown

*   You can also add "starcore.jar" to environment variable "CLASSPATH"

### 5. Using SRPWatch to capture output from cle

*   output

    ![](/images/java_call_ruby_kramdown_srpwatch.png){:width="320px"}

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

*   download from here

    [java_call_ruby_kramdown.zip](/datas/java_call_ruby_kramdown.zip  "java_call_ruby_kramdown")

<h1 align = "left"><font color="#FF9900">History</font></h1>


