---
layout: post
title:  "Delphi Call Markdown With Ruby"
date:   2017-12-25 16:49:07 +0800
categories: delphi
---
{% include codeformat.html %}

<h1 align = "left"><font color="#FF9900">Introduction</font></h1>

Delphi is widely used programming language and kramdown is used by Jekyll to parse the markdown file. kramdown can convert markdown file to html which is written in pure ruby.

The example shows how to use kramdown to convert markdown file to html in apps developed with delphi.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example is based on CLE. CLE supports windows platform, ruby language, and interaction between ruby and delphi.

The sample is developed with delphi xe2, ruby 2.2(Installed with RubyInstaller), and kramdown-1.16.2.

### 1. Install CLE

*   Download and install CLE for win32

    [starcore_for_windows](https://github.com/srplab/starcore_for_windows  "starcore_for_windows")
    
    ![](/images/install_starcore_win32_2_5_2.png){:width="320px"}

### 2. Create delphi project
    
*   create a vcl form project
    
    ![](/images/create_delphi_vcl_project.png){:width="320px"}

### 3. Add starcore.pas to the project

*   Add starcore.pas to the project

    "starcore.pas" is open source file written in pascal, which encapsulates the interface functions of CLE.

    ![](/images/add_starcore_pas_to_project.png){:width="320px"}
    
    The code 
    
    {% highlight pascal %}
uses
  Winapi.Windows, Winapi.Messages, System.SysUtils, System.Variants, System.Classes, Vcl.Graphics,
  Vcl.Controls, Vcl.Forms, Vcl.Dialogs, Vcl.StdCtrls,starcore;
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}
    
### 4. Use variant access methods and properties of objects

*   variant    

    For easy access to the object's methods and properties. The CLE object is encapsulated as a variant variable.
    In this way, you can handle the cle object to the same access delphi object. For example,
    {% highlight pascal %}

var
  varruby : variant;
  LOAD_PATH : variant;
  ...
  
begin
  varruby := SRPOBJECT_TOVARIANT(ruby,true);  
  LOAD_PATH := varruby.LOAD_PATH;
  LOAD_PATH.unshift('C:\Ruby22\lib\ruby\gems\2.2.0\gems\kramdown-1.16.2\lib');  
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}    
    
*   Please refer to the programmer's guide for more details.

<h1 align = "left"><font color="#FF9900">Using the code</font></h1>
    

### 1. Initialize CLE

*   Callback function

    {% highlight pascal %}
function MsgCallBack( ServiceGroupID:VS_ULONG; uMsg:VS_ULONG; wParam:VS_UWORD; lParam:VS_UWORD; IsProcessed:PVS_BOOL; Para:VS_UWORD ) : VS_UWORD; cdecl;
var
   str : string;
begin
   if ( uMsg = MSG_VSDISPLUAMSG) or (uMsg = MSG_VSDISPMSG ) or (uMsg = MSG_DISPMSG ) or (uMsg = MSG_DISPLUAMSG ) then
   begin
        str := TOVS_STRING(PVS_CHAR(wParam));
        if not (Length(str) = 0 ) then
        begin
            Form1.Memo1.Lines.Add(str);
        end;
   end;
   Result := 0;
end;
    {% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}  
    
*   Use the following code to initialize CLE

    {% highlight pascal %}    
var
  SRPControlInterface : Pointer;
  BasicSRPInterface : Pointer;
  ServiceID : VS_UUID;
  CLEInterface : Pointer;
  
begin
  
  StarCore_Init('libstarcore.dll');
  VSCore_RegisterCallBackInfo(@MsgCallBack,0);
  VSCore_Init( TRUE, TRUE, '', 0, '', 0,nil);
	SRPControlInterface := VSCore_QueryControlInterface();
	BasicSRPInterface := SRPControl_QueryBasicInterface(SRPControlInterface,0);
  INIT_UUID( ServiceID );
  if( SRPBasic_CreateService( BasicSRPInterface, '','test',@ServiceID,'123',0,0,0,0,0,0 ) = false ) then
     exit;
  CLEInterface := SRPBasic_GetSRPInterface(BasicSRPInterface,'test','root','123');
  (*--must set SRP_CLEInterface after getting the service interface--*)
  SRP_CLEInterface := CLEInterface;  
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}      

### 2. Initialize ruby and set ruby libraries searching path

*  kramdown-1.16.2 library path must be set to ruby before load.

{% highlight pascal %}
  SRPControl_SetScriptInterface(SRPControlInterface,'ruby','', '-v 2.2.0');
  SRPBasic_InitRaw(BasicSRPInterface,'ruby',CLEInterface);
  ruby := SRPI_ImportRawContext(CLEInterface,'ruby','',false,NULL);
  varruby := SRPOBJECT_TOVARIANT(ruby,true);

  LOAD_PATH := varruby.LOAD_PATH;
  LOAD_PATH.unshift('C:\Ruby22\lib\ruby\gems\2.2.0\gems\kramdown-1.16.2\lib');
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"} 
     
### 3. Call kramdown

{% highlight pascal %}
  varruby.require('kramdown');
  KramdownDocument := varruby.eval('Kramdown::Document');
  Memo1.Lines.Add(KramdownDocument);
  resobj := KramdownDocument.create('# aaaaaa');

  Result := resobj.method_missing('to_html');
  Memo1.Lines.Add(Result);   
{% endhighlight %}{: style="color:gray;margin:0 0 0 2em"}

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

*   download from here

    [delphi_ruby_call_kramdown.zip](/datas/delphi_ruby_call_kramdown.zip  "delphi_ruby_call_kramdown")

<h1 align = "left"><font color="#FF9900">History</font></h1>


