---
title: "Writing geo spatial scripts with Java and jbang"
date: 2021-11-23T11:23:23-08:00
tags: ["java","jts","jbang","geospatial"]
featured_image: "img/beach2.jpg"
draft: false
---

Usually writing a Java application requires using Maven or Gradle.  While
both are great tools for large projects, they are too complicated for 
small scripts.  jbang is a new tool that lets you create scripts or simple
applications with Java.  I will create a geospatial command line tool using jbang and the Java Topology Suite (JTS).

<!--more-->

There are a few different ways to install [jbang](https://www.jbang.dev/), but I used [sdk man](https://sdkman.io/).

```bash
skd install jbang
```

You can use jbang to create a script.

```bash
jbang init buffer.java
```

Use your favorite editor to fill in the details.  Our script
will use the [Java Topology Suite](https://github.com/locationtech/jts) to read a WKT geometry from the command line, buffer that geometry using a distance from the command line, and then write the geometry as WKT.

First, add JTS as a dependency in our script:

```java
//DEPS org.locationtech.jts:jts-core:1.18.2
```

Then replace the main method with the following:

```java
WKTReader reader = new WKTReader();
Geometry geometry = reader.read(args[0]);
WKTWriter writer = new WKTWriter();
System.out.println(writer.write(geometry.buffer(Double.parseDouble(args[1]))));
```

and then add some imports:

```java
import org.locationtech.jts.io.WKTReader;
import org.locationtech.jts.io.WKTWriter;
import org.locationtech.jts.geom.Geometry;
```

The final result should look like this:

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//DEPS org.locationtech.jts:jts-core:1.18.2

import org.locationtech.jts.io.WKTReader;
import org.locationtech.jts.io.WKTWriter;
import org.locationtech.jts.geom.Geometry;

public class buffer {

    public static void main(String... args) throws Exception {
        WKTReader reader = new WKTReader();
        Geometry geometry = reader.read(args[0]);
        WKTWriter writer = new WKTWriter();
        System.out.println(writer.write(geometry.buffer(Double.parseDouble(args[1]))));
    }
    
}
```

We can use jbang to run our script (no pre-complication required).

```bash
jbang buffer.java "POINT(1 1)" 2
```

If we want to distribute our script, we can use the jbang export portable command to create a jar and lib directory (which contains the jars our script depends on).

```bash
jbang export portable buffer.java
```

We can try out the jar from the command line.

```bash
java -jar buffer.jar "POINT(1 1)" 2
```

We can also use GraalVM to create a native executable.  I used sdk man to install GraalVM version 21.3.0.

```bash
sdk install 21.3.0.r17-grl
sdk use java 21.3.0.r17-grl
```

To make sure things work correctly, try running your script again but this time with the --native flag.

```bash
jbang --native buffer.java "POINT(1 1)" 2
```

Now we can build the native executable with the jbang export portable command with the --native flag.

```bash
jbang export portable --native --output geom-buffer buffer.java
```

Try it out!

```bash
./geom-buffer "POINT(1 1)" 2
```

The combination of jbang and GraalVM make writing scripts with Java really easy.
