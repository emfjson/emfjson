---
layout: doc
title: emfjson-jackson
navigation:
  - What is it
  - Where can I use it
  - Download
  - Getting Started
  - EMF API
  - Object mapper API
  - Annotations
  - Customization
links:
  - label: Github
    url: https://github.com/emfjson/emfjson-jackson
help: true
---

## What is it

The emf-json jackson module provides JSON serialization and deserialization for the Eclipse Modeling Framework ([EMF](https://eclipse.org/modeling/emf)). It is based on the popular JSON library for Java,  [Jackson](https://github.com/FasterXML/jackson).

The emf-json jackson module is meant to be used by EMF application developers who want to replace the default XML serialization with JSON.

## Where can I use it

The emf-json jackson module can be used in standalone Java applications as well as in OSGI based Java application, such as Eclipse plugins.

## Download

It can be downloaded from the maven central repository by build tools such as Maven or Gradle. It can also be downloaded from a p2 update site

### Maven

To use it in Maven, add this following dependency in your project pom file.

```xml
<dependency>
    <groupId>org.emfjson</groupId>
    <artifactId>emfjson-jackson</artifactId>
    <version>{{ site.version.jackson }}</version>
</dependency>
```

### Gradle

To use it in a Gradle build, add this dependency in your project gradle file.

```groovy
compile 'org.emfjson:emfjson-jackson:{{ site.version.jackson }}'
```

### Eclipse update site

For Eclipse plugins developers, a p2 update site is available and can be used to install this module inside Eclipse as a plugin, and also be used with [Maven/Tycho](https://eclipse.org/tycho/).

Link to the update site [http://ghillairet.github.io/p2](http://ghillairet.github.io/p2)

To use emf-json jackson with Maven/Tycho, add the following to your project pom file.

```xml
<repository>
  <name>emfjson</name>
  <url>http://ghillairet.github.io/p2</url>
  <layout>p2</layout>
</repository>
```

### Download the jars

The jar file can be found in the maven central repository, [here](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22emfjson-jackson%22).

> Note that if you choose to download this jar directly you will also need to download the EMF and Jackson jars.

# Getting started

## EMF API

We will now see how to setup and use this library to serialize EMF models into JSON. The emf-json jackson module can be used in different ways, through the standard EMF API or through the Jackson API. We will start with the most common way for EMF developers, that is by using it with the EMF Resource API. This section assumes that you are already familiar with the Eclipse Modeling Framework, if not you can first go through these  [great](http://www.vogella.de/articles/EclipseEMF/article.html)  [tutorials](http://eclipsesource.com/blogs/tutorials/emf-tutorial/).

### Setup

The emf-json jackson module integrates seamlessly with the EMF Resource API by providing a Resource Factory that can be register to the ResourceSet.

See how it is done.

```java
import org.eclipse.emf.ecore.resource.ResourceSet;
import org.eclipse.emf.ecore.resource.impl.ResourceSetImpl;
import org.emfjson.jackson.resource.JsonResourceFactory;

ResourceSet resourceSet = new ResourceSetImpl();
resourceSet.getResourceFactoryRegistry()
				.getExtensionToFactoryMap()
				.put("json", new JsonResourceFactory());
```

And that's all there is to do.

By simply registering the `JsonResourceFactory` to the resourceSet, you are now able to read and write all the models, having a `.json` extension to their file names, in JSON.

You can register the `JsonResourceFactory` to any extension, or register it for more than one extension, or use it for all extensions by using `*` as extension.

### Writing JSON

We can now create EMF Resources using the standard EMF API. For example let's create a Resource named `data.json`.

```java
import org.eclipse.emf.common.util.URI;
import org.eclipse.emf.ecore.resource.Resource;

Resource resource = resourceSet.createResource
  (URI.createFileURI("src/main/resources/data.json"));
```

The resourceSet will know how to create resources that can handle JSON, since we register a `JsonResourceFactory` for the files with a `json` extension. This factory will create a `JsonResource` for that case. A `JsonResource` is a special kind of `Resource` that implements
methods to write and read JSON content.

We can now add some content to the resource. For that we need to create an object using a model factory.

```java
User bob = DomainFactory.eINSTANCE.createUser();
bob.setId(1);
bob.setName("Bob");
```

And we add that object to the resource, and save the resource using the `save` method. This will create a file named `data.json`.

```java
resource.getContents().add(bob);
resource.save(null);
```

If we now open the file `data.json`, we should see that it contains a JSON object. This JSON object is the representation of our EMF object.

It contains a field `eClass` that indicates the type of the object. By default the value is the URI of the object's EClass. The other fields contain the values of the object's attributes.

```json
{
  "eClass" : "http://emfjson.org/domain#//User",
  "id": 1,
  "name" : "Bob"
}
```

### Reading JSON

Reading a JSON file is done via the standard `load` method of a EMF resource. Considering the previous file `data.json`, we can read it's content by creating another resource, and calling the `load` method like this.

```java
Resource resource = resourceSet.createResource
  (URI.createFileURI("src/main/resources/data.json"));

resource.load(null);
```

The content of the resource can now be access like this:

```java
User u1 = (User) resource.getContents().get(0);
```

## Object Mapper API

You can also use the Object Mapper API provided by the Jackson library to serialize and deserialize the content of
a EMF resource or de/serialize only a single EObject.

### Setup

The setup is done by registering a EMFModule instance to an ObjectMapper.

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.emfjson.jackson.module.EMFModule;

ObjectMapper mapper = new ObjectMapper();
EMFModule module = new EMFModule();
mapper.registerModule(module);
```

Alternatively you can use the static method `setupDefaultMapper` from `EMFModule`
to initialize an ObjectMapper with a pre defined configuration.

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.emfjson.jackson.module.EMFModule;

ObjectMapper mapper = EMFModule.setupDefaultMapper();
```

We saw previously that the resourceSet needs a `JsonResourceFactory` to be register so that it knows how to
handle JSON content. The `JsonResourceFactory` contains an `ObjectMapper` that is either pre-configured or can
be setup by the user and pass as an argument to the `JsonResourceFactory` constructor.

```java
ObjectMapper mapper = new ObjectMapper();
EMFModule module = new EMFModule();
// ...
// configure the module and the mapper here...
mapper.registerModule(module);

JsonResourceFactory factory = new JsonResourceFactory(mapper);

// obtain the mapper later like this
ObjectMapper mapper = factory.getMapper();
```

### Writing JSON

The ObjectMapper can be use to convert any Java objects into JSON. This works as well for EMF Objects and Resources once
we have setup the mapper with the EMFModule.

Considering the following resource and object, same as we did before.

```java
Resource resource = resourceSet.createURI
  (URI.createURI("src/main/resources/data.json"));

User bob = DomainFactory.eINSTANCE.createUser();
bob.setId(1);
bob.setName("Bob");

resource.getContents().add(bob);
```

The resource can be serialize into JSON directly by using the mapper. For example,
it can be serialize into a String by using the `writeValueAsString` method.

```json
String jsonString = mapper.writeValueAsString(resource);
```

Or it can be converted into a `JsonNode` by using the `valueToTree` method.

```json
JsonNode jsonNode = mapper.valueToTree(resource);
```

EMF Objects can also be serialize independently. For that just pass the object you want to serialize to one of
the mapper write method.

```json
String jsonString = mapper.writeValueAsString(bob);
```

### Reading JSON

Using the mapper to parse JSON data into EObjects require some configuration. The mapper indeed needs to know what is the current resourceSet that it should use. This is done by setting attribute to the mapper reader.

In that example, we tell the mapper's reader which resourceSet to use as well as which URI should be set to the resulting resource.

```java
JsonNode data = ...

Resource resource = mapper
  .reader()
	.withAttribute(EMFContext.Attributes.RESOURCE_SET, resourceSet)
  .withAttribute(EMFContext.Attributes.RESOURCE_URI, "src/main/resources/data.json")
	.forType(Resource.class)
	.readValue(data);
```

These assertions should then be verified.

```java
assertTrue(resourceSet.getResources().contains(resource));
assertEquals(URI.createURI("src/main/resources/data.json"), resource.getURI());
```

Similarly it is possible to read single objects with a mapper. In the following example, we tell the mapper's reader to add the parsed object into a specific resource.

```java
JsonNode data = ...
Resource resource = ...

User user = mapper.reader()
	.withAttribute(EMFContext.Attributes.RESOURCE, resource)
	.forType(User.class)
	.readValue(data);
```

This assertion should then be verified.

```java
assertTrue(resource.getContents().contains(user));
```

# Annotations

Model elements can be annotated with various annotations similar to the one found in the Jackson library. These
annotations will be use by the mapper to modify the output of the serializer.

## JsonProperty

This annotation can be used on any kind of features to modify the field name.

```xcore
package org.emfjson.sample

annotation "JsonProperty" as JsonProperty

class User {
	@JsonProperty( value = "user_name" )
	String name
}
```

The output

```json
{
  "eClass": "http://emfjson.org/domain#//User",  
  "user_name": "Bob"
}
```

This annotation can also be used on operations. In that case the annotation indicates the mapper that the resulting
value of the operation execution should be serialize.

> Note that this annotation will work only on parameter less operations.

```xcore
package org.emfjson.sample

annotation "JsonProperty" as JsonProperty

class User {
	@JsonProperty( value = "user_name" )
	String name

  @JsonProperty
  op String hello() {
    "hello World"
  }
}
```

This will result in this output

```json
{
  "eClass": "http://emfjson.org/domain#//User",  
  "user_name": "Bob",
  "hello": "Hello World"
}
```

# Customization

The emfjson-jackson module allows several customizations to the JSON format it can handle. It is possible to customization the reserved fields names for types, ids and references. It is also possible to customize how the module will resolve an EClass or a reference from a JSON content.

## Type field

The type field is use to determine the EClass of the object. By default the field is named `eClass` and it's value is the URI of the EClass. The URI is in general the EClass package nsURI follow by a fragment being the EClass name. This URI is used by the resourceSet to locate the resource containing the package definition.

### Custom field name

The field name can be change by configuring the EMF module. The configuration is done by using a EcoreTypeInfo.

```java
import org.emfjson.jackson.annotations.EcoreTypeInfo;

ObjectMapper mapper = new ObjectMapper();
EMFModule module = new EMFModule();
module.setTypeInfo(new EcoreTypeInfo("type"));
mapper.registerModule(module);
```

Once this is done, all objects will now contain a `type` field instead of a `eClass` field. The value of that field is still the EClass URI.

```json
{
  "type": "http://emfjson.org/domain#//User",
  "id": 1,
  "name": "Bob"
}
```

### Custom serializer

To change the value of the type field, it is necessary to tell the module to use a specific `ValueWriter`. This is done by passing a second argument to the `EcoreTypeInfo`.

In this example, the `ValueWriter` will return the EClass name instead of it's URI.

```java
module.setTypeInfo(new EcoreTypeInfo("type",
  new ValueWriter<EClass, String>() {
	  @Override
	  public String writeValue(EClass value, SerializerProvider context) {
		  return value.getName();
	  }
  }));
```

The JSON output will now be like this

```json
{
  "type": "User",
  "id": 1,
  "name": "Bob"
}
```

### Custom deserializer

It is also possible to parse custom type values by specifying a `ValueReader`. This reader will take as input a string and return the EClass corresponding to that string.

In this example, we assume that the values of the type field is the name of the EClass. In that case we return the EClass from our domain package that matches that name.

```java
module.setTypeInfo(new EcoreTypeInfo("type",
  new ValueReader<String, EClass>() {
	  public EClass readValue(String value, DeserializationContext context) {
		  return (EClass) ModelPackage.eINSTANCE.getEClassifier(value);
	  }
  }));
```

When customizing both the serialization and deserialization of a type field, the valueReader and valueWriter
have to be set to the same `EcoreTypeInfo`.

```java
module.setTypeInfo(new EcoreTypeInfo("type", valueReader, valueWriter));
```

## Custom id field

The id field is used to uniquely identify an object inside a resource. The id field is in general only use when the option `OPTION_USE_ID` is set.

### Custom field name

To customize this field, it is necessary to tell the module to use a custom `EcoreIdentityInfo`. In the following example we tell the module to serialize all id fields as `_id`.

```java
EMFModule module = new EMFModule();
module.configure(EMFModule.Feature.OPTION_USE_ID, true);
module.configure(EMFModule.Feature.OPTION_SERIALIZE_TYPE, false);

module.setIdentityInfo(new EcoreIdentityInfo("_id"));
mapper.registerModule(module);
```

This will result in that output. Note that in that case, there are no type field because we have set the option
`OPTION_SERIALIZE_TYPE` to false.

```json
{
  "_id": 1,
  "name": "Bob"
}
```

### Custom serializer

It is possible to use a custom serializer for ids. In that case we tell the module to use a specific `ValueWriter` for ids. This writer takes as input an object and return a value. That value can be of any type.

```java
module.setIdentityInfo(new EcoreIdentityInfo("_id",
  new ValueWriter<EObject, Object>() {
	  @Override
	  public Object writeValue(EObject value, SerializerProvider context) {
		  return 1;
	  }
  }));
```

The output will be

```json
{
  "_id": 1,
  "name": "Bob"
}
```

### Custom deserializer

To deserialize custom id values, it is necessary to tell the module to use a custom `ValueReader` for ids. This reader will take as input a value (can be of any type) and should return a String.

```java
module.setIdentityInfo(new EcoreIdentityInfo("_id",
  new ValueReader<Object, String>() {
	  @Override
    public String readValue(Object value, DeserializationContext context) {
      return value.toString();
    }
  }));
```

## Custom reference handling

References are by default serialized as JSON objects that contain two fields. The first field is the type of the referenced object and the second field is the URI of the referenced object. The type field is named `eClass` and the URI field is named `$ref`.

Here is an example of references as serialize in JSON

```json
{
  "eClass": "http://emfjson.org/domain#//User",
  "name": "Bob",
  "friends": [ {
    "eClass": "http://emfjson.org/domain#//User",
    "$ref":"src/main/resources/data2.json#/"
  } ]
}
```

It is possible to fully customize how references are serialize and deserialize, meaning that it is not only possible to customize the field names but also add more fields to the reference field or use simple values instead of objects to represent references.

### Custom reference object fields

To customize the field names, it is necessary to tell the module to use a special `EcoreReferenceInfo`. The latter will take two arguments. The first being the name to use for the URI field and the second being the name to use for the type field.

Here we tell the module to use `my_ref` instead of `$ref` and to use `my_type` instead of `eClass`.

```java
EMFModule module = new EMFModule();
module.setReferenceInfo(new EcoreReferenceInfo.Base("my_ref", "my_type"));
mapper.registerModule(module);
```

This will give us such output

```json
{
  "eClass": "http://emfjson.org/domain#//User",
  "name": "Bob",
  "friends": [ {
    "my_type": "http://emfjson.org/domain#//User",
    "my_ref":"src/main/resources/data2.json#/"
  } ]
}
```

### Custom reference serializer

It is also possible to use a custom serializer for references. This should be use when you want to fully control how references are serialize.

The serializer is register directly to the module. It takes as input an `EObject`, that is the object that is referenced. It is then up to you to use the `JsonGenerator` to decide how the reference should be serialize.

In this example we decided to serialize the reference as a simple string. The string being the id of the object.

```java
module.setReferenceSerializer(new JsonSerializer<EObject>() {
  @Override
  public void serialize(EObject v, JsonGenerator g, SerializerProvider s)
  throws IOException {
    g.writeString(((JsonResource) v.eResource()).getID(v));
  }
});
```

We will then have an output like this

```json
{
  "eClass": "http://emfjson.org/domain#//User",
  "name": "Bob",
  "friends": [ "2" ]
}
```

### Custom reference deserializer

When using custom serializer for references, it is necessary to tell the module how to parse those. This can be done by using a custom reference deserializer.

The deserializer is register directly to the module. It is a standard Jackson deserializer that expects as output a `ReferenceEntry`. A `ReferenceEntry` is an object that contains the necessary information needed by the module to locate and instantiate an EObject.

In the following example, we assume that references are strings that contain ids of referenced objects. The deserializer reads the current reference id by calling `parser.getText()` and creates a `ReferenceEntry`.

```java
module.setReferenceDeserializer(new JsonDeserializer<ReferenceEntry>() {
  @Override
  public ReferenceEntry deserialize(JsonParser parser, DeserializationContext ctxt)
  throws IOException {
    final EObject parent = EMFContext.getParent(ctxt);
    final EReference reference = EMFContext.getReference(ctxt);

    if (parser.getCurrentToken() == JsonToken.FIELD_NAME) {
      parser.nextToken();
    }

    return new ReferenceEntry.Base(parent, reference, parser.getText());
  }
});
```
