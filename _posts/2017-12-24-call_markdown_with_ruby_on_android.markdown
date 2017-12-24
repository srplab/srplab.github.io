---
layout: post
title:  "Call Kramdown with Ruby on Android"
date:   2017-12-21 15:31:07 +0800
categories: android
---

### This is the first blog. 

<h1 align = "left"><font color="#FF9900">Introduction</font></h1>

Normally, jekyll uses kramdown to parse the markdown file. kramdown can convert markdown file to html which is written in pure ruby.

The example shows how to use kramdown to convert markdown file to html.

<h1 align = "left"><font color="#FF9900">Background</font></h1>

This example is based on CLE. CLE supports android platform, ruby language, and interaction between ruby and java.

### 1. Create project and Add the dependent files

* add starcore_android_rX.XX.jar and share libraries

![](/images/2017-12-24-call_markdown_with_ruby_on_android_pic1.png){:width="320px"}

* pack ruby libraries to zip file

In this sample, ruby 2.2 is used.

![](/images/2017-12-24-call_markdown_with_ruby_on_android_pic2.png){:width="320px"}

* pack kramdown-1.16.2 to zip file

![](/images/2017-12-24-call_markdown_with_ruby_on_android_pic3.png){:width="320px"}

* add files to project folder assets

![](/images/2017-12-24-call_markdown_with_ruby_on_android_pic4.png){:width="160px"}

### 2. Public function, Extract the zip file to the application's installation directory

* copyFile

{% highlight java %}
    private void copyFile(Activity c, String Name,String desPath) throws IOException {
        File outfile = null;
        if( desPath != null )
            outfile = new File("/data/data/"+getPackageName()+"/files/"+desPath+Name);
        else
            outfile = new File("/data/data/"+getPackageName()+"/files/"+Name);
        //if (!outfile.exists()) {
        {
            outfile.createNewFile();
            FileOutputStream out = new FileOutputStream(outfile);
            byte[] buffer = new byte[1024];
            InputStream in;
            int readLen = 0;
            if( desPath != null )
                in = c.getAssets().open(desPath+Name);
            else
                in = c.getAssets().open(Name);
            while((readLen = in.read(buffer)) != -1){
                out.write(buffer, 0, readLen);
            }
            out.flush();
            in.close();
            out.close();
        }
    }
{% endhighlight %}{: style="color:gray;"}

* unzip

{% highlight java %}
private static boolean CreatePath(String Path){
        File destCardDir = new File(Path);
        if(!destCardDir.exists()){
            int Index = Path.lastIndexOf(File.separator.charAt(0));
            if( Index < 0 ){
                if( destCardDir.mkdirs() == false )
                    return false;
            }else{
                String ParentPath = Path.substring(0, Index);
                if( CreatePath(ParentPath) == false )
                    return false;
                if( destCardDir.mkdirs() == false )
                    return false;
            }
        }
        return true;
    }

    private static boolean unzip(InputStream zipFileName, String outputDirectory,Boolean OverWriteFlag ) {
        try {
            ZipInputStream in = new ZipInputStream(zipFileName);
            ZipEntry entry = in.getNextEntry();
            byte[] buffer = new byte[1024];
            while (entry != null) {
                File file = new File(outputDirectory);
                file.mkdir();
                if (entry.isDirectory()) {
                    String name = entry.getName();
                    name = name.substring(0, name.length() - 1);
                    if( CreatePath(outputDirectory + File.separator + name) == false )
                        return false;
                } else {
                    String name = outputDirectory + File.separator + entry.getName();
                    int Index = name.lastIndexOf(File.separator.charAt(0));
                    if( Index < 0 ){
                        file = new File(outputDirectory + File.separator + entry.getName());
                    }else{
                        String ParentPath = name.substring(0, Index);
                        if( CreatePath(ParentPath) == false )
                            return false;
                        file = new File(outputDirectory + File.separator + entry.getName());
                    }
                    if( !file.exists() || OverWriteFlag == true){
                        file.createNewFile();
                        FileOutputStream out = new FileOutputStream(file);
                        int readLen = 0;
                        while((readLen = in.read(buffer)) != -1){
                            out.write(buffer, 0, readLen);
                        }
                        out.close();
                    }
                }
                entry = in.getNextEntry();
            }
            in.close();
            return true;
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            return false;
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        }
    }
{% endhighlight %}{: style="color:gray;"}

<h1 align = "left"><font color="#FF9900">Using the code</font></h1>

### 1. Unzip the file to app folder

{% highlight java %}
        File destDir = new File("/data/data/"+getPackageName()+"/files");
        if(!destDir.exists())
            destDir.mkdirs();
        final String nativeLibraryDir = this.getApplicationInfo().nativeLibraryDir;
        try{
            AssetManager assetManager = getAssets();
            InputStream dataSource = null;
            dataSource = assetManager.open("ruby-2.2.zip");
            unzip(dataSource, "/data/data/"+getPackageName()+"/files",false );
            dataSource.close();

            dataSource = assetManager.open("kramdown-1.16.2.zip");
            unzip(dataSource, "/data/data/"+getPackageName()+"/files",false );
            dataSource.close();
        }
        catch(Exception e)
        {

        }
{% endhighlight %}{: style="color:gray;"}

### 2. Initilize CLE

{% highlight java %}
        /*----init starcore----*/
        StarCoreFactoryPath.StarCoreCoreLibraryPath = this.getApplicationInfo().nativeLibraryDir;
        StarCoreFactoryPath.StarCoreShareLibraryPath = this.getApplicationInfo().nativeLibraryDir;
        StarCoreFactoryPath.StarCoreOperationPath = "/data/data/"+getPackageName()+"/files";

        final StarCoreFactory starcore= StarCoreFactory.GetFactory();
        StarServiceClass Service=starcore._InitSimple("test","123",0,0);
        SrvGroup = (StarSrvGroupClass)Service._Get("_ServiceGroup");
        Service._CheckPassword(false);

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
{% endhighlight %}{: style="color:gray;"}   

### 2. Initilize ruby

{% highlight java %}
        try{
            System.load(this.getApplicationInfo().nativeLibraryDir+"/libruby.so");
        }
        catch(UnsatisfiedLinkError ex)
        {
            System.out.println(ex.toString());
        }
        SrvGroup._InitRaw("ruby",Service);
        StarObjectClass ruby = Service._ImportRawContext("ruby","",false,"");
        System.out.println(ruby._Get("LOAD_PATH"));
        System.out.println(ruby._Get("File"));
{% endhighlight %}{: style="color:gray;"} 
     
### 3. Set ruby libraries searching path

{% highlight java %}
        StarObjectClass LOAD_PATH = (StarObjectClass)ruby._R("LOAD_PATH");

        LOAD_PATH._Call("unshift", "/data/data/"+getPackageName()+"/files/lib/ruby/2.2.0");
        LOAD_PATH._Call("unshift", "/data/data/"+getPackageName()+"/files/armeabi-v7a/lib-dynload");
        LOAD_PATH._Call("unshift", "/data/data/"+getPackageName()+"/files");
        LOAD_PATH._Call("unshift", "/data/data/"+getPackageName()+"/files/kramdown-1.16.2/lib");
{% endhighlight %}{: style="color:gray;"} 

### 4. Call kramdown

{% highlight java %}
        ruby._Call("require","enc/encdb");  //--else, will encoding error
        ruby._Call("require","kramdown");
        StarObjectClass KramdownDocument = (StarObjectClass)ruby._Call("eval","Kramdown::Document");
        StarObjectClass resobj = KramdownDocument._New("","","# aaaaaa");
        Object res = resobj._Call("method_missing","to_html");     
{% endhighlight %}{: style="color:gray;"}

<h1 align = "left"><font color="#FF9900">Sample Download</font></h1>

[call_markdown_with_ruby_on_android.zip](/datas/call_markdown_with_ruby_on_android.zip  "call_markdown_with_ruby_on_android")

<h1 align = "left"><font color="#FF9900">History</font></h1>


