---
layout: tutorial
title: Views
permalink: /tutorial/views/
---

In the previous chapter we have worked with documents: we learned how to create, save, read and update a single document. 
Before we talk about querying documents, you need to introduce few, important, concepts.
I must warn you, since you're probably accustomed to the old Standard Query Language (SQL), please open your mind, because 
CouchDB is a different animal. SQL doesn't exist at all in the world of CouchDB. If you remember, we have spent some 
words in the introduction chapter about views.

A view is a persistent index, which you then query to find data. Like any other index, a view is related to a single 
database. A view is consisting of a mandatory map function and an optional reduce function. These functions, combined, 
take the form of a MapReduce program.
 
Imagine CouchDB like a big repository for all your documents. A document doesn't have any structure: you might have in 
the same database invoices, users, orders, articles or whatever, CouchDB doesn't really care. To keep track of all the invoices 
(or the orders), people use catalogs. A catalog can be either an index of all documents of a very specific type or with 
certain characteristics. A view is, neither more nor less, a catalog, that CouchDB builds upon your requests. To be precise,
CouchDB designates a Query Server to carry out the task.

A view is stored into a design document. Many views can be part of the same design document. For example, we might have
a design document called `books` including both `novel`, `psychology` and `history` views.
CouchDB doesn't update its views when you insert or update a document, but only when you query them (before or after 
the query, it's up to the application requirements). Just like a librarian catalogs a new book at the time someone ask for 
[Nineteen Eighty-Four](http://en.wikipedia.org/wiki/Nineteen_Eighty-Four), CouchDB updates 
`novel`, `psychology` and `history` views, and finally query `novel` searching for the required novel.

CouchDB uses a technique called MapReduce to generate views according to arbitrary criteria, defined by your application. 
Queries can then look up a range of rows from a view, and either use the rows' keys and values directly or get the 
documents they came from.
The main component of a view (other than its name) is its map function. We have already seen you can write functions in 
many languages, but defaults are JavaScript and Erlang. If I recall, CouchDB uses Mozilla SpiderMonkey as JavaScript 
engine, which is pretty fast. EoC provides its own implementation of a PHP Query Server. Many other languages are 
supported through third parties.

## Map

A map function takes a document as input, and emits (outputs) any number of key/value pairs to be indexed. The view 
generates a complete index by calling the map function on every document in the database, and adding each emitted 
key/value pair to the index, sorted by key.
In our previous example, the "Nineteen Eighty-Four" book is mapped against all the views. The resulting indexex are 
persistent, and updated incrementally as documents change.
Keep in mind that a view is not a query, it’s an index. Views are persistent, and need to be updated (incrementally) 
whenever documents change, so having large numbers of them can be expensive. Instead, it’s better to have a smaller 
number of views that can be queried in interesting ways.

## Reduce

A view may also have a reduce function. If present, it can be used during queries to combine multiple rows into one. 
It can be used to compute aggregate values like totals or averages, or to group rows by common criteria (like collecting 
all the authors in a book collection).
CouchDB comes with 3 built-in reduce functions:

- `_count`: counts the number of emitted values;
- `_sum`: sums all the emitted values, which must be numbers;
- `_stats`: calculates some numerical statistics on your emitted values, which must be numbers.

These functions are fast, since they are written in Erlang.

## Adding MapReduce functions

In the example below we created a design document `books` and we added a view called `novel`. The view uses PHP as language 
for both map and reduce functions, since PHP is the EoC default language. The map function uses 
the [nowdoc syntax](http://php.net/manual/en/language.types.string.php#language.types.string.syntax.nowdoc) to make the 
code legible.

{% highlight php %}
<?php

namespace MyPress;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username', 'password'));
$couch->selectDb('database_name');

$doc = DesignDoc::create('books');

$handler = new ViewHandler("novel");
$handler->mapFn = <<<'MAP'
function($doc) use ($emit) {
  if ($doc->type == 'book' && $doc->category == 'novel')
    $emit($doc->category, $doc->id);
};
MAP;

$handler->useBuiltInReduceFnCount();

$doc->addHandler($handler);

$this->couch->saveDoc($doc);
{% endhighlight %}

Or if you prefer you can chose JavaScript as language for your map and reduce functions. Just remember 
to specify it during the handler creation.

{% highlight php %}
<?php

// The equivalent map implementation in JavaScript.
$map = <<<'MAP'
function(doc) {
  if (doc.type === 'book' && doc.category === 'novel')
    emit(doc.category, doc.id);
}
MAP;

// Implementation of the _count native function in JavaScript.
$reduce = <<<'REDUCE'
function(keys, values, rereduce) {
  if (rereduce) {
    return sum(values);
  } else {
    return values.length;
  }
}
REDUCE;

$handler = new ViewHandler("novel", "javascript"); // Force the use of the JavaScript language.
$handler->mapFn = $map;
$handler->reduce = $reduce;

$doc->addHandler($handler);
{% endhighlight %}

It's important to note that, in case of PHP, the closure uses the variable `$emit`, 
so that you can call `$emit($doc->category, $doc->id)`. The EoC Server, internally, assigns to the `$emit` variable an 
anonymous function that does the mapping.

You must also remember the `;` at the end of the closure, since it's mandatory. EoC Server makes a double check to 
assure that your map and reduce functions are properly written: first it checks the declaration using a regular 
expression, then it uses a lint program to check if your function is a valid PHP code.