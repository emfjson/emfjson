---
layout: page
title: Specification
menus:
    - 
        id: json-document
        title: JSON Document
    - 
        id: eobject
        title: EObject
---

The EMF/JSON format specification presents the format use to represent EMF models in JSON. This format preserves
the specifics of EMF, e.g. object fragment identifiers, inter and cross documents references.

## JSON Document

A JSON document contains the description of a model. It can contain a single element or a collection of elements in
the form of an array. Each JSON object contains the description of an EObject. The format use to represent an EObject
is explained [later](#eobject).

This is a single element document.

```json
{
    "eClass": "ecore:EClass",
    "name": "Foo",
    "eStructuralFeatures": [
        {
            "eClass": "ecore:EAttribute",
            "name": "bar",
            "eType": "ecore:EString"
        },
        {
            "eClass": "ecore:EReference",
            "name": "foos",
            "eType": {
                "$ref": "//Foo"
            }
        }
    ]
}
```

This is a multi elements document.

```json
[
    {
        "eClass": "..."
    },
    {
        "eClass": "..."
    }
]
```

## EObject

EObject are represented in JSON in the form of JSON object. Each key of the JSON object represents a structural feature (EAttribute or EReference) of the EObject. The key of the object is the name of the structural feature. The associated value is the value of the structural feature. The value can be represented in the form of a string, number, boolean, object or array depending of the type of the feature.

Each JSON object corresponding to an EObject can contain a special key named `eClass` that gives the type of the EObject in the
form of a URI or prefixed value. The following excerpt presents the representation of an instance of EClass as JSON object.

```json
{
    "eClass": "http://www.eclipse.org/emf/2002/Ecore#//EClass",
    "name": "Foo"
}
```

## EList

EList are represented in the form of JSON arrays. Each element of the array is a JSON object.

```json
{
    "eClass": "ecore:EClass",
    "eStructuralFeatures": [
        { ... },
        { ... }
    ]
}
```

## EAttributes

EAttributes are properties of EObjects and are mapped to JSON key values, where values are primitive types (string, number, boolean).

```json
{
    "name": "Joe",
    "age": 18,
    "male": true
}
```

## EReferences

EReferences represent links between EObjects. They can be containments or references. In both cases, they can link elements
from the same document or link elements from different documents.

### Containments

Single value containment:

```json
{
    "eClass": "...",
    "element": {
        "eClass": "..."
    }
}
```

Multi value containment:

```json
{
    "eClass": "...",
    "elements": [
        {
            "eClass": "..."
        },
        {
            "eClass": "..."
        }
    ]
}
```

### Inner document references


References are represented as a JSON object containing a key ```$ref```. In the case of an inner document reference, the value of
the key is the fragment identifier of the referenced object.

Single value reference:

```json
{
    "eClass": "http://www.eclipselabs.org/emfjson/junit#//Node",
    "label": "root",
    "target": {
        "$ref": "//@child.0"
    },
    "child": [
        {
            "eClass": "http://www.eclipselabs.org/emfjson/junit#//Node",
            "label": "n1",
            "source" : {
                "$ref": "/"
            }
        }
    ]
}
```

Multi value references are represented by JSON object in an array:

```json
{
    "eClass": "...",
    "element": [
        {
            "$ref": "//@foo.0/@foo.1"
        },
        {
            "$ref": "//@foo.0/@foo.2"
        }
    ]
}
```

### Cross document references

Single value reference:

```json
{
    "userId": "1",
    "name": "Paul",
    "friends": [
        {
            "$ref" : "platform:/plugin/org.eclipselabs.emfjson.junit/tests/test-proxy-2.json#2"
        } 
    ],
    "uniqueFriend": {
        "$ref" : "platform:/plugin/org.eclipselabs.emfjson.junit/tests/test-proxy-2.json#3"
    }
}
```

