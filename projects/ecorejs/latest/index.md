---
layout: doc
title: ecore.js
version: 0.9.0
navigation:
  - What is it
  - Downloads
  - Getting Started  
---

# What is it

**Ecorejs** brings Ecore [metamodeling](http://en.wikipedia.org/wiki/Metamodeling) to Javascript. It implements the Ecore model
in Javascript and includes JSON and [XMI](http://en.wikipedia.org/wiki/XML_Metadata_Interchange) (XML Metadata Interchange format)
writers and parsers for the Browser and [Node.js](http://nodejs.org/).

## Install

### Browser

Download Ecore.js from dist/ folder, and include it in your html along with underscore.js.

```
<script src="underscore.js"></script>
<script src="ecore.js"></script>
```

Alternatively you can use the dependency manager [Bower](http://bower.io/) to install Ecore.js in your project.

```
bower install ecore
```

### Node

Ecore.js is available on npm and can be use as a Node module. To install it simply use the following command from your terminal:

```
npm install ecore
```

Importing Ecore.js in a Node module is done as follow:


```
var Ecore = require('ecore');
```

# Getting Started

In this short guide, you will learn how to create an Ecore model with **ecore.js**, how to create instances of it, what is the
concept of resources and how to use them. You will also learn how to serialize resources into JSON and XMI and how to use events
to listen to changes made on models and instances.

## Install

The first step is to install Ecore.js.

If you intend to use it in the Browser, download ecore.js from the download links given
above, and include it in your html file. You will also need to include [underscore.js](http://underscorejs.org/) as ecore.js depends
on it. If you want to have XMI support, you'll also need to include [sax-js](https://github.com/isaacs/sax-js).

```
<script src="underscore.js"></script>
<script src="sax.js"></script> <!-- sax is needed for XMI support -->
<script src="ecore.js"></script>
```

Alternatively you can use the dependency manager [Bower](http://bower.io/) to install Ecore.js in your project.

```
bower install ecore
```

If you want to use ecore.js on the server side, e.g. with [Node.js](http://www.nodejs.org), simply use [npm](https://npmjs.org/) and
install the ecore package.

```
> npm install ecore

var Ecore = require('ecore');
```

## Models

In Ecore, models are made of packages (EPackage), classifiers (EClass, EDataType, EEnum) and structural features (EAttribute, EReference).
A package contains the classifiers while these latter are made of strcutural features.

```javascript
var LibraryPackage = Ecore.EPackage.create({
    name: 'library',
    nsPrefix: 'library',
    nsURI: 'http://www.example.org/library'
});
```

The example chosen is a model of library, adapted from a classic EMF example. This model represents the concept
of Book, Writer and Library and their relations.

The following code shows how to create the EClass Library.

```javascript
var Library = Ecore.EClass.create({
    name: 'Library',
    eStructuralFeatures: [
        Ecore.EAttribute.create({ name: 'name', eType: Ecore.EString });
    ]
});
```

The Library object is an ```EObject``` having for eClass ```Ecore.EClass```. This latter can be accessed via
the property ```eClass```.

```javascript
Library.eClass // -> Ecore.EClass
```

EObjects have getter and setter methods that allow access and modifications to their values. We can for example change
the value of the property name of Library like this:

```javascript
Library.get('name'); // -> Library
Library.set('name', 'Foo');
Library.get('name'); // -> Foo
```

Note that features with an upper bound > 1 return an EList and cannot be set.

```javascript
Library.get('eStructuralFeatures'); // -> EList([ EAttribute(name) ])
Library.get('eStructuralFeatures').add(aFeature);
Library.get('eStructuralFeatures').at(0); // -> EAttribute(name)
Library.get('eStructuralFeatures').remove(aFeature);
Library.get('eStructuralFeatures').array() // -> [EAttribute(name)]

// Note that derived features return an array, not an EList.
Library.get('eAllAttributes'); // -> [EAttribute(name)]
```

We can now create the remaining classes of our model.

```javascript
var Item = Ecore.EClass.create({
    name: 'Item'
    abstract: true,
    eStructuralFeatures: [
        { eClass: Ecore.EAttribute, name: 'publicationDate', eType: Ecore.EDate }
    ]
});

var BookCategory = Ecore.EEnum.create({
    name: 'BookCategory',
    eLiterals: [
        Ecore.EEnumLiteral.create({ literal: 'Mystery', value: 0 }),
        Ecore.EEnumLiteral.create({ literal: 'Science Fiction', value: 1 }),
        Ecore.EEnumLiteral.create({ literal: 'Biography', value: 2 })
    ]
});

var Book = Ecore.EClass.create({
    name: 'Book',
    eSuperTypes: [ Item ],
    eStructuralFeatures: [
        Ecore.EAttribute.create({ name: 'title', eType: Ecore.EString }),
        Ecore.EAttribute.create({ name: 'pages', eType: Ecore.EInt }),
        Ecore.EAttribute.create({ name: 'category', eType: BookCategory })
    ]
});

var Writer = Ecore.EClass.create({
    name: 'Writer',
    eStructuralFeatures: [
        Ecore.EAttribute.create({ name: 'name', eType: Ecore.EString })
    ]
});
```

The remaining part of our model is about relationships between classes. In Ecore, a relationship is modeled with a
EReference.

```javascript
var LibraryItems = Ecore.EReference.create({
    name: 'items',
    upperBound: -1,
    containment: true,
    eType: Item
});

var LibraryWriters = Ecore.EReference.create({
    name: 'writers',
    upperBound: -1,
    containment: true,
    eType: Writer
});

var BookAuthor = Ecore.EReference.create({
    name: 'authors',
    upperBound: -1,
    eType: Writer
});
```

And we add those references to their respective classes.

```javascript
Library.get('eStructuralFeatures')
    .add(LibraryItems)
    .add(LibraryWriters);

Book.get('eStructuralFeatures').add(BookAuthor);
```

## Resources

Resources are containers for models and their instances. They are created from a ResourceSet and are identified by a URI.

```javascript
var resourceSet = Ecore.ResourceSet.create();
var resource = resourceSet.create({ uri: '/library' });
```

Every EObjects must be contained in a Resource, whether directly or via their parent EObject. For example, when adding
LibraryPackage in a Resource, all it's contained EObject (the classifiers) will be also contained by the resource.

```javascript
resource.get('contents').add(LibraryPackage);
LibraryPackage.eResource(); // -> resource
```

Loading resources from the server by doing an ajax call is done via ```load```. The method takes for
parameter a success callback and an error callback.

```javascript
resource.load(function(resource) {
    console.log('success loading', resource);
}, function(e) {
    console.log('error', e);
});
```

Fetching the content of a ResourceSet from the server is done via ```fetch```. This will trigger a ```change```
event when done.

```javascript
resourceSet.fetch();
```

## Instances

You can now create instances of Library and Book by calling the method create.

```javascript
var myLibrary = Library.create({ name: 'My Library' });
var emfBook = Book.create();

myLibrary.get('items').add(emfBook);
```

You can access the istance properties via the methods get and set like this.

```javascript
emfBook.set('title', 'EMF Modeling Framework');
emfBook.get('title'); // -> EMF Modeling Framework
```

Note also that underscore collections functions are available on EList, so you can use each, map, reduce, etc., functions
on EList.

```javascript
myLibrary.get('books').map(function(book) { return book.get('title'); });
```

## Events

Every EObject and EList are given the ability to bind and trigger custom named events. Defaults events are triggered when an
EObject value is set or modified (change event) or when an element is added or removed from an EList (add and remove events). Events
are also available on Resource and ResourceSet.

To bind a callback to an EObject, call the method ```on```. The callback will be invoked whenver the event is fired. Note that events
can be namespaces, for example the event ```change:title``` will only be fire when the attribute title is modified.

```javascript
myLibrary.on('change:title', function(changed) {
    console.log('title changed');
});

var onAddBook = function(changed) {
    console.log('book added');
};
myLibrary.on('add:items', onAddBook);
```

Triggering events is done by calling ```trigger```.

```javascript
myLibrary.trigger('add remove');
```

Events can be removed by calling ```off```.

```javascript
myLibrary.off('add:items', onAddBook);
```

## Working with JSON

JSON is the default serialization format supported by ecore.js. The JSON format is described [here](https://github.com/ghillairet/emfjson)
and looks like this:

```javascript
{
    "eClass" : "/library#//Library",
    "name" : "myLibrary",
    "items" : [
        {
            "eClass": "/library#//Book",
            "title": "EMF Modeling Framework"
        }
    ]
}
```

Getting the JSON representation of a Resource can be done by invoking ```Resource.to```.

```javascript
resource.to() // -> JSON object
```

Parsing a JSON object into a Resource can be done via ```parse```.

```javascript
resource.parse(data);
```

## Working with XMI

XMI support is provided in ecore.js, but it is limited and does not support all the XMI specification. This support requires [sax.js](https://github.com/isaacs/sax-js).

Getting the XMI representation of a Resource is also done via the method ```to```. This time you'll need to indicate that you
want to obtain XMI by adding Ecore.XMI as parameter.

```javascript
resource.to(Ecore.XMI, true); // returns the XMI string
```

Parsing XMI is done via the ```parse``` method.

```javascript
resource.parse(data, Ecore.XMI); // data being a string containing the XMI.
```
