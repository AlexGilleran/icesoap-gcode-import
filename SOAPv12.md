If you've followed the Getting Started guide, then you'll have set up a client that communicates with a SOAP v1.1 service - for most users this is okay, because most web-based SOAP services use or at least support SOAP 1.1 - SOAP 1.2 tends to get more use inside enterprises where it's used with JMS and so on.

However, IceSoap does support SOAP v1.2, it just takes a few tweaks here and there. The key differences between SOAP 1.2 when using IceSoap are that you have to extend a different `Envelope` class (`BaseSOAP12Envelope`) and that rather than use the default SOAP 1.1 request and observer classes, use the generic ones and specify `SOAP12Fault` as your fault type.

## Extending `BaseSOAP12Envelope` instead of `BaseSOAP11Envelope` ##
This is quite straightforward - in GettingStarted2 you saw how to create an envelope by extending `BaseSOAP11Envelope`. For SOAP 1.2, just extend `BaseSOAP12Envelope` instead, like so:

```
public class GetDictionariesEnvelope extends BaseSOAP12Envelope {

}
```

This will change the namespaces in the output envelope so that it uses SOAP 1.2 instead of SOAP 1.1.

## Using Generic `Request`s instead of `SOAP11Request` ##
Once again, this is quite straightforward. In GettingStarted4 you would've created a request like so:

```
SOAP11Request<Definition> definitionRequest = requestFactory.buildRequest(
    "http://services.aonaware.com/DictService/DictService.asmx",
    new DefineWordEnvelope(dictionaryId, word), 
    "http://services.aonaware.com/webservices/DefineInDict",
    Definition.class);
```

However, if you look in the code you'll see that `SOAP11Request<ResultType>` is just an empty extension of `Request<ResultType, SOAP11Fault`.

```
public interface SOAP11Request<ResultType> extends
                Request<ResultType, SOAP11Fault> {

}
```

So all we need to do to make it work with SOAP 1.2 program to the generic `Request` interface instead of `SOAP11Request`, while specifying the `SOAP12Fault` type in both the generic declaration of the `Request`, and as the last parameter passed to the factory:

```
Request<Definition, SOAP12Fault> definitionRequest = requestFactory.buildRequest(
    "http://services.aonaware.com/DictService/DictService.asmx",
    new DefineWordEnvelope(dictionaryId, word), 
    "http://services.aonaware.com/webservices/DefineInDict",
    Definition.class, SOAP12Fault.class);
```

This process is the same for `SOAP11ListRequest`.

## Use the Generic `SOAPObserver` ##
Once again, in GettingStarted5 you would've seen the observer set up like this:

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

The `SOAP11Observer` interface is very similar to `SOAP11Request` - it's just a blank extension of the more generic interface that hard-codes `SOAP11Fault`. To make this work with SOAP 1.2, just change it to use the generic interface/implementation and specify `SOAP12Fault` as the fault type:

```
private SOAPObserver<T, SOAP12Fault> AnonymousObserver = new SOAPObserver<T, SOAP12Fault>() {
    @Override
    public void onCompletion(Request<T, SOAP12Fault> request) {

    }

    @Override
    public void onException(Request<T, SOAP12Fault> request, SOAPException e) {

    }
};
```

Once again, this is true for list observers as well.

You may want to use a custom SOAP Fault type instead of the built-in `SOAP12Fault` - in this case, follow the guide in SOAPFaultCustomisation but extend `SOAP12Fault` instead of `SOAP11Fault` as the formats differ between the versions.