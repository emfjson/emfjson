---
layout: doc
title: emfjson-mongo
---

An easy to use adapter that works on top of the EMF Resource API to store and retrieve EMF Models on MongoDB.

## Download

Add the following dependency to your pom file.

```xml
<dependency>
    <groupId>org.emfjson</groupId>
    <artifactId>emfjson-mongo</artifactId>
    <version>{{ site.version.mongo }}</version>
</dependency>
```

You can also find the jars in [maven central](http://search.maven.org/#search|ga|1|emfjson-mongo)

## Usage

### Setup

Usage of emfjson-mongo is done through the standard EMF API. Configure your ResourceSet like in the following snippet and you will be ready to store your models in MongoDB.

```java
ResourceSet resourceSet = new ResourceSetImpl();

// register packages for use in non osgi env.
resourceSet.getPackageRegistry()
  .put(EcorePackage.eNS_URI, EcorePackage.eINSTANCE);

// register JSON de/serializers
resourceSet.getResourceFactoryRegistry()
  .getExtensionToFactoryMap()
  .put("*", new JsonResourceFactory());

// register mongo support
resourceSet.getURIConverter()
  .getURIHandlers()
  .add(0, new MongoHandler());
```

Create a Resource with a URI pointing to MongoDB. The URI must contains 3 segments identifying the database, the collection and the document id.

The URI should have the following form:

```
mongodb://{host}[:{port}]/{db}/{collection}/{id}
```

Creating a resource with a valid mongo URI looks like this:

```java
URI uri = URI.createURI("mongodb://localhost:27017/emfjson-test/models/model1");
Resource resource = resourceSet.createResource(uri);
```

Alternatively, you can use a URI mapping:

```java
resourceSet.getURIConverter().getURIMap().put(
	URI.createURI("http://resources/"),
	URI.createURI("mongodb://localhost:27017/emfjson-test/models/"));

Resource resource = resourceSet.createResource
  (URI.createURI("http://resources/model1"));
```

Override the name of the field to avoid problem with the reserved name of Bson. And set the URI handler, to avoid problem with external resources.

```java
Map<String, Object> options = new HashMap<>();
options.put(EMFJs.OPTION_URI_HANDLER, new IdentityURIHandler());
options.put(EMFJs.OPTION_REF_FIELD, "_ref");

resourceSet.getLoadOptions().putAll(options);
```

### Saving a document

First create a new instance.

```java
User u = ModelFactory.eINSTANCE.createUser();
u.setName("Bob");
```

Add it to your previously created resource.

```java
resource.getContents().add(p);
```

Call the method save on your resource, this will store the content of the resource as a JSON document inside MongoDB.

```java
resource.save(null);
```

### Loading a document

Simply call the load method on your resource.

```java
resource.load(null);
```
