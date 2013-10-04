---
layout: default
title: "Upgrading to Polymer.dart from Web UI"
description: "Learn tips for upgrading your Web UI app to Polymer.dart."
has-permalinks: true
---

# {{ page.title }}

Here is a non-exhaustive list of tips for developers upgrading from
Web UI to [polymer.dart](/polymer-dart/).

Do you have other tips for upgrading? Please send us a
[pull request](https://github.com/dart-lang/dartlang.org)
for this page, or email your tips to
[web-ui@dartlang.org](https://groups.google.com/a/dartlang.org/forum/#!forum/web-ui).
Thanks in advance!

### Getting Started

* You *should* include `<script src="packages/polymer/boot.js"></script>`
  and **not** dart.js in your HTML file.

* The boot.js file **must** go into the `<head>` and *not* the `<body>`.
  (We may allow boot.js in the body later,
  see [issue 12388](https://code.google.com/p/dart/issues/detail?id=12388).)

* We recommend creating a component for your "application" HTML page.
  Polymer believes that everything is a component.
  - This enables Polymer-style event binding, which only works inside a
    polymer-element's template.
  - This also sets up data binding to your component's fields, and turns on
    [Polymer Expressions](http://pub.dartlang.org/packages/polymer_expressions)
    in bindings.

* You can place `<template>` elements on the main HTML page, but they will *not*
  be bound to the main library's scope. Polymer.dart does not support binding
  to a library scope, but you can instantiate a template by setting the
  [model](http://api.dartlang.org/docs/releases/latest/dart_html/Element.html#model)
  to a Dart object.
  
  The template of a polymer element *will* be instantiate automatically with
  itself as the model, providing convenient access to its fields, exactly like
  Web UI.

* To create an app that works when compiled to JavaScript, you need to first
  build it. See the
  [deploy_to_javascript](https://github.com/sethladd/dart-polymer-dart-examples/tree/master/web/deploy_to_javascript)
  and its `build.dart` file. Notice the `--deploy` argument.

### Custom Elements

* Use `<polymer-element>` instead of `<element>`.

* The `extends` attribute on polymer-element is optional. If you use it,
  you should use the form of `<div is="my-element">`. If you omit the
  `extends` attribute, you are safe to use `<my-element>`.

* The `constructor` attribute on polymer-element is no longer used.

* Polymer.dart does **not** support polymer.js's `noscript` attribute on
  polymer-element. All custom elements must have a Dart class (see the
  next item).

* Every custom element must have a Dart class. Use an empty Dart class
  if necessary.
  See [issue 12254](https://code.google.com/p/dart/issues/detail?id=12254).
  If you really don't want to create an empty class, use:

{% prettify dart %}
registerPolymerElement('my-element', () => new PolymerElement())
{% endprettify %}

* The Dart class for the custom element must _registered_.
  An easy way to register your class is to use the
  `@CustomTag('element-name')` annotation.
  Alternatively, you can register it manually by calling
  `registerPolymerElement`.

* You **must** call `super` in your `created` lifecycle callback.
  It is recommended to do this from `inserted` and `removed` as well, if you
  are inheriting from another custom element.

* Go through `shadowRoot` to find nodes inside of your custom element.

* The `apply-author-styles` attribute (which used to be on the
  `<polymer-element>` tag)
  is now retrieved as a getter property on the class for the custom element.
  e.g.:

    class MyElement extends PolymerElement {
       bool get applyAuthorStyles => true;
       // ...
    }

* Declarative event handing only works inside of a custom element.
  Also, instead of `on-click="doFoo()"`, drop the parens and use
  'on-click="doFoo"'.

### Data Binding

* Objects **must** be Observable to have changes detected. See the
  [observe](http://api.dartlang.org/docs/releases/latest/observe.html)
  library for more information.

* Data binding expressions are now
  [Polymer Expressions](http://pub.dartlang.org/packages/polymer_expressions)
  instead of Dart expressions. Polymer Expressions are a powerful data binding
  language which offers null safety and convenient filterting operations.

* Getters are no longer observable. Instead, use
  [bindProperty](http://api.dartlang.org/docs/releases/latest/observe.html#bindProperty) and
  [notifyProperty](http://api.dartlang.org/docs/releases/latest/observe.html#notifyProperty)
  in the `created` callback to let the system know that the computed getter has
  changed whenever its dependencies have changed. You can use other helpers from
  the observe library too, such as
  [PathObserver](http://api.dartlang.org/docs/releases/latest/observe/PathObserver.html)
  and [ListPathObserver](http://api.dartlang.org/docs/releases/latest/observe/ListPathObserver.html).

* When manually observing an object, the
  [PropertyChangeRecord](http://api.dartlang.org/docs/releases/latest/observe/PropertyChangeRecord.html)
  only has the field name, not the old and new value. You have to use mirrors to
  get the new value. This is a bug, see
  [issue 12075](https://code.google.com/p/dart/issues/detail?id=12075).

* The name of the modified
  [field](http://api.dartlang.org/docs/releases/latest/observe/PropertyChangeRecord.html#field)
  is now a Symbol instead of a String.

* Null is treated as false in `template if` expressions.
  Non-null and non-false values are treated as true.
  There are known issues with this, please track
  [issue 13044](https://code.google.com/p/dart/issues/detail?id=13044).

* The `iterate` attribute no longer exists on template, use `repeat` instead.

* Polymer.dart does not support the `instantiate` attribute on the template
  tag. To instantiate a template, simply bind a model to it, and ensure the
  template has a `bind` attribute.

  If you were using `instantiate` with a conditional, use `if`.

* All data binding expressions in Polymer.dart require `{{ }}`, including
  template `if` and `repeat`.
  - Old Web UI: `template instantiate="some boolean"`.
  - New Polymer.dart: `template if="{{some boolean}}"`.