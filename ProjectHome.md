IceSoap provides quick, easy, asynchronous access to SOAP web services from Android devices. It allows for SOAP responses to be bound to Java POJOs via annotations (using a subset of xpath), while still retaining the speed of the native android XmlPullParser.

This came about from a project intending to provide a front-end for Websphere Commerce Server via its (rather complex) web service layer. I found KSoap to be too difficult to work with, and doing each SOAP call manually with its own parser was a nightmare of nested-ifs to try to represent XML hierarchies with one-dimensional XmlPullParser code. I was also frustrated at how easy it would all be if I could just connect to a JSON-based service instead. There had to be a better way...

And so, IceSoap was born. IceSoap aims to get you up and connected to your web service quickly and easily, without messing about with WSDLs or code generation. I've noticed that while a lot of JSON-and-REST-based web services provide lovely, simple APIs, a lot of SOAP services are often complex, mid-00s behemoths designed with full SOA in mind (like the Websphere Commerce API) rather than providing a bit of information to a phone. As such, IceSoap is designed in such a way that the complexity of the service you're connected to won't leak into your lean, mean mobile app - you can use just as much of the web service as you need and forget about the rest.

IceSoap doesn't generate code to implement SOAP - rather its design is closer to that of JSON libraries like GSON - you simply create a Java POJO to represent the object you want to get from the web service, annotate it with XPaths so it can find the fields it needs, then let IceSoap parse it for you. Use of background threads is baked into the code, so all you have to worry about is what to send and what to do with it when it comes back.

So [download IceSoap](http://code.google.com/p/icesoap/wiki/Installation), read the [Getting Started Guide](http://code.google.com/p/icesoap/wiki/GettingStarted_Contents), have a look at the [example](http://code.google.com/p/icesoap/source/browse/IceSoapExample#IceSoapExample%2Fsrc%2Fmain%2Fjava%2Fcom%2Falexgilleran%2Ficesoap%2Fexample) and get going! Javadoc is [here](http://icesoap.googlecode.com/git-history/icesoap-1.0.7/IceSoap/javadoc/index.html).

If you prefer, you can also find/fork the code on [Github](https://github.com/AlexGilleran/IceSoap).

The latest version is **1.0.7**. This is a little maintenance release that addresses some concerns that have come up since 1.0.6.
  * You can now use the Object versions of primitives (e.g. `Integer` instead of `int`) in `@XMLObject` classes ([Issue 22](https://code.google.com/p/icesoap/issues/detail?id=22)).
  * The content type for SOAP 1.2 has been corrected to be `application/soap+xml` ([Issue 23](https://code.google.com/p/icesoap/issues/detail?id=23))
  * You can now execute `SOAPRequests` on the same thread by calling executeBlocking. Obviously this is a bad idea if you're on the UI thread, but it's pretty useful if you're using a background service or some equivalent background thread to do SOAP requests.

This project is now in maintenance mode - the code is stable and I'll fix happily bugs if they come up, but I won't be developing new features. That said, if you want to develop it, feel free to send me a pull request on Github or just fork the project altogether. I've seen downloads dwindle as time has gone on, which hopefully means that SOAP is declining as a messaging format, and hence there's not much point enhancing this library much further.

```
Copyright 2012-2013 Alex Gilleran

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```