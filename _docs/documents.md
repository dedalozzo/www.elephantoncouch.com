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

## The Doc class

You have the original idea to create a blog platform like WordPress, and you have chosen to do that using PHP and 
CouchDB, because you desire to try something new, you're tired about SQL, ORM and all that stuff. Your application must 
handle posts, users and comments, in the most simple way possible. Ah, almost forgot, you decided to call it MyPress.
You may start creating the `User class:

A class to represent a user.

{% highlight php %}
<?php

namespace MyPress;

class User {
}
{% endhighlight %}

## Persistence in a breeze

There are two ways for adding persistence to the above classe. The most simple one, that should normally be used, is to 
inherit every class from the superclass Doc. Sometimes you have to deal with the fact that PHP doesn't support multiple 
inheritance: this happens when a class, having already an ancestor, can't extend Doc. To handle a situation like this,
we have created a trait, called TDoc, which implements every single method of the IDoc interface. That's all you need.

### Inherit from Doc

This is the most simple case, just extends Doc class.

{% highlight php %}
<?php

namespace MyPress;

use EoC\Doc\Doc;

class User extends Doc {
}
{% endhighlight %}

### Or implement the IDoc interface using the TDoc trait

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

### Add some properties

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

### Save our objects

Let's see how to create an user and save him to CouchDB.

{% highlight php %}
<?php

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