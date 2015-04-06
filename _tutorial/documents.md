---
layout: docs
title: Documents
permalink: /tutorial/documents/
---

CouchDB is a document database: it works with documents. It's not like MySQL where you have tables, it's not even like 
any key/value store. CouchDB is a database capable to store and retrieves documents. A document is a JSON representation 
of a data set, simply as that. A document's body takes the form of a JSON object: a collection of key/value pairs where 
the values can be different types of data such as numbers, strings, arrays or even nested objects.
 
Every document is identified by a document ID, which can be automatically generated (as a UUID) or determined by the 
application; the only constraints are that it must be unique within the database, and it can't be changed.
 
In addition, a document may have attachments, named binary blobs that are useful for storing media files or 
other unstructured data. We'll talk about attachments in a a dedicated chapter.

## Document + object = a single coherent whole

As we know, the notion of document in PHP is missing, but we have objects, instances of classes. On the other hand, 
CouchDB doesn't know anything about objects, since it works with documents.

Here EoC comes into play providing a special class called `Doc`, aims to harmonise the systems and completely overhaul 
the concepts of document and object into a single coherent whole.
The encapsulation of a document inside an object is done, in fact, through a `meta` attribute, a typical PHP 
associative array. This attribute contains the object's metadata, or if you prefer the document's body.
 
The client can't work with any object, it requires the object is an instance of a class that inherits from 
`Doc` (or implements the IDoc interface). This is the most important concept you must keep in 
mind using EoC: to be stored, an object must be an instance of a `Doc` subclass.

EoC does all the dirty work under the hood, therefore you don't need to worry about nothing. Within the 
document itself, EoC stores the class name, including its full namespace, to recreate the object when you get 
back the document from the database.

Well, we are talking about persistent objects here, but I'm sure you put that all together yourself, didn't you?

## Persistent classes

Now, let's suppose you have the original idea to create a blog platform like WordPress, and you have chosen to do that 
using PHP and CouchDB, because you desire to try something new, you're tired about SQL, ORM and all that stuff. 
Your application must handle posts, users and comments, in the most simple way possible. Ah, almost forgot, you decided 
to call it MyPress (just to know how to call the namespace). You may start creating the `User` class:

A class to represent a user.

{% highlight php %}
<?php

namespace MyPress;

class User {
}
{% endhighlight %}

There are two ways for adding persistence to the above class. The most simple one, that should be normally used, is to 
inherit every class from the superclass Doc. Sometimes you have to deal with the fact that PHP doesn't support multiple 
inheritance: this happens when a class, having already an ancestor, can't extend Doc. To handle a situation like this,
EoC provides a trait, called TDoc, which implements every single method of the IDoc interface. That's all you need.

### Inheriting from Doc

This is the most simple case, just extends Doc class.

{% highlight php %}
<?php

namespace MyPress;

use EoC\Doc\Doc;

class User extends Doc {
}
{% endhighlight %}

### Implementing the IDoc interface using the TDoc trait

Since `User` inherits from `Person`, and PHP doesn't support multiple inheritance, let's implements IDoc interface, using 
TDoc trait.

{% highlight php %}
<?php

namespace MyPress;

use EoC\Doc\TDoc;

class User extends Person implements IDoc {
  use TDoc;
}
{% endhighlight %}

## Document's properties

Our class still doesn't have any property. At least, an user will have a first name and a last name, so let's add 
getters and setters for these properties. It's important to note here, we don't use any protected members, on the 
contrary methods relay on the `meta` array. Elephant on Couch just care about this array. Every single key/value inside 
the array will be stored, while the other private or protected members are not taken into account, never.

{% highlight php %}
<?php

namespace MyPress;

use EoC\Doc\Doc;

class User extends Doc {

  public function getFirstName() {
    return $this->meta['firstName'];
  }

  public function issetFirstName() {
    return isset($this->meta['firstName']);
  }

  public function setFirstName($value) {
    $this->meta['firstName'] = $value;
  }

  public function unsetFirstName() {
    if ($this->isMetadataPresent('firstName'))
      unset($this->meta['firstName']);
  }

  public function getLastName() {
    return $this->meta['lastName'];
  }

  public function issetLastName() {
    return isset($this->meta['lastName']);
  }

  public function setLastName($value) {
    $this->meta['lastName'] = $value;
  }

  public function unsetLastName() {
    if ($this->isMetadataPresent('lastName'))
      unset($this->meta['lastName']);
  }

}
{% endhighlight %}

Said this, which is very important, Doc comes with a feature to handle properties "a la" C#, helping to keep the 
application design clean. That means, once you implement getters and setters for the User properties, you are done. 

## Creating, Reading, Updating and Deleting documents (CRUD)

EoC supports the typical database "CRUD" operations on documents: Create, Read, Update, Delete.

### Creating documents

Let's see how to create an user and save him to CouchDB.

{% highlight php %}
<?php

namespace MyPress;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username','password'));
$couch->selectDb('database_name');

$user = new User();
$user->firstName = 'John';
$user->lastName = 'Smith';

$couch->saveDoc($user);
{% endhighlight %}

As you can see in the above example, `firstName` and `lastName` behave like public properties, even if they aren't such. 
The `saveDoc` method is all we need to store the user into CouchDB.

In a minute we have created a class, whose instances are now persistent.

### Reading documents

### Updating documents

### Deleting documents