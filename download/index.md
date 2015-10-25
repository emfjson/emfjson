---
layout: down
title: Download
version: 0.13.0
---

# Download instructions

Choose the version of emfjson that is right for your target platform.

For server side or desktop Java applications, you need the version of emfjson-jackson that is available in Maven 
Central.

Eclipse plugins and Eclipse RCP developers should use the version of emfjson-jackson available in our p2 repository. This is 
the same version as the one available in Maven Central. We encourage the use of [Tycho](https://www.eclipse.org/tycho/), 
the maven plugin for Eclipse applications dependency management, for building EMF plugins and RCP apps.

Browser support is available through emfjson-gwt, that should be use with the [EMF-GWT generator](https://wiki.eclipse.org/EMF/GWT).
For Javascript based applications, take a look at [ecorejs](http://ghillairet.github.io/ecore.js/) that supports the emfjson format and 
can be in the browser or in nodejs applications.

# Download emfjson-jackson

emfjson-jackson is available both in [Maven Central](http://search.maven.org/#search|ga|1|a%3A%22emfjson-jackson%22) and
in a [p2 repository](http://ghillairet.github.io/p2).

## Maven users

Add the following dependency to your project [POM](http://maven.apache.org/pom.html).

```xml
<dependency>
    <groupId>org.emfjson</groupId>
    <artifactId>emfjson-jackson</artifactId>
    <version>{{ page.version }}</version>
</dependency>
```

## Non Maven users

For users who want to do not use Maven and want to download the jars, those are available on
[Maven Central](http://search.maven.org/#search|ga|1|%20a%3A%22emfjson-core%22%20OR%20a%3A%22emfjson-jackson%22). Click on
the jar links to download them.

## Eclipse users

You can install emfjson Jackson in your Eclipse via the following update site:

[http://ghillairet.github.io/p2](http://ghillairet.github.io/p2)

If you are using a version of Eclipse inferior to Mars (4.5), please add the following orbit repository, that contains the Jackson library, 
before trying to install emfjson. [http://download.eclipse.org/tools/orbit/downloads/drops/R20150519210750/repository/](http://download.eclipse.org/tools/orbit/downloads/drops/R20150519210750/repository/)

Eclipse Mars users can also install emfjson from the Eclipse Marketplace.

# Download emfjson-gwt

## Maven GWT

Generation of GWT compatible code is supported by the EMF code generator, see [EMF/GWT wiki](https://wiki.eclipse.org/EMF/GWT) for more information. For developing GWT 
applications, we encourage the use of the [GWT Maven Plugin](https://gwt-maven-plugin.github.io/gwt-maven-plugin/). To use emfjson-gwt, 
add the following dependency to your project POM file.

```xml
<dependency>
    <groupId>org.emfjson</groupId>
    <artifactId>emfjson-gwt</artifactId>
    <version>{{ page.version }}</version>
</dependency>
```
