---
layout: docs
title: Working with documents
permalink: /docs/documents/
---

CouchDB is a document database: it works with documents. It's not like MySQL where you have tables, it's not even like 
any key/value store. CouchDB is a database capable to store and retrieves documents. A document is a JSON representation 
of a data set, simply as that. Of course in PHP we don't have the notion of document, instead we have objects, instances 
of classes. That's why the EoC library provides a new class called `Doc`. Any instance of this powerful class 
actually of any subclass) encapsulates a JSON representation of the object itself and can be stored into CouchDB, like a 
document. The client can't work with any object, it requires the object is an instance of a class that inherits from 
`Doc` (or uses a trait, or implements a special interface). This is the most important concept you must keep in 
mind using EoC: to be stored, an object must be an instance of a `Doc` subclass. Well, we are talking about persistent 
objects here, but I'm sure you put that all together yourself, did you? Is cool, isn't it? :-)

### The `Doc` class

You have the original idea to create a blog platform like WordPress, and you have chosen to do that using PHP and 
CouchDB, because you desire to try something new, you're tired about SQL, ORM and all that stuff. Your application must 
handle posts, users and comments, in the most simple way possible. Ah, almost forgot, you decided to call it MyPress.
You may start creating these classes:

{% highlight php %}
<?php

namespace MyPress;

class User {
}

class Article {
}

class Comment {
}
{% endhighlight %}

Let's add persistence:

{% highlight php %}
<?php

namespace MyPress;

class User extends Doc {
}

// There are cases you can't inherits from Doc, because Article already extends another class, so use a the trait TDoc. 
class Article extends Post {
  use TDoc;
}

// You have also the ability to implements the IDoc interface yourself.
class Comment implements IDoc {

  /**
   * @brief Sets the object type.
   * @param[in] string $value Usually the class name purged of his namespace.
   */
  function setType($value) {
    ...
  }


  /**
   * @brief Returns `true` if your document class already defines his type internally, `false` otherwise.
   * @details Sometime happens you have two classes with the same name but located under different namespaces. In case,
   * you should provide a type yourself for at least one of these classes, to avoid Couch::saveDoc() using the same type
   * for both. Default implementation should return `false`.
   * @return bool
   */
  function hasType() {
    ...
  }

  /**
   * @brief Gets the document identifier.
   * @return string
   */
  function getId() {
    ...
  }


  /**
   * @brief Returns `true` if the document has an identifier, `false` otherwise.
   * @return bool
   */
  function issetId() {
    ...
  }


  /**
   * @brief Sets the document identifier. Mandatory and immutable.
   */
  function setId($value) {
    ...
  }


  /**
   * @brief Unset the document identifier.
   */
  function unsetId() {
    ...
  }


  /**
   * @brief Sets the full name space class name into the the provided metadata into the metadata array.
   * @details The method Couch.getDoc will use this to create an object of the same class you previously stored using
   * Couch::saveDoc() method.
   * @param[in] string $value The full namespace class name, like returned from get_class() function.
   */
  function setClass($value) {
    ...
  }


  /**
   * @brief Gets the document path.
   * @details Returns an empty string for standard document, `_local/` for local document and `_design/` for
   * design document.
   * @return string
   */
  function getPath() {
    ...
  }


  /**
   * @brief Returns the document representation as a JSON object.
   * @return JSON object
   */
  function asJson() {
    ...
  }


  /**
   * @brief Returns the document representation as an associative array.
   * @return array An associative array
   */
  public function asArray() {
    ...
  }
}
{% endhighlight %}