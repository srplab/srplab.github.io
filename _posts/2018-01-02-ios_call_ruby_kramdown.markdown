---
layout: post
title:  "ios Call Ruby Kramdown"
date:   2018-01-02 20:47:07 +0800
categories: ios
tags: ios ruby xcode
---

### The example shows how to use kramdown to convert markdown file to html in apps developed with object-c/c++ for iphone.

{% include codeformat.html %}

<h1 align = "left"><font color="#FF9900">Introduction</font></h1>

kramdown can convert markdown file to html, which is written in pure ruby.

The example shows how to use kramdown to convert markdown file to html in apps developed with object-c/c++ for iphone.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example is based on CLE. CLE supports windows platform, ruby language, and interaction between ruby and delphi.

The sample is developed with xcode, ruby 2.2, and kramdown-1.16.2.

### 1. Install CLE

*   Download and unzip starcore for ios

    [starcore_for_ios](https://github.com/srplab/starcore_for_ios  "starcore_for_ios")
    
    ![](/images/install_starcore_ios_2_5_2.png){:width="640px"}

### 2. Create new xcode project
    
*   create a single view application
    
    ![](/images/ios_xcode_singleview_project.png){:width="640px"}

### 3. Set project parameters

*   Set preprocessor macros, add "ENV_IOS"

    ![](/images/ios_xcode_project_macros.png){:width="640px"}    
    
*   Set "header search paths" and "library search paths"

    ![](/images/ios_xcode_project_path.png){:width="640px"}    
    
*   Change file "ViewController.m" to "ViewController.mm"

    ![](/images/ios_xcode_project_file_tomm.png)  
    
*   Add link binary "libiconv.tbd", etc

    ![](/images/ios_call_ruby_kramdown_add_binary.png){:width="640px"}      
    
*   Add ruby libraries

    ![](/images/ios_call_ruby_kramdown_add_ruby_files.png)
    
*   Add kramdown-1.16.2.

    ![](/images/ios_call_ruby_kramdown_add_kramdown.png){:width="640px"}     
    
<h1 align = "left"><font color="#FF9900">Using the code</font></h1>
    

### 1. Initialize CLE

*   add "vsopenapi.h"

    {% highlight c++ %}
#include "vsopenapi.h"
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}  
    
*   define callback function

    {% highlight c++ %}    
static class ClassOfSRPInterface *SRPInterface;

static VS_UWORD MsgCallBack( VS_ULONG ServiceGroupID, VS_ULONG uMsg, VS_UWORD wParam, VS_UWORD lParam, VS_BOOL *IsProcessed, VS_UWORD Para )
{
    switch( uMsg ){
        case MSG_VSDISPMSG :
        case MSG_VSDISPLUAMSG :
            printf("[core]%s\n",(VS_CHAR *)wParam);
            break;
        case MSG_DISPMSG :
        case MSG_DISPLUAMSG :
            printf("%s\n",(VS_CHAR *)wParam);
            break;
    }
    return 0;
}
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}      

### 2. Initialize cle,ruby and set ruby libraries searching path

*  initialize cle

{% highlight c++ %}
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,   NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];
    
    const char* destDir = [documentsDirectory UTF8String];
    VS_BOOL Result = StarCore_Init((VS_CHAR *)destDir);
    
    VSCoreLib_InitRuby();
    
    VS_CORESIMPLECONTEXT Context;
    
    SRPInterface = VSCoreLib_InitSimple(&Context,"test","123",0,0,MsgCallBack,0,NULL);
    SRPInterface ->CheckPassword(VS_FALSE);
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 

*  initialize ruby

{% highlight c++ %}
    NSString *respaths = [[NSBundle mainBundle] resourcePath];
    const VS_CHAR *res_cpath = [respaths UTF8String];
    
    class ClassOfBasicSRPInterface *BasicSRPInterface;
    
    BasicSRPInterface = SRPInterface ->GetBasicInterface();
    BasicSRPInterface ->InitRaw("ruby", SRPInterface);
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 
     
*  ser ruby library search path

{% highlight c++ %}
    void *ruby = SRPInterface -> ImportRawContext("ruby", "", VS_FALSE, "");
    void *LOAD_PATH = (void *)SRPInterface -> ScriptGetObject(ruby,"LOAD_PATH", NULL);
    
    char pathbuf[512];
    sprintf(pathbuf,"%s/ruby/2.2.0",res_cpath);
    SRPInterface->ScriptCall(LOAD_PATH,NULL, "unshift", "(s)",pathbuf);
    
    sprintf(pathbuf,"%s/kramdown-1.16.2/lib",res_cpath);
    SRPInterface->ScriptCall(LOAD_PATH,NULL, "unshift", "(s)",pathbuf);
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 
     
### 3. Call kramdown

{% highlight c++ %}
    SRPInterface->ScriptCall(ruby,NULL, "require", "(s)","kramdown");
    void *KramdownDocument = (void *)SRPInterface->ScriptCall(ruby,NULL, "eval", "(s)o","Kramdown::Document");
    
    void *resobj = (void *)SRPInterface ->IMallocObjectL2(SRPInterface->GetIDEx(KramdownDocument),"s","# aaaaaa");
    char *html = (char *)SRPInterface->ScriptCall(resobj,NULL, "method_missing", "(s)s","to_html");
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

*   download from here

    [ios_call_ruby_kramdown.zip](/datas/ios_call_ruby_kramdown.zip  "ios_call_ruby_kramdown")

<h1 align = "left"><font color="#FF9900">History</font></h1>


