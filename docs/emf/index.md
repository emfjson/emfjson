---
layout: doc
title: emfjson - tutorial
---

# EMF

To use EMFJson with the EMF API, simply add the JsonResourceFactory to a ResourceSet Factory map.

```java
ResourceSet resourceSet = new ResourceSetImpl();
resourceSet.getResourceFactoryRegistry()
	.getExtensionToFactoryMap()
	.put("*", new JsonResourceFactory());
```

Create a Resource and add an object to it.

```java
Resource resource = resourceSet.createResource(URI.createURI("test"));
EPackage p = EcoreFactory.eINSTANCE.createEPackage();
resource.getContents().add(p);
```

Save it's content.

```java		
resource.save(System.out, null);
```

The output will be:

```json
{
  "eClass" : "http://www.eclipse.org/emf/2002/Ecore#//EPackage"
}
```