# Part 5: Using Requests In Your App #

If you've followed the instructions from GettingStarted4 then you have your requests set up and instantiated. But before you let them loose you'll have to use an observer to them to get the result back.

## SOAPObserver ##
All calls made by IceSoap are asynchronous - this means that you can't just call a function to get the result of a SOAP call, as that would lock the thread. Instead you must use an implementation of `SOAPObserver` to catch the response of the service when it happens.

The easiest way to implement `SOAPObserver` is to just create an anonymous implementation in another class (usually an Activity or Fragment, but it's not important). Because we're not worried about custom soap faults, we'll use the simpler `SOAP11Observer`, which is an extension of `SOAPObserver` that defaults the SOAP Fault Type to `SOAP11Fault`.

```
private SOAP11Observer<T> AnonymousObserver = new SOAP11Observer<T>() {
    @Override
    public void onCompletion(Request<T, SOAP11Fault> request) {

    }

    @Override
    public void onException(Request<T, SOAP11Fault> request, SOAPException e) {

    }
};
```
(where `T` is the type you want to get back from the request. `SOAP11Fault` is the default SOAPFault class for `SOAP11Request`s)

You'll notice there are two methods that you have to implement:

### onCompletion(Request`<`T, SOAP11Fault`>` request) ###
This is called on the completion of a request, and passes the request that's completed through - you can get the object that's been parsed by invoking `request.getResult()`. This is where you put whatever you want to do with the data you've retrieved from the service - this will usually be something like refresh a `TextView`.

### onException(Request`<`T, SOAP11Fault`>` request, SOAPException e) ###
This will occur if there's an exception during the execution of the request (this will be the cause of the SOAPException that's returned). This could be network issues, a server error or a parsing issue. This has to be passed as a parameter because the conventional Java method of throwing exceptions doesn't gel with the asynchronous nature of IceSoap. Therefore, think of this method as a `catch` block for your request - what do you want to do if something goes wrong? E.g. log the error, display a message to the user etc.

In the event of the server returning an HTTP 500 error code, information will be returned in XML format and parsed by IceSoap - you can access this by calling `getSOAPFault()` on the request. If you choose to use the `SOAP11Request` interface, this will default to the basic `SOAP11Fault` type - if you need more information from SOAPFaults, refer to [the wiki](http://code.google.com/p/icesoap/wiki/SOAPFaultCustomisation).

For an example of a `SOAPObserver`, look at [DefineActivity](http://code.google.com/p/icesoap/source/browse/IceSoapExample/src/main/java/com/alexgilleran/icesoap/example/activities/DefineActivity.java) in the example app.

## SOAPListObserver ##
The `SOAPListObserver` is an extension of the `SOAPObserver` interface that handles `ListRequest`s. This has an extra method:

### onNewItem(Request`<`List`<`T`>`, SOAP11Fault`>` request, T item) ###
This is called every time the `ListRequest` completely parses an item in the list. The most common use case for this is having a ListView that loads in new items as they're downloaded and parsed, rather than waiting until the whole list is parsed before displaying anything - what you do with it, however, is limited only by your creativity. The item that's been parsed will be passed to this method as the last parameter.

For an example of a `ListSOAPObserver` see [DictionaryListActivity](http://code.google.com/p/icesoap/source/browse/IceSoapExample/src/main/java/com/alexgilleran/icesoap/example/activities/DictionaryListActivity.java) in the example app.

## Executing the Request ##
Now that you've got an observer, it's time to register it with the request (_before_ execution - that'll happen in a second). You can do this via the `registerObserver` method like so:

```
request.registerObserver(observer)
```

This registers the observer with the request - when the request completes, hits an exception or parses a new item, the appropriate method will be called on all observers registered to it - this means that you want to use multiple observers to do various things, you can do so. The `registerObserver` is overloaded with methods to handle both standard `SOAPObserver`s and `SOAPListObservers`, so the syntax is the same for either.

Once you've registered your observer, execute the request like so:

```
request.execute();
```

Note that `execute` is a `void` - it doesn't return anything, as the returning of data will be done via the observer you just set up.

Alternatively if you only have one observer and don't want to register any more, you can use the simple 1-line `execute` overload:

```
request.execute(observer);
```

This simply registers the passed observer then executes the request.

## And we're done! ##

Once you've got all of that setup, you'll have completed your first request with IceSoap! Have a pat on the back!

In the likely event you need a bit more help, you can have a look at the sample app [here](http://code.google.com/p/icesoap/source/browse/IceSoapExample#IceSoapExample%2Fsrc%2Fmain%2Fjava%2Fcom%2Falexgilleran%2Ficesoap%2Fexample). The Javadoc explaining all the individual method calls (and more) is [here](http://icesoap.googlecode.com/git-history/icesoap-1.0.2/IceSoap/javadoc/index.html).

If you want some more clarification on what all the `SOAP11Fault` nonsense was all about, check out the article on customising SOAP Fault handling [here](http://code.google.com/p/icesoap/wiki/SOAPFaultCustomisation).

Best of luck!