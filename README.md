# Android WebView SSL Safety Checker

### What's it do?
This tool analyzes each of the apps on your phone for bad behavior when handling SSL errors in WebView. Our research has shown that a whopping 18% of WebView apps do at least *something* weird with SSL errors, leaving their users vulnerable to attack by malicious networks without any indication that something is going wrong. This tool can help you figure out if your apps are using SSL safely or not. For more details about SSL errors in WebView and how things can go wrong check out [this blog post](stanford.edu/~pcm2d/blog/ssl.html).

### Usage
TODO

### An example
TODO

### More details
TODO

### How it works
This tool uses [Soot](http://www.sable.mcgill.ca/soot/), a Java analysis and instrumentation framework with support for Dalvik Bytecode, to analyze each of the apks on your phone for implementations of `onReceivedSslError`. Lots of apps have very simple implementations of this method so it can trivially determine their behavior statically. For more complicated implementations it computes a backwards slice of the method implementation, extracts it from the apk, and runs it with a bunch of sample inputs to get a good approximation of the true behavior. 

The backwards slicing approach definitely isn't as good as just figuring out the actual path conditions through the method implementation using a SAT solver but for now it does a good job. I might update this in the future to use a SAT solver as a good exercise but no promises.

Note that this tool computes a local backwards slice rather than a global one to keep the running time down. This hurts precision a bit but in practice things don't seem to be so bad. 