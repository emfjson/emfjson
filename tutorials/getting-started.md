---
layout: doc
title: Getting Started
version: 1.0.0-rc.1
---

Getting Started
---

This guide will get you started with the basic usage of emfjson-jackson. This guide requires a basic knowledge
of the Eclipse Modeling Framework, ([EMF](http://www.eclipse.org/emf)) and of its API.

If you  are not familiar with those concepts, you can follow this excellent [tutorial](http://www.vogella.de/articles/EclipseEMF/article.html) 
or follow our [guides](/docs/) to learn how to use EMF to create domain models with JSON.

## Download

The emfjson-jackson module can be use in standalone Java applications, OSGI and Eclipse based Plugins. In this
tutorial we will use it in a standalone Java application build with Maven.

First, start by creating a new Maven project, then add the following dependency to the POM file. That will add the emfjson-jackson module as well as
EMF and Jackson dependencies to your project.

```xml
<dependency>
    <groupId>org.emfjson</groupId>
    <artifactId>emfjson-jackson</artifactId>
    <version>{{ page.version }}</version>
</dependency>
```

## Setup the ResourceSet

Create a new Java file, and in the main method, create a ResourceSet and add the JsonResourceFactory to the 
factory map. 

```java
package sample;

import org.eclipse.emf.ecore.resource.ResourceSet;
import org.eclipse.emf.ecore.resource.impl.ResourceSetImpl;
import org.emfjson.jackson.resource.JsonResourceFactory;

public class Main { 

    public static void main(String[] args) {
        ResourceSet resourceSet = new ResourceSetImpl();
        resourceSet.getResourceFactoryRegistry()
            .getExtensionToFactoryMap()
            .put("*", new JsonResourceFactory());
    }
}
```

The ResourceSet is responsible for managing a collection of Resources (documents containing objects). The ResourceSet 
should know what kind of documents we want to create, that is why we specify a JsonResourceFactory after it's initialization. 
This way, each time we will ask the ResourceSet to create a new Resource, the latter will be a JsonResource able to read and write 
 documents in a JSON format.

As we are working in a standalone environment (i.e. outside OSGI), we also need to register the EPackages we 
are going to use, here the EcorePackage.

## Domain modeling with Xcore

We can now create an Ecore model using the EMF dynamic API. It is also possible to use code generation to generate Java code 
 from EMF models, for this look at other tutorials about EMF.

An Ecore model is made of EPackage that contains a collections of classes (EClass) that represent the kind of objects we will create in our documents.

We create a package that contains a single class named User. This class contains a single attribute, ```name```. This way we 
will be able to create objects of type ```User``` and assign them a name. 

```java
@Ecore(nsURI="http://emfjson.org/domain")
@GenModel(

)
package domain

class User {
	String name
	String email
}

class Comment {
	String text
	Date time
}
```


```java
resourceSet.getPackageRegistry()
    .put(EcorePackage.eNS_URI, EcorePackage.eINSTANCE);
resourceSet.getPackageRegistry()
    .put(DomainPackage.eNS_URI, DomainPackage.eINSTANCE);
```

The result will be a JSON object representing our EPackage.
The next step will be to create an instance of the class ```User```. For that we will still use the EMF dynamic API.

To create a new instance from a EClass, use the utility class EcoreUtil.
 
```java
User bob = DomainFactory.eINSTANCE.createUser();
bob.setName("Bob");
```

That's it, we created a dynamic instance of the class User. We can now set a value for the attribute ```name```.
This is done by using the method ```eSet```.

Before we can serialize this instance in JSON, we need to add it to a Resource, like we did previously.

```java
Resource resource = resourceSet.createResource(URI.createURI("http://localhost:8080/users/bob"));
resource.getContents().add(bob);
```

And save it.

```java
resource.save(null);
```

This will result in a single JSON object being our instance of User.

```json
{
  "eClass" : "http://emfjson.org/domain#//User",
  "name" : "Bob"
}
```

That's it, you've learn how to save an EMF Resource in JSON with EMFJson. For more 
information about EMFJson features check the [documentation](/docs).
