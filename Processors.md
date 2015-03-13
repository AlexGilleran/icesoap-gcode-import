# The Problem #

99% of values in SOAP Services look like this:

```
<type>
    <field>value</field>
</type>
```

This is easy to convert to a java type. However, every now and again you'll run across something like this:

```
<type>
    <csv>1,2,3,4,5</csv>
</type>
```

Here the service has taken the (admittedly stupid) approach of putting some comma separated values into a field. Hopefully you can change the service to fix this, but if you can't you're going to have to deal with it on the client side.

# The Solution: Processors #

A _Processor_ in IceSoap is a class that converts input from a single XML value or attribute value (as a `String`) into whatever type you want, doing whatever processing you want along the way.

## Step 1: Create your processor ##

First you'll have to decide what type you want to process your value into. I'm going to process into a List of Integers to suit my input. Create a new class and implement the `Processor` interface... the `OutputType` generic is the type that you'll process the value to. You'll end up with something like this:

```
public class CSVProcessor implements Processor<List<Integer>> {
    @Override
    public List<Integer> process(String inputValue) {
        // ...
    }
}
```

The `process` method will return whatever type you've passed into the `OutputValue` generic. The `inputValue` is passed as a String made up of all the text inside the tag or the attribute that it will process. Write code in this method to do whatever processing you need to do, then return it as the type you've specified (this might involve parsing an integer or creating a whole new object and passing values into it). I'm going to use String.split() to get an array and then make an ArrayList from it.

```
public class CSVProcessor implements Processor<List<Integer>> {
    @Override
    public List<Integer> process(String inputValue) {
        String[] values = inputValue.split(",");

        List<Integer> integerList = new ArrayList<Integer>();
        for (String value: values) {
            integerList.add(Integer.parseInt(value));
        }

        return integerList;
    }
}
```

If this were more than an example you might want to include code to handle non-integer input values, but this'll do for an example.

## Step 2: Attach Your Processor to a Field ##
Now you've got a processor, you'll need to attach it to the field that you want to use it to process the value for.

```
/** A list of integer values - this comes from the service as a CSV*/
@XMLField(value = "csv", processor = CSVProcessor.class)
private List<Integer> valueList;
```

... and now I just run it and test it works. Simple, huh?