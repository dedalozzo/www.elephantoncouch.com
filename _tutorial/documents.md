---
layout: tutorial
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

CouchDB uses a data structure called a B-tree to index its documents and views. In particular CouchDB uses an append-only
log file to write documents. Since the log file is append-only CouchDB keeps track of the change history of every document, 
as a series of revisions, implementing [MVCC](http://en.wikipedia.org/wiki/Multiversion_concurrency_control).
A document is never really updated or deleted. This can be achieved only through a process called compaction.
B-trees are used to store the main database file as well as the views. Both, the database itself and the views are B-trees.
The pusrpose a the Multi-Version Concurrency Control is not to be able to access old data, but rather to assist the 
replicator in deciding what data to synchronize and what documents have conflicts. Every time a document is created or 
updated, it is assigned a new unique revision ID. The IDs of past revisions are available, and the contents of past 
revisions may be available, but only if the revision was created locally and the database has not yet been compacted.

To summarize, a document has the following attributes:

- a document ID;
- a current revision ID (which changes every time the document is updated);
- a history of past revision IDs;
- a body in the form of a JSON object, i.e. a set of key/value pairs;
- zero or more named binary attachments.

## A single coherent whole

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

Now, let's suppose you have the original idea to create a platform to manage a library catalog, and you have chosen to do  
that using PHP and CouchDB, because you desire to try something new, you're tired about SQL, ORM and all that stuff. 
Your application must handle books, authors and, of course, categories, in the most simple way possible. Ah, almost forgot, 
you decided to call it Babylon (we need a name to call the namespace). You may start creating the `Author` class:

A class to represent an author.

{% highlight php %}
<?php

namespace Babylon;

class Author {
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

namespace Babylon;

use EoC\Doc\Doc;

class Author extends Doc {
}
{% endhighlight %}

### Implementing the IDoc interface using the TDoc trait

Since `Author` inherits from `Person`, and PHP doesn't support multiple inheritance, let's implements IDoc interface, using 
TDoc trait.

{% highlight php %}
<?php

namespace Babylon;

use EoC\Doc\IDoc;
use EoC\Doc\TDoc;

class Author extends Person implements IDoc {
  use TDoc;
}
{% endhighlight %}

## Document's properties

Our class still doesn't have any property, apart `id` and `rev`, which are inhereted from `Doc`.
At least, an author will have a first name and a last name, so let's add getters and setters for these properties. 
It's important to note here, we don't use any protected members, on the contrary we relay on the `meta` protected array. 
Elephant on Couch just care about this array. Every single key/value inside the array will be stored, 
while the other private or protected members are not taken into account, never.

{% highlight php %}
<?php

namespace Babylon;

use EoC\Doc\Doc;

// An author.
class Author extends Doc {

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
  
  // Add many other information...

}

// A book.
class Book extends Doc {

  public function getIsbn() {
    return $this->meta['isbn'];
  }

  public function issetIsbn() {
    return isset($this->meta['isbn']);
  }

  public function setIsbn($value) {
    $this->meta['isbn'] = $value;
  }

  public function unsetIsbn() {
    if ($this->isMetadataPresent('isbn'))
      unset($this->meta['isbn']);
  }

  public function getTitle() {
    return $this->meta['title'];
  }

  public function issetTitle() {
    return isset($this->meta['title']);
  }

  public function setTitle($value) {
    $this->meta['title'] = $value;
  }

  public function unsetTitle() {
    if ($this->isMetadataPresent('title'))
      unset($this->meta['title']);
  }
  
  // Add many other information...

}

// A category: novel, psychology, etc.
class Category {

  public function getName() {
    return $this->meta['name'];
  }

  public function issetName() {
    return isset($this->meta['name']);
  }

  public function setName($value) {
    $this->meta['name'] = $value;
  }

  public function unsetName() {
    if ($this->isMetadataPresent('name'))
      unset($this->meta['name']);
  }

  public function getDescription() {
    return $this->meta['description'];
  }

  public function issetDescription() {
    return isset($this->meta['description']);
  }

  public function setDescription($value) {
    $this->meta['description'] = $value;
  }

  public function unsetDescription() {
    if ($this->isMetadataPresent('description'))
      unset($this->meta['description']);
  }
  
  // Add many other information...

}

{% endhighlight %}

You may think the code is a bit verbose, but EoC strongly encourages best practises. You don't need to create all the 
above methods, EoC in fact doesn't care about getters and setters. Inside a view (as we'll see later), you can 
access the members of an object using the dereference operator (`->` in PHP and `.` in JavaScript), even if you don't 
declare getters and setters for the class properties. 
Since `firstName` and `lastName` are properties, you probably need getters and setters anyway, but they can have different 
names.  
Said this, which is very important, `Doc` comes with a feature to handle properties _a la_ C#, helping to keep the 
application design clean. That means, once you implement getters and setters for the `Author` class' properties, you can 
refer to these properties using the dereference operator, like the member were public. For example you can write:

{% highlight php %}
<?php

$author = new Author();
$author->firstName = 'Fyodor';
$author->lastName = 'Dostoyevsky'; 

}

This is made possible because `Doc` uses a trait that implements some [magic methods](http://php.net/manual/en/language.oop5.magic.php).

In a minute we have created classes, whose instances are now persistent.

## Creating, Reading, Updating and Deleting documents (CRUD)

EoC supports the typical database "CRUD" operations on documents: Create, Read, Update, Delete.

### Creating documents

Let's see how to create an author and save it into CouchDB.

{% highlight php %}
<?php

namespace Babylon;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username', 'password'));
$couch->selectDb('database_name');

$author = new Author();
$author->firstName = 'George';
$author->lastName = 'Orwell';

$couch->saveDoc($author);
{% endhighlight %}

As you can see in the above example, `firstName` and `lastName` behave like public properties, even if they aren't such. 
The `saveDoc()` method is all we need to store the author into CouchDB.

You must note we didn't provide an ID for the author, because EoC will generate one for us. The ID is somewhat important 
because, as we'll see later, it is the only way to retrieve a document from the database. We strongly suggest to use 
a universal unique identifier [UUID](http://en.wikipedia.org/wiki/Universally_unique_identifier), but are free to 
assign whater you want, just remember the ID **must be** unique. In our example we use `77d09b72d0cdbfd73255a9a158000dcf`.  

{% highlight php %}
<?php

namespace Babylon;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username', 'password'));
$couch->selectDb('database_name');

$author = new Author();
$author->id = '77d09b72d0cdbfd73255a9a158000dcf';
$author->firstName = 'Chuck';
$author->lastName = 'Palahniuk';

$couch->saveDoc($author);
{% endhighlight %}

For your convenience EoC provides a static method to generate a UUID. 

{% highlight php %}
<?php

use EoC\Generator\UUID;

$uuid = UUID::generate(UUID::UUID_RANDOM, UUID::FMT_STRING);
{% endhighlight %}

### Reading documents

In the example above we created an author instance that we stored into CouchDB. Let's see how we can retrieve the same 
author from the database, using `77d09b72d0cdbfd73255a9a158000dcf`, the ID we previously assigned.

{% highlight php %}
<?php

namespace Babylon;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username', 'password'));
$couch->selectDb('database_name');

$author = $couch->getDoc(Couch::STD_DOC_PATH, '77d09b72d0cdbfd73255a9a158000dcf');

// This will print Peter Palahniuk.
echo $author->firstName . ' ' . $author->lastName;
{% endhighlight %}

It's important to note the constant `Couch::STD_DOC_PATH`, which is equivalent to an empty string. Since both documents, 
local and design documents resides on the same database, they use different paths. Standard documents don't have one, design 
documents are prefixed by `_design/`, and local documents by `_local/`. You don't have to remember them, just use the 
following class constants: `Couch::STD_DOC_PATH`, `Couch::LOCAL_DOC_PATH`, `Couch::DESIGN_DOC_PATH`.

### Updating documents

As you may have noticed Palahniuk's name is Chuck, not Peter. Let's see how to modify the name of the previously created 
author from Chuck to Peter.

{% highlight php %}
<?php

namespace Babylon;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username', 'password'));
$couch->selectDb('database_name');

$author = $couch->getDoc(Couch::STD_DOC_PATH, '77d09b72d0cdbfd73255a9a158000dcf');
$author->firstName = 'Peter';

$couch->saveDoc($author);
{% endhighlight %}

That's all, updating a document is easy like to create a new one, we have just to provide its ID.

### Deleting documents

To delete a document is necessary know three things: the document path, the ID and finally its revision.
There are two ways to delete a document. The first one is simply mark the document as deleted and save it. The second 
one implies a call to the `deleteDoc()` method.
Let's now delete Chuck Palahniuk from the database. 

{% highlight php %}
<?php

namespace Babylon;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username', 'password'));
$couch->selectDb('database_name');

$author = $couch->getDoc(Couch::STD_DOC_PATH, '77d09b72d0cdbfd73255a9a158000dcf');
$author->delete();

$couch->saveDoc($author);
{% endhighlight %}

Alternatively:

{% highlight php %}
<?php

namespace Babylon;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username', 'password'));
$couch->selectDb('database_name');

$author = $couch->getDoc(Couch::STD_DOC_PATH, '77d09b72d0cdbfd73255a9a158000dcf');

$couch->deleteDoc(Couch::STD_DOC_PATH, $author->id, $author->rev);
{% endhighlight %}

<div class="note info">
  <h5>Important note on deleted documents</h5>
  <p>
    It's important to note that the document isn't really deleted, it will exist until the next compaction.
  </p>
</div>

## Revisions

Aside the more common master/slave configuration you may setup a [multi-master replication](http://en.wikipedia.org/wiki/Multi-master_replication#Apache_CouchDB). To keep the masters in sync,
CouchDB uses a simple HTTP-based replication system that relays on MVCC. As stated in this chapter's introduction, 
CouchDB's B-Tree variant implements the multiversion concurrency control, saving a new document's revision as consequence 
of an update, an insert or a deletion. These revisions are used, essentially, to resolve [conflicts](http://guide.couchdb.org/draft/conflicts.html) arisen 
during replication. The revision ID, formerly another UUID, is stored into a reserved document's attribute called `_rev`.
The attribute is accessible through the already quoted property `rev`.
To update an existing document, you must always provide its current revision. CouchDB prevents the overwriting of an
existent document if a new version of the same document is saved in the meantime. 
In case of a conflict, the application must fetch the last version, reconcile any changes, and try to make a new update.
At first glance you might be led to believe CouchDB is a version control system, but it's not. The old revisions, in fact,
are periodically deleted, this time for real, during compaction, and anyway they’re never replicated. They only exist 
to serve the replication cause.

### Tombstones

Tombstones are revisions of deleted documents. CouchDB documents are never really deleted until compaction, they still 
exist as deleted revisions. Tombstones exist for a reason, to synchronize changes in a master/slave or in a multi-master 
configuration. `_deleted`is the special attribute devoted to determine if a revision is a tombstone or not. `Doc` provides 
the method `isDeleted()`to check if the current revision has been deleted.