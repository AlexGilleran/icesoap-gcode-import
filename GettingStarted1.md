# Introduction #
IceSoap was designed with ease-of-use in mind, so it should be pretty quick to get up and running. These getting started pages will show you the steps to getting set up with a basic web service.

# Step 1: Figure out what you want to send and recieve #
Using IceSoap requires knowledge of the requests you want to send and the responses that you'll receive. Chances are, however, that all you've got is a WSDL and a go-get-'em attitude. That's fine too.

I strongly recommend the use of [SoapUI](http://www.soapui.org/) when setting up IceSoap. Basically, you throw it your WSDL and it generates sample requests for all the stated operations - you can then execute these to get responses. It also comes as an Eclipse plugin, which is most convenient when developing for Android.

I'm going to be using the AonAware dictionary service, which has a WSDL at http://services.aonaware.com/DictService/DictService.asmx?WSDL (until it goes down, I guess). You can use this if you want to follow the guide exactly, or substitute your own - I'll try to keep the steps generic enough that this will still be useful for you.

Having loaded my WSDL into SoapUI I get a list of operations I can perform:
```
Define
DefineInDict
DictionaryInfo
DictionaryList
DictionaryListExtended
Match
MatchInDict
ServerInfo
StrategyList
```

All I really want to do to start with is get a list of dictionaries, so I'll stick to the DictionaryList service. When I click on that I'll find a sample request, which looks something like this:

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:web="http://services.aonaware.com/webservices/">
   <soapenv:Header/>
   <soapenv:Body>
      <web:DictionaryList/>
   </soapenv:Body>
</soapenv:Envelope>
```

Well that's pretty simple! Whoever said SOAP had to be complicated? On a more serious note, if you're using a different service then your sample request will probably be a mess of nested information, as SoapUI will generate XML for every optional field it finds in the WSDL. Before you start coding, what you want to do here is whittle down the response until you're only passing in what you absolutely have to - it's that response that you'll then code up. Alternatively you _can_ just model the whole thing, but who wants to do more work for the same result?

I can't whittle this one down, so I'll just hit the "Submit" button.

```
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
   <soap:Body>
      <DictionaryListResponse xmlns="http://services.aonaware.com/webservices/">
         <DictionaryListResult>
            <Dictionary>
               <Id>devils</Id>
               <Name>THE DEVIL'S DICTIONARY ((C)1911 Released April 15 1993)</Name>
            </Dictionary>
            <Dictionary>
               <Id>easton</Id>
               <Name>Easton's 1897 Bible Dictionary</Name>
            </Dictionary>
            <Dictionary>
               <Id>elements</Id>
               <Name>Elements database 20001107</Name>
            </Dictionary>
            <Dictionary>
               <Id>foldoc</Id>
               <Name>The Free On-line Dictionary of Computing (27 SEP 03)</Name>
            </Dictionary>
            <Dictionary>
               <Id>gazetteer</Id>
               <Name>U.S. Gazetteer (1990)</Name>
            </Dictionary>
            <Dictionary>
               <Id>gcide</Id>
               <Name>The Collaborative International Dictionary of English v.0.44</Name>
            </Dictionary>
            <Dictionary>
               <Id>hitchcock</Id>
               <Name>Hitchcock's Bible Names Dictionary (late 1800's)</Name>
            </Dictionary>
            <Dictionary>
               <Id>jargon</Id>
               <Name>Jargon File (4.3.1, 29 Jun 2001)</Name>
            </Dictionary>
            <Dictionary>
               <Id>moby-thes</Id>
               <Name>Moby Thesaurus II by Grady Ward, 1.0</Name>
            </Dictionary>
            <Dictionary>
               <Id>vera</Id>
               <Name>Virtual Entity of Relevant Acronyms (Version 1.9, June 2002)</Name>
            </Dictionary>
            <Dictionary>
               <Id>wn</Id>
               <Name>WordNet (r) 2.0</Name>
            </Dictionary>
            <Dictionary>
               <Id>world02</Id>
               <Name>CIA World Factbook 2002</Name>
            </Dictionary>
         </DictionaryListResult>
      </DictionaryListResponse>
   </soap:Body>
</soap:Envelope>
```

Devil's Dictionary eh? Sounds interesting. The important thing here is to identify exactly what we need from the response. I personally just want to display a list of dictionary names and keep the ids for later lookups.

This means that the XML data that's interesting to me is this:

```
<Dictionary>
    <Id></Id>
    <Name></Name>
</Dictionary>
```

This is a pretty good web service in that it only gives me what I need. A lot of SOAP services will give you every tiny piece of related information like this:

```
<Dictionary>
    <Id></Id>
    <AnotherPointlessId></AnotherPointlessId>
    <Name></Name>
    <RevisionHistoryForNoReason>
        <Revision>...</Revision>
        <Revision>...</Revision>
        <Revision>...</Revision>
    </RevisionHistoryForNoReason>
</Dictionary>
```

It's important for you to identify exactly what you need and only model that in your app - every bit of cruft you ignore makes your app a bit better designed and a bit faster when parsing.

Next up: GettingStarted2, in which we actually write some Java code based on the information we just gleaned.