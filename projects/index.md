---
layout: doc
title: Projects
navigation:
 - emfjson-jackson
 - emfjson-gwt
 - ecore.js
 - emfjson-mongo
 - emfjson-couchdb
---

A separate documentation is available for each emf-json project.

<div class="docs">
  {% for project in site.data.projects %}
  <div>
    <h2 id="{{ project.id }}">{{ project.id }}</h2>
    <p>{{ project.description }}</p>
    <div>
      <p>> <a href="{{ project.docs }}">Documentation</a></p>
      <p>> <a target="_blank" href="{{ project.source }}">Source</a></p>
    </div>
  </div>
  {% endfor %}
</div>
