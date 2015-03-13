# Using Custom SOAP Faults #

If you've read through the Getting Started guide (particularly GettingStarted5) then you've seen the default `SOAP11Fault`, which allows for use of the base SOAP 1.1 fault details. However, both SOAP 1.1 and 1.2 faults allow for a custom details element, in which custom XML can be returned to further describe the fault that's occured.

This probably isn't something that's going to be useful to most users of IceSoap - the most common use case is if you're communicating with your own service and hence have an idea of what will come up in the `detail` element. In order to use this, you'll have to know what will come up in the detail element in the event of a SOAP Fault (if you don't know this, you could try to get it by intentionally causing faults in SoapUI).

Say I have a service that remote controls a car (admittedly SOAP would be a poor choice for this kind of interface, but bare with me)... in the event of the car breaking down, the service gives me information about what's broken. It's a SOAP 1.1 service, so an example fault looks like this:

```
<Fault>
    <faultcode>TK421</faultcode>
    <faultstring>Out of fuel</faultstring>
    <faultactor>http://www.example.com/carservice</faultactor>
    <detail>
        <system>intake</system>
        <severity>high</severity>
        <fixability>simple</fixability>
    </detail>
</Fault>
```

Parsing these details is done in much the same way as a normal `XMLObject` - we already have the built-in `SOAP11Fault` for the `faultcode`, `faultstring`, and `faultactor` elements, so we'll just extend that class:

```
public class CarSOAPFault extends SOAP11Fault {
    @XMLField("detail/system")
    private String system;

    @XMLField("detail/severity")
    private String severity;

    @XMLField("detail/fixability")
    private String fixability;

    // etc etc etc
}
```

Note that we don't need an `@XMLObject` annotation because that's already done in the parent class. Now we've got our class, we've just got to tell our request to use it.

```
Request<Response, CarSOAPFault> request = requestFactory.buildRequest(url, envelope, action, Response.class, CarSOAPFault.class);
```

This time when we make the request, we add an extra parameter that specifies our `CarSOAPFault` as the class that should be used if a fault is encountered.

Because we've specified this class in the generic declaration of `request`, when we invoke `request.getSOAPFault()` it'll return our new CarSOAPFault class, and we can get the fields we defined in that class out of it.