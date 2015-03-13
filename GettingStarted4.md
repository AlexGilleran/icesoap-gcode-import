# Part 4: Building Requests #

By now we have all the building blocks for our SOAP call - we have the envelope to send and we have the objects that we want to get back. So now we've just got to put it together.

In IceSoap this is done via the use of the `Request` interface, which takes in a URL, an Envelope, a SOAP action (as a String) and the class to parse (and potentially the soapfault to parse - more on that later), and handles all the networking and parsing - you just have to instantiate it.

**Note:** For the purposes of simplicity, this tutorial uses the `SOAP11Request` interface which is an extension of `Request` that defaults what is returned in the event of a SOAP Fault and is fine for most applications. For customising the handling of SOAPFaults, refer to the wiki article on [SOAP Fault Customisation](http://code.google.com/p/icesoap/wiki/SOAPFaultCustomisation).

## The RequestFactory ##

Requests are obtained from the `RequestFactory`, the default implementation of which is `RequestFactoryImpl`. The easiest way to get this factory is simply:

```
private RequestFactory requestFactory = new RequestFactoryImpl();
```

I've included this as an interface and given you control of how to implement it to make it easier for you to mock out if you wish to.

## Request Types ##
Once you've got your factory, you'll notice there's two important methods - `buildRequest` and `buildListRequest`. There are two kinds of requests within IceSoap - `Request`s and `ListRequest`s.

### Request ###
A `Request` is used to retrieve a single object from a service.
When executed, it'll send the supplied envelope to the supplied envelope with the supplied SOAPAction, then look at the SOAP Envelope retrieved looking for an XML node matching the XPath supplied in the `@SOAPObject` annotation of the supplied class. Once it's found this node, it'll populate a new instance of that class with information in the XMLNode. When the XMLNode ends, it stops parsing. A standard request should be used for most calls.

In the example, I need a `Request` to get back a single definition for a supplied word. I can do so like this:
```
SOAP11Request<Definition> definitionRequest = requestFactory.buildRequest(
    "http://services.aonaware.com/DictService/DictService.asmx",
    new DefineWordEnvelope(dictionaryId, word), 
    "http://services.aonaware.com/webservices/DefineInDict",
    Definition.class);
```
(obviously you should use constants for a lot of these values)

### ListRequest ###
A `ListRequest` is similar to a normal `Request` in most ways, except that rather than looking for a single XML node matching the `@SOAPObject` annotation in the supplied class and then ending at the end of that node, it will go through the entire return envelope looking for _any_ object matching the XPath in the annotation. It will return these objects as a list when it reaches the end of the return SOAP envelope.

Use these when you're dealing with a response like this:

```
<List>
    <ListItem>
        <!-- Item Data -->
    </ListItem>
    <ListItem>
        <!-- Item Data -->
    </ListItem>
    <ListItem>
        <!-- Item Data -->
    </ListItem>
</List>
```

Note that you can also parse lists with a standard `Request` by creating an object with a List field in it and annotating it with @XMLField - see GettingStarted3.

In the example, I need to use a `ListRequest` to get a list of Dictionaries back from the service. I create one like this:
```
SOAP11ListRequest<Dictionary> dictionaryRequest = requestFactory.buildListRequest(
    "http://services.aonaware.com/DictService/DictService.asmx",
    new GetDictionariesEnvelope(),
    "http://services.aonaware.com/webservices/DictionaryList",
    Dictionary.class);
```
(obviously you should use constants for a lot of these values)

**Up Next:** GettingStarted5, in which we use our brand new Requests and perform some SOAP calls!