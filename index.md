---
layout: default
title: emfjson
---

<div class="site-hero">
    <section>
        <div>
            <div>
                <div class="shadow-text hero-text">
                    <h1>Eclipse Modeling Framework meets JSON</h1>
                    <p class="hero-heading">
                        JSON as data format for the Eclipse Modeling Framework. 
                    </p>
                    <p>
                        Works in Eclipse, the browser and server side.
                    </p>
                </div>

                <div class="hero-buttons">
                    <a class="button" href="/docs">Get Started</a>
                    <a class="button" href="/download">Download</a>
                </div>
            </div>

        </div>
    </section>
</div>

<section class="features">
    <div>
        <div>
            <h2>Simple, Fast and Customizable</h2>
            <p>
            Integration with EMF Resource and Jackson APIs. 
            Uses Jackson Streaming API for performance.
            </p>
            <a href="/docs">More...</a>
        </div>

        <div>
            <h2>Support for various platforms</h2>
            <p>
            Build for standalone and OSGI environments (e.g. Eclipse plugins),
            GWT and Javascript.
            </p>
            <a href="/docs">More...</a>
        </div>

        <div>
            <h2>Support for Json Databases</h2>
            <p>
            You can easily interact with CouchDB and 
            MongoDB through adapters.
            </p>
            <a href="/docs">More...</a>
        </div>
    </div>
</section>

<section>

<h1>Getting Started</h1>

<h2>Simple to use, only register the correct resource factory</h2>

{% highlight java %}
ResourceSet resourceSet = new ResourceSetImpl();
resourceSet.getResourceFactoryRegistry()
    .getExtensionToFactoryMap()
    .put("*", new JsonResourceFactory());
{% endhighlight %}

<h2>Create your objects with the EMF API</h2>

{% highlight java %}
User u1 = UserFactory.eINSANCE.createUser();
u1.setId("u1");
u1.setName("Alice");

User u2 = UserFactory.eINSANCE.createUser();
u2.setId("u2");
u2.setName("Eric");

u1.getFriends().add(u2);
{% endhighlight %}

<h2>Add your objects to a resource and save it</h2>

{% highlight java %}
Resource res = resourceSet.createResource(URI.createURI("example.json"));
res.getContents().add(u1);
res.getContents().add(u2);

res.save(System.out, options);
{% endhighlight %}

<h2>Result</h2>

{% highlight json %}
[
    { 
        "id": "u1",
        "name": "Alice",
        "friends": [ "u2" ]
    },
    {
        "id": "u2",
        "name": "Eric"
    }
]
{% endhighlight %}

</section>