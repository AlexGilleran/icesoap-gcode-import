# Using Custom Ports and SSL #

IceSoap uses an implementation of the `SOAPRequester` to perform requests. By default, it will use the reference `ApacheSOAPRequester`, which uses the Apache HttpClient built into Android to do requests. This implementation uses HTTP as a transport, and uses port 80 (or port 443 for SSL) with the associated PlainSocketFactory or SSLSocketFactory depending on whether the call is SSL or not.

This works for most cases, but there some in which it won't:
  * Your service is on a non-standard port
  * Your service uses SSL and a self-signed certificate
  * You want to use a method other than the default HttpClient to do HTTP calls (e.g. to use `UrlConnection`, do fancier caching etc.)
  * Your service is a SOAP 1.2 service that uses a transport other than HTTP

In the first two cases, you can extend the built-in `ApacheSOAPRequester` (keep reading) - for the rest, you'll need to implement your own `SOAPRequester` (skip to the end).

## Extending ApacheSOAPRequester ##
Create a new class that extends `ApacheSOAPRequester` - you'll need to override the getSchemeRegistry() method.

Normally, this looks like so:

```
protected SchemeRegistry getSchemeRegistry() {
    SchemeRegistry schemeRegistry = new SchemeRegistry();

    schemeRegistry.register(new Scheme(HTTP_NAME, PlainSocketFactory
        .getSocketFactory(), DEFAULT_HTTP_PORT));
    schemeRegistry.register(new Scheme(HTTPS_NAME, SSLSocketFactory
        .getSocketFactory(), DEFAULT_HTTPS_PORT));
    return schemeRegistry;
}
```

Where `DEFAULT_HTTP_PORT` is 80 and `DEFAULT_HTTPS_PORT` is 443. You'll want to replace this with your own socket and factory combinations. If all you're worried about is changing ports, you can simply copy the code above substituting your port(s) for the default ones.

e.g.
```
protected SchemeRegistry getSchemeRegistry() {
    SchemeRegistry schemeRegistry = new SchemeRegistry();

    schemeRegistry.register(new Scheme(HTTP_NAME, PlainSocketFactory
        .getSocketFactory(), 666));
    
    return schemeRegistry;
}
```

## Using Your Own Socket Factories (Self-Signed Certificates) ##
In addition to changing around the ports, you can also override `getSchemeRegistry()` to specify your own `SocketFactory` for certain ports, like so:

```
protected SchemeRegistry getSchemeRegistry() {
    SchemeRegistry schemeRegistry = new SchemeRegistry();

    schemeRegistry.register(new Scheme(HTTP_NAME, new MySocketFactory(), 
        DEFAULT_HTTP_PORT));
    
    return schemeRegistry;
}
```

This is mostly useful for using self-signed certificates for SSL - by default, if you try to connect via SSL to a SOAP service that doesn't have an SSL Certificate signed by a trusted provider (e.g. Verisign), it'll throw exceptions.

Creating a `SocketFactory` to allow connections to self-signed certificate is dangerous and not something I'm keen to document with IceSoap. The best quick-and-dirty guide to making a certificate factory that just accepts everything (_for development not production!_) is here:

http://stackoverflow.com/questions/1217141/self-signed-ssl-acceptance-android

## Using your new SOAPRequester Implementation ##
Now you've got your new `SOAPRequester`, either by extending `ApacheSOAPRequester` or by creating your own from scratch, you're going to want to use it for your requests.

This is done at the `RequestFactory` level - either pass the requester to the constructor `RequestFactoryImpl` (assuming that that's the implementation of `RequestFactory` that you're using):

```
private RequestFactory requestFactory = new RequestFactoryImpl(new MyCustomSOAPRequester());
```

or use the `setSOAPRequester(SOAPRequester soapRequester)` method which is present on all `SOAPRequester` implementations.

```
requestFactory.setSOAPRequester(new MyCustomSOAPRequester());
```