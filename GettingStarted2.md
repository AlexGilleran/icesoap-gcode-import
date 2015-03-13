# Part 2: Creating Envelopes #

## SOAP 1.1 vs SOAP 1.2 ##

New in IceSoap v1.0.2 is better support for SOAP 1.2. If you're not familiar with different SOAP versions, there are two main ones in use today - 1.1 and 1.2, and whether I want to use one or the other is going to affect how I build up my envelope.

If you've read GettingStarted1 then you know that I've seen my request, and it has to look something like this:

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:web="http://services.aonaware.com/webservices/">
   <soapenv:Header/>
   <soapenv:Body>
      <web:DictionaryList/>
   </soapenv:Body>
</soapenv:Envelope>
```

If you look carefully you'll see that the SOAP namespace it uses is `http://schemas.xmlsoap.org/soap/envelope` - this indicates it's a Soap 1.1 service. If the namespace was `"http://www.w3.org/2003/05/soap-envelope` that would mean SOAP 1.2. In my experience, most SOAP services that you'd want to consume with a mobile phone are SOAP 1.1, and yours probably will be too - if it isn't, keep following this tutorial, and also consult the [SOAPv12 wiki article](http://code.google.com/p/icesoap/wiki/SOAPv12) to see what you'll have to do differently.

## Building the Envelope ##
At this point, envelope creation in IceSoap is done programmatically (i.e. there's no generator to help you). This means that you get a bunch more control over what you do, but you have to work a bit harder.

To create envelopes you have a choice - you can either create classes that extend BaseSOAP11Envelope to represent envelopes, or just instantiate a new BaseSOAP11Envelope for each new envelope and use the public methods on its interface to build it up. If you've got a bunch of different envelopes that are reasonably complex and have a lot in common, you'll probably want to create new classes and have them all inherit from one base class. It's up to you.

Note that we use BaseSOAP11Envelope here because we're targeting a SOAP 1.1 service - if it were SOAP 1.2, we'd use the BaseSOAP12Envelope.

In any case, you're going to want to build something up that resembles the XML of your request. I'm going to extend BaseSOAP11Envelope in this tutorial - if you want to do it the other way then it's not too different.

Having created a new "<your namespace>.envelopes" package, I create a new class and make it extend BaseSOAPEnvelope. As BaseSOAP11Envelope doesn't actually have any abstract methods, I end up with this:

```
public class GetDictionariesEnvelope extends BaseSOAP11Envelope {

}
```

That's not much fun, so I create a constructor and get started:

```
public GetDictionariesEnvelope() {    
    declarePrefix("web", AON_AWARE_NAMESPACE);
    getBody().addNode(super.getAonAwareNamespace(), "DictionaryList");
}
```

... and that's it. If you instantiate that class and call .toString() on it, it'll look like the XML above.

I'll also need an envelope to a dictionary definition, using an envelope like this:

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:web="http://services.aonaware.com/webservices/">
   <soapenv:Header/>
   <soapenv:Body>
      <web:DefineInDict>
         <web:dictId></web:dictId>
         <web:word></web:word>
      </web:DefineInDict>
   </soapenv:Body>
</soapenv:Envelope>
```

And so I'll create this class:

```
public class DefineWordEnvelope extends BaseSOAP11Envelope {
    private final static String AON_AWARE_NAMESPACE = "http://services.aonaware.com/webservices/";

    public DefineWordEnvelope(String dictId, String word) {
        declarePrefix("web", AON_AWARE_NAMESPACE);

        XMLParentNode defineInDict = getBody().addNode(
        AON_AWARE_NAMESPACE, "DefineInDict");
        defineInDict.addTextNode(AON_AWARE_NAMESPACE, "dictId", dictId);
        defineInDict.addTextNode(AON_AWARE_NAMESPACE, "word", word);
    }
}
```

You'll notice that there's some commonality between the classes - if you have this too, you'll probably want to pull that up into a superclass as I have in the [example](http://code.google.com/p/icesoap/source/browse/IceSoapExample#IceSoapExample%2Fsrc%2Fmain%2Fjava%2Fcom%2Falexgilleran%2Ficesoap%2Fexample%2Fenvelopes).

There's a bunch of methods on the SOAPEnvelope interface and I'm not going to go into them here (there's a Javadoc for that). The important concepts to remember are that an XML document is made up of many child nodes, which may in turn have their own child nodes (making them Parent Nodes), and so on. These nodes are represented by the XMLNode interface, which has a number of methods for setting up types, namespaces, attributes etc, and the XMLParentNode interface, which is for nodes that have child nodes. A SOAPEnvelope is basically a pre-baked XMLParentNode, with all the SOAP namespaces already setup and with two easily accessible child nodes (getHeader() and getBody()) for you to change as you wish.

Next up: Now I've got my envelopes, it's over to GettingStarted3 to parse the responses from them.