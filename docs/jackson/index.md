---
layout: doc
title: Jackson
---

# Jackson

EMFJson provides a Jackson Module that can be used to serialize and deserialize EMF 
Models as JSON with the Jackson API.

To use it, register the EMF module with a ObjectMapper.

```java
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new EMFModule(resourceSet));
```

You can now convert a Resource into a JsonNode

```java
JsonNode node = mapper.valueToTree(resource)
```

Write the content of a resource

```java
mapper.writeValueAsString(resource);
```

Or read JSON data into an EObject or a Resource

```java
String data = "{\"eClass\":\"http://www.eclipse.org/emf/2002/Ecore#//EPackage\"}";

EPackage p = (EPackage) mapper.readValue(data, EObject.class);

Resource r = mapper.readValue(data, Resource.class);
```
