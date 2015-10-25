---
layout: doc
title: emfjson - GWT
---

# GWT

Add the dependency to EMFJson in your GWT module.

```xml
<inherits name="org.emfjson.gwt.EMFJs" />
```

You can now use EMFJson in a GWT client via the EMF API.

```java
import org.eclipse.emf.ecore.resource.ResourceSet;
import org.eclipse.emf.ecore.resource.impl.ResourceSetImpl;
import org.emfjson.gwt.resource.JsonResourceFactory;

...

ResourceSet resourceSet = new ResourceSetImpl();
resourceSet.getPackageRegistry().put(EcorePackage.eNS_URI,EcorePackage.eINSTANCE);
resourceSet.getResourceFactoryRegistry().getExtensionToFactoryMap().put(
    "*",
    new JsonResourceFactory());
```

The difference with the standard EMF API, is that on GWT you will have to use the 
save and load methods with a callback parameter.

To save a resource on the client.

```java
import org.eclipse.emf.common.util.Callback;
import org.eclipse.emf.ecore.resource.Resource;

...

resource.save(null, new Callback<Resource>() {
    @Override
    public void onFailure(Throwable caught) {
        GWT.log(caught.getMessage());
    }
    @Override
    public void onSuccess(Resource result) {
        Window.alert("Successfully saved");
    }
});
```

To load a resource on the client.

```java
import org.eclipse.emf.common.util.Callback;
import org.eclipse.emf.ecore.resource.Resource;

...

resource.load(null, new Callback<Resource>() {
    @Override
    public void onFailure(Throwable caught) {
        GWT.log(caught.getMessage());
    }
    @Override
    public void onSuccess(Resource result) {
        Window.alert("Successfully loaded");
    }
});
```
# URI Handlers

EMFJSON-GWT provides different URI Handlers that are use by the EMF Resource framework to locate the Resources 
location on the server or in the browser local storage.

## GWT-RPC

GWT-RPC is the default mechanism provided by GWT to communicate with the server. To use GWT-RPC you need to define a mapping 
to your implementation of URIService in the web.xml configuration file. The URIService interface is provided by EMF-GWT.

```xml
<servlet>
	<servlet-name>uriService</servlet-name>
	<servlet-class>org.eclipselabs.emfjson.junit.model.server.ModelURIService</servlet-class>
</servlet>

<servlet-mapping>
	<servlet-name>uriService</servlet-name>
	<url-pattern>/model/uriService</url-pattern>
</servlet-mapping>
```

After that you can add the URIHandler initialized with your URIService implementation to your ResourceSet.

```java
import org.eclipse.emf.ecore.resource.URIService;
import org.eclipse.emf.ecore.resource.URIServiceAsync;
import org.eclipse.emf.ecore.resource.URIServiceCallback;
...

final URIServiceAsync modelService = GWT.create(URIService.class);
final URIServiceCallback modelCallback = new URIServiceCallback(modelService);

resourceSet.getURIConverter().getURIHandlers().add(modelCallback);
```

## HTTP + JSON

GWT also allow communication to an HTTP server delivering JSON data, see this [tutorial](https://developers.google.com/web-toolkit/doc/latest/tutorial/JSON). To use 
it in EMFJSON-GWT, you first need to defined the mapping in the web.xml configuration file. 

```xml
<servlet>
	<servlet-name>json</servlet-name>
	<servlet-class>org.eclipselabs.emfjson.junit.model.server.ModelHttpServlet</servlet-class>
</servlet>

<servlet-mapping>
	<servlet-name>json</servlet-name>
	<url-pattern>/models</url-pattern>
</servlet-mapping>
```

And add the HttpHandler to your ResourceSet.

```java
resourceSet.getURIConverter().getURIHandlers().add(0, new HttpHandler());
```

You may also need to define a URI mapping, that will map your Resource's URI to the HTTP server URI delivering the JSON data. 

```java
resourceSet.getURIConverter().getURIMap().put(
	URI.createURI("http://eclipselabs.org/emfjson/models/"),
	URI.createURI(GWT.getHostPageBaseURL() + "models/"));
```

## Local Storage

HTML5 local storage can also be used to store EMF/JSON models directly in the browser local store. For that add the LocalStorageHandler to 
your ResourceSet. The resources will be store in the local key/value store with for keys the resources URIs prefixed by emfjs:.

```java
resourceSet.getURIConverter().getURIHandlers().add(new LocalStorageHandler());
```