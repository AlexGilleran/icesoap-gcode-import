# Part 3: Annotating POJOs for Parsing #

Now I've got the envelopes to send to the web service, it's about time I did something about what gets sent back. This is where the magic happens in IceSoap.

You'll recall before that I wanted to get a list of dictionaries, and that I wanted the id and name of each one. The format of the response was something like this:

```
<soap:Body>
    <DictionaryListResponse xmlns="http://services.aonaware.com/webservices/">
        <DictionaryListResult>
            <Dictionary>
                <Id></Id>
                <Name></Name>
            </Dictionary>

            <!-- Other <Dictionary> objects -->

        </DictionaryListResult>
    </DictionaryListResponse>
</soap:Body>
```

So now I've got to make an object to hold those individual dictionaries:

```
public class Dictionary {
	private String id;
	private String name;
}
```

## Using `@XMLObject` ##

Obviously this isn't going to work as-is. I have to annotate it with IceSoap annotations. The first annotation to worry about is `@XMLObject` -  this tells the parser what the associated XML object is for this class. This is important because it allows the parser to stop parsing when it reaches the end of the associated XML object, and it also allows it to know when to stop parsing the current object and instantiate a new one in lists.

```
@XMLObject("//Dictionary")
public class Dictionary {
	private String id;
	private String name;
}
```

The @XMLObject (and @XMLField) annotation accepts a String-based XPath as a parameter. Note that this is only a _subset_ of XPath. It supports:
  * The absolute operator: "`/`"
  * The descendant operator: "`//`"
  * Relative XPaths in the `@XMLField` annotation but not the `@XMLObject` annotation (you'll find out about these below)
  * A single predicate per name ("`//xpath/xpath`" is fine, "`//xpath[@att='1']/xpath[@att='2']`" is fine, but "`//xpath[@att='1' and @att='2']/xpath`" is not.
  * The XPath union operator ("`|`")
  * The '@' prefix for attributes (e.g. "node/@attributename")

When you specify XPaths, you want to put as little detail in as you can get away with. For instance, I could put "`/Envelope/Body/DictionaryListResponse/DictionaryListResult/Dictionary`" and it'd still work, but I'd be left with an XPath that's both messier for me to read, and slower for the phone to parse, as the parser has to check every bit of information that you've put into the XPath before it can be sure it's at the right node.

## Using `@XMLField` ##

Now I've specified my object XPath, I can use the `@XMLField` annotation to tell the parser where to find the fields.

```
@XMLField("Id")
private String id;

@XMLField("Name")
private String name;
```

If you're familiar with XPath you'll notice that I've specified relative rather than absolute XPaths here. If you're _not_ familiar with XPath you'll just notice that there's no funny slashes at the beginning of the string. Either way you're right - when specifying fields you can specify a relative XPath - this is relative to the XPath you've specified in the `@XMLObject` annotation... so for the "id" field, the parser will be looking for "`//Dictionary/Id`". You can still specify absolute XPaths for fields if you like, but seeing how your fields should always be looking for XML tags that are inside the parent XML node that you specified in your `@XMLObject` annotation.

Note that you aren't limited to Strings with your fields. As of version 1.0.3, the following types are supported:
  * `long`
  * `float`
  * `int`
  * `double`
  * `boolean`
  * `BigDecimal`
  * `String`
  * `Date` (note that this requires you to specify the format of the incoming date using the same syntax as Java's `SimpleDateFormat` - check the [javadoc](http://icesoap.googlecode.com/git/IceSoap/javadoc/com/alexgilleran/icesoap/annotation/XMLField.html) for details)

Keep in mind that SOAP is a text-based protocol, and that as such specifying these fields just results in the parsing of their values using the standard Java method (e.g. `Integer.parseInt(String string)` for int). If the format of the text that's to be converted is invalid, the whole parser will throw an exception and quit. Most of the time you can rely on the format of incoming information so that's fine - if you can't be sure the format is always going to be right, then it's probably better to just leave that field as a string and perform sanitization/parsing in your own code.

## Using `@XMLField` For Your Own Objects ##
The exception to the list above is that when you annotate your own classes with `@XMLObject`, you can then use them as `@XMLField`-annotated-fields in other classes.

For instance, you'll recall from GettingStarted2 that I wanted to get two objects from the service - the `Dictionary` seen above, and the definitions. When I submit the envelope that I made in GettingStarted2, I get back a response like this:

```
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
   <soap:Body>
      <DefineInDictResponse xmlns="http://services.aonaware.com/webservices/">
         <DefineInDictResult>
            <Word></Word>
            <Definitions>
               <Definition>
                  <Word></Word>

                  <Dictionary>
                     <Id></Id>
                     <Name></Name>
                  </Dictionary>

                  <WordDefinition></WordDefinition>
               </Definition>
            </Definitions>
         </DefineInDictResult>
      </DefineInDictResponse>
   </soap:Body>
</soap:Envelope>
```

You'll notice that within the envelope there's a `<Dictionary>` tag that matches the annotations on the object I've just set up. So I can set up my `Definition` object like so:

```
@XMLObject("//Definitions/Definition")
public class Definition {
	@XMLField("Word")
	private String word;

	@XMLField("Dictionary")
	private Dictionary dictionary;

	@XMLField("WordDefinition")
	private String wordDefinition;
}
```

When this class is passed the parser, it will automatically parse the entire `dictionary` field, based on the annotations I specified in the `Dictionary` class. Easy as pie.

## Using `@XMLField` for Lists ##
If I had multiple "Dictionary" tags then I could parse all of them simply by changing the type of the field from `Dictionary` to `List<Dictionary>` like so:

```
@XMLObject("//Definitions/Definition")
public class Definition {
	@XMLField("Word")
	private String word;

	@XMLField("Dictionary")
	private List<Dictionary> dictionaries;

	@XMLField("WordDefinition")
	private String wordDefinition;
}
```

By declaring it as a `List` (or any type assignable to `List`, such as `ArrayList`), the parser will look for the XPath that you specified in the XMLField notation and parse a new instance of it every time it's encountered, simply adding it to the list each time. Note that this works whether the XML tags are one after the other, e.g.

```
<Dictionary>
    <Id></Id>
    <Name></Name>
</Dictionary>

<Dictionary>
    <Id></Id>
    <Name></Name>
</Dictionary>
```

... or if there's tags in between:

```
<Dictionary>
    <Id></Id>
    <Name></Name>
</Dictionary>

<AnnoyingIntermediateTag />

<Dictionary>
    <Id></Id>
    <Name></Name>
</Dictionary>
```

**Up next:** GettingStarted4, in which we build requests.