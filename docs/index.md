---
layout: doc
title: Getting Started
version: 0.13.0
---

Getting Started
---

This guide will get you started with the basic usage of EMFJson.

This guide requires a basic knowledge of the Eclipse Modeling Framework, ([EMF](http://www.eclipse.org/emf)) and its API. 

If you  are not familiar with those concepts, you can follow this excellent [tutorial](http://www.vogella.de/articles/EclipseEMF/article.html) 
or follow our [tutorial](/docs/tutorial/) to learn how to use EMF to create domain models and EMFJson to do JSON serialization.

## Installation

EMFJson can be use in standalone Java applications, OSGI and Eclipse based Plugins. In this 
tutorial we will use EMFJson in a standalone Java application build with Maven.

First, start by creating a new Maven project, then add the following dependency to the POM file.

```xml
<dependency>
    <groupId>org.emfjson</groupId>
    <artifactId>emfjson-jackson</artifactId>
    <version>{{ page.version }}</version>
</dependency>
```

This will add emfjson-jackson module as well as dependencies to EMF modules.

## Quick Start

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

```java
resourceSet.getPackageRegistry()
    .put(EcorePackage.eNS_URI, EcorePackage.eINSTANCE);
```

We can now create an Ecore model using the EMF dynamic API. It is also possible to use code generation to generate Java code 
 from EMF models, for this look at other tutorials about EMF.

An Ecore model is made of EPackage that contains a collections of classes (EClass) that represent the kind of objects we will create in our documents.

We create a package that contains a single class named User. This class contains a single attribute, ```name```. This way we 
will be able to create objects of type ```User``` and assign them a name. 

```java
EPackage samplePackage = EcoreFactory.eINSTANCE.createEPackage();
samplePackage.setName("sample");
samplePackage.setNsPrefix("sample");
samplePackage.setNsURI("http://emfjson.org/sample");

EClass userClass = EcoreFactory.eINSTANCE.createEClass();
userClass.setName("User");

EAttribute userName = EcoreFactory.eINSTANCE.createEAttribute();
userName.setName("name");
userName.setEType(EcorePackage.Literals.ESTRING);

userClass
    .getEStructuralFeatures()
    .add(userName);

samplePackage
    .getEClassifiers()
    .add(userClass);
```

After creating the EPackage, add it to a resource.


```java
Resource resource = resourceSet.createResource(
    URI.createURI("http://emfjson.org/sample"));
    
resource
    .getContents()
    .add(samplePackage);
```

We can now save the resource as JSON by invoking the save method.

```java
resource.save(null);
```

The result will be a JSON object representing our EPackage. 

```json
{
  "eClass" : "http://www.eclipse.org/emf/2002/Ecore#//EPackage",
  "name" : "sample",
  "nsURI" : "http://emfjson.org/sample",
  "nsPrefix" : "sample",
  "eClassifiers" : [ {
    "eClass" : "http://www.eclipse.org/emf/2002/Ecore#//EClass",
    "name" : "User",
    "eStructuralFeatures" : [ {
      "eClass" : "http://www.eclipse.org/emf/2002/Ecore#//EAttribute",
      "name" : "name",
      "eType" : {
        "$ref" : "http://www.eclipse.org/emf/2002/Ecore#//EString"
      }
    } ]
  } ]
}
```

The next step will be to create an instance of the class ```User```. For that we will still use the EMF dynamic API.

To create a new instance from a EClass, use the utility class EcoreUtil.
 
```java
EObject u1 = EcoreUtil.create(userClass);
```

That's it, we created a dynamic instance of the class User. We can now set a value for the attribute ```name```.
This is done by using the method ```eSet```.

```java
u1.eSet(userName, "u1");
```

Before we can serialize this instance in JSON, we need to add it to a Resource, like we did previously.

```java
Resource resource = resourceSet.createResource(
    URI.createURI("data.json"));

resource
    .getContents()
    .add(u1);
```

And save it.

```java
resource.save(null);
```

This will result in a single JSON object being our instance of User.

```json
{
  "eClass" : "http://emfjson.org/sample#//User",
  "name" : "u1"
}
```

That's it, you've learn how to save an EMF Resource in JSON with EMFJson. For more 
information about EMFJson features check the [documentation](/docs).
