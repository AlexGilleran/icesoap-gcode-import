# v1.0.5 #
## Change to annotations for lists annotated with `@XMLField` ##
In order to make improvements to the way lists inside XML objects are parsed, the way that XPaths are used has changed.

Previously when setting an `@XMLField` annotation on a list, you'd specify the root XPath to parse within on the field, then the XPath for the actual tag in the object would be set in the `@XMLObject` annotation for the parsed object.

e.g. for XML like this:

```
<Object>
    <Metadata>Blah</Medadata>
    <List>
        <ListItem>
            ...
        </ListItem>
        <ListItem>
            ...
        </ListItem>
    </List>
</Object>
```

... you would have classes like this:

```
@XMLObject("/Object")
public class Object {
    @XMLField("Metadata")
    private String metadata;

    @XMLField("List")
    private List<ListItem> list;
}

@XMLObject("//ListItem")
public class ListItem {
    // fields etc.
}
```

Now the whole idea of a root XPath is gone - instead, simply set the `@XPathField` value on the `List` field to be the XPath of each item that gets passed, i.e.

```
@XMLObject("/Object")
public class Object {
    @XMLField("Metadata")
    private String metadata;

    @XMLField("List/ListItem")
    private List<ListItem> list;
}

// No longer require an XMLObject annotation here.
public class ListItem {
    // fields etc.
}
```

Note that you no longer need to set the `@XMLObject` annotation on the list item class, as it's taken from the `@XMLField`.

# v1.0.2 #
Unfortunately this involves a number of interface changes - I know it's poor form to be changing stuff around for a minor release, but it's better that things get fixed now and have a more understandable interface as a whole.

The changes mainly stem from the fact that IceSoap now supports parsing SOAP Faults - this creates a bit of complexity, because SOAPFaults change based both on whether they come from a SOAP 1.1 or a SOAP 1.2 service, and also because they have `<detail>` tags that can include non-uniform information, which to be consumed by IceSoap has to be specified by you, the developer. Changes also stem from the fact that in v1.0.0, IceSoap was purely SOAP 1.1, whereas now it supports 1.2 as well.

The result of this is that many classes from v1.0.0 are the same, but are now prefixed with SOAP11 to specify that they are for SOAP11 only. A number of these must be changed:

### Change `BaseSOAPEnvelope` to `BaseSOAP11Envelope` ###
What was called `BaseSOAPEnvelope` is now `BaseSOAP11Envelope` - this is because the v1.0.0 `BaseSOAPEnvelope` specified default namespaces that were SOAP 1.1 specific. The new name reflects that it's for SOAP 1.1 only, and there's a counterpart `BaseSOAP12Envelope` for SOAP 1.2 calls.

### Change `SOAPRequest` and `SOAPListRequest` to `SOAP11Request` and `SOAP11Request` ###
What was `SOAPRequest` is now `SOAP11Request`. The new `SOAPRequest` interface is much the same, except that it has an extra generic type parameter to deal with the SOAP Fault type that will be parsed in the event of an HTTP 500 error code - `SOAP11Request` simply defaults this to `SOAP11Fault`, which is a very basic SOAP 1.1 fault with the basic SOAP 1.1 Fault features.


### Change `SOAPObserver` and `SOAPListObserver` to `SOAP11Observer` and `SOAP11ListObserver` ###
Because requests now carry an extra generic type, so do their associated observers. Much as with `Request`s, the `SOAPObserver` type defaults this to `SOAP11Fault`.

### Add `SOAP11Fault` Generic Parameter to `SOAP11Observer` methods ###
Unfortunately the methods of `SOAP11Observer` now need to change, by having the extra `SOAP11Fault` generic type added. Like so:

```
private SOAP11ListObserver<T> soapObserver = new SOAP11ListObserver<T>() {
	@Override
	public void onNewItem(Request<List<T>, SOAP11Fault> request, T item) {
		// Do things
	}

	@Override
	public void onCompletion(Request<List<T>, SOAP11Fault> request) {
		// Do things
	}

	@Override
	public void onException(Request<List<T>, SOAP11Fault> request, SOAPException e) {
		// Do things
	}
}
```

After changing all that around, it should compile and work as it did (if it doesn't, please email me or leave a comment!). However, you might want to look into the new ability to parse SOAPFaults - see SOAPFaultCustomisation.