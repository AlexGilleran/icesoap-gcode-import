# Using with Maven #

If you can, I recommend you use Maven to resolve the dependency to IceSoap - it's a bit of a hump to initially get working but it makes things much easier once you do.

IceSoap is hosted on the central repository by Sonatype, so all you have to do is put this into `<dependencies>` in your pom.xml file:

```
<dependency>
    <groupId>com.alexgilleran</groupId>
    <artifactId>icesoap</artifactId>
    <type>jar</type>
    <version>1.0.7</version>
</dependency>
```

For more detail look at the pom.xml in the example.

# Using without Maven #

If you don't want to use Maven or just can't be bothered that's fine too, the jars for all versions of IceSoap can be found on the Sonatype repo.

The latest version is v1.0.7:
  * [Jar with Dependencies](https://oss.sonatype.org/content/repositories/releases/com/alexgilleran/icesoap/1.0.7/icesoap-1.0.7-jar-with-dependencies.jar) This has IceSoap bundled with Jaxen, which it uses for XPath resolution - unless you're already using Jaxen for something, get this.
  * [Javadoc Jar](https://oss.sonatype.org/content/repositories/releases/com/alexgilleran/icesoap/1.0.7/icesoap-1.0.7-javadoc.jar) Contains the javadoc for the code
  * [Sources Jar](https://oss.sonatype.org/content/repositories/releases/com/alexgilleran/icesoap/1.0.7/icesoap-1.0.7-sources.jar) Contains the sources for the code
  * [Jar Without Dependencies](https://oss.sonatype.org/content/repositories/releases/com/alexgilleran/icesoap/1.0.7/icesoap-1.0.7.jar) Use this if you've already got Jaxen on the build path.

Previous versions of IceSoap can be found on [Sonatype](https://oss.sonatype.org/content/repositories/releases/com/alexgilleran/icesoap/).