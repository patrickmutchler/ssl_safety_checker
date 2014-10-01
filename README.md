# Android WebView SSL Safety Checker

### What's it do?
This tool analyzes each of the apps on your phone for bad behavior when 
handling SSL errors in WebView. Our research has shown that a whopping 18% of 
WebView apps do at least *something* weird with SSL errors, leaving their 
users vulnerable to attack by malicious networks without any indication that 
something is going wrong. This tool can help you figure out if your apps are 
using SSL safely or not. For more details about SSL errors in WebView and how 
things can go wrong check out 
[this blog post](http://stanford.edu/~pcm2d/blog/ssl.html).

### Usage
Run `python run.py -h` to see usage details

### Requirements
You must have Python and Java installed on your machine and you must be able 
to run java from the command line. I've only tested this on Java1.6 but it
should work fine on other versions as well. All other required libraries are
included in this repository.

### An example
An app (located in /test/test.apk) contains two implementations of 
`onReceivedSslError`, one in library code and one in the main app package.
Running "python run.py test/test.apk" produces the following output:

    AppName: Test App
        Main Package: com.test
        Target API Level: 16
        onReceivedSslError implementations: 2
            com.test.MyClient: always ignore errors
            com.testlib.LibClient: sometimes ignore errors
        addJavascriptInterface found in: 
            com.test.MainActivity

This means that there were two implementations of `onReceivedSslError` found 
in the app, one in `com.test.MyClient` and one in `com.testlib.LibClient`.
Note that the first implementation is in the app's main package and the other
is in library code. The first implementation *must* ignore certificate errors
while the second only ignores certificate errors on some code paths. The first
implementation is definitely a security violation and the second one might be
depending on how it is implemented. You can look at either implementation by
decoding the apk using apktool and then looking at the smali code (readable
Dalvik Bytecode). Decode an apk with apktool by running:

    lib/apktool/apktool d PATH_TO_APK_FILE

The tool also found uses of the JavaScript Bridge in `com.test.MainActivity`.
Because this is in the same package as an offending implementatation of 
`onReceivedSslError` it is very likely that this is happening on the same
WebView instance. Finally, because the Target API Level is less than 17 the
JavaScript Bridge can be used to execute arbitary Java code, upping the 
severity of an attack. 

### How it works
This tool uses [Soot](http://www.sable.mcgill.ca/soot/), a Java analysis and 
instrumentation framework with support for Dalvik Bytecode, to analyze
implementations of `onReceivedSslError` in apps and libraries. Most apps have
very simple implementations of this method so it can trivially determine the
behavior statically. The tool also looks for behavior using the JavaScript 
Bridge, which can increase the severity of a problem in `onReceivedSslError`. 

### Future work
I'd like to do some symbolic computation of the complex implementations of 
`onReceivedSslError` so we can get a better sense for the conditions when they
will ignore or respect certificate errors. I'd also like to verify that the
JavaScript Bridge is actually being used with the same WebView instance that
has the offending behavior. Its possible to cheat here and get somewhat precise
results but the best way to do this is to just include a full points-to
analysis in this tool, which can end up being massive overkill for many apps.

