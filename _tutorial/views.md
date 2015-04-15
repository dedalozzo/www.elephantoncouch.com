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

### Rules for the map function

The map function is called by the indexer to help generate an index, and it has to meet certain requirements, otherwise 
the index won't be consistent. It's important to understand some rules so you can create a proper map function, otherwise 
your queries can misbehave in strange ways.

- It must be a "pure" function: that means any time it's called with the same input, it must produce exactly the same output. In other words, it can't use any external state, just its input JSON.
- It can't have side effects: it shouldn't change any external state, because it's unpredictable when it's called or how often it's called or in what order documents are passed to it.
- It must be thread-safe: it may be called on a background thread belonging to the indexer, or even in parallel on several threads at once.

In particular, avoid these common mistakes:

Don't do anything that depends on the current date and time -- that breaks the first rule, since your function's output can change depending on the date/time it's called. Common mistakes include emitting a timestamp, emitting a person's age, or emitting only documents that have been modified in the past week.
Don't try to "parameterize" the map function by referring to an external variable whose value you change when querying. It won't work. People sometimes try this because they want to find various subsets of the data, like all the items of a particular color. Instead, emit all the values of that property, and use a key range in the query to pick out the rows with the specific value you want.
Don't make any assumptions about when the map function is called. That's an implementation detail of the indexer. (For example, it's not called every time a document changes.)
Avoid having the map function call out into complex external code. That code might change later on to be stateful or have side effects, breaking your map function.

### Keys and values

Both the key and value passed to emit can be any JSON-compatible objects: not just strings, but also numbers, booleans, arrays, dictionaries/maps, and the special JSON null object (which is distinct from a null/nil pointer.) In addition, the value emitted, but not the key, can be a null/nil pointer. (It's pretty common to not need a value in a view, in which case it's more efficient to not emit one.)

Keys are commonly strings, but it turns out that arrays are a very useful type of key as well. This is because of the way arrays are sorted: given two array keys, the first items are compared first, then if those match the second items are compared, and so on. That means that you can use array keys to establish multiple levels of sorting. If the map function emits keys of the form [lastname, firstname], then the index will be sorted by last name, and entries with the same last name will be sorted by first name, just as if you'd used ORDER BY lastname, firstname in SQL.

Here are the exact rules for sorting (collation) of keys. The most significant factor is the key's object type; keys of one type always sort before or after keys of a different type. This list gives the types in order, and states how objects of that type are compared:

null
false, true (in that order)
Numbers, in numeric order of course
Strings, case-insensitive. The exact ordering is specified by the Unicode Collation Algorithm. This is not the same as ASCII ordering, so the results might surprise you -- for example, all symbols, including "~", sort before alphanumeric characters.
Arrays, compared item-by-item as described above.
Maps/dictionaries, also compared item-by-item. Unfortunately the order of items is ambiguous (since JSON doesn't specify any ordering of keys, and most implementations use hash tables which randomize the order) so using these as keys isn't recommended.

### Map function design

When to emit a whole document as the value? In some places you'll see code that does something like emit(key, doc), i.e. emitting the document's entire body as the value. (Some people seem to do this by reflex whenever they don't have a specific value in mind.) It's not necessarily bad, but most of the time you shouldn't do it. The benefit is that, by having the document's properties right at hand when you process a query row, it can make querying a little bit faster (saving a trip to the database to load the document.) But the downside is that it makes the view index a lot larger, which can make querying slower. So whether it's a net gain or loss depends on the specific use case. We recommend that you just set the value to null if you don't need to emit any specific value.

Is it OK is the same key is emitted more than once? The index allows duplicate keys, whether emitted by the same document or different documents. A query will return all of those key/value pairs if they match. They'll be sorted by the ID of the document that was responsible for emitting them; if a doc emits the same key multiple times, the order is undefined.

When is the map function called? View indexes are updated on demand when queried. So after a document changes, the next query made to a view will cause that view's map function to be called on the doc's new contents, updating the view index. (But remember that you shouldn't write any code that makes assumptions about when map functions are called.)

If a document has conflicts, which conflicting revision gets indexed? The document's currentRevision, sometimes called the "winning" revision, is the one that you see in the API if you don't request a revision by ID.

## Reduce

A view may also have a reduce function. If present, it can be used during queries to combine multiple rows into one. 
It can be used to compute aggregate values like totals or averages, or to group rows by common criteria (like collecting 
all the authors in a book collection).
CouchDB comes with 3 built-in reduce functions:

- `_count`: counts the number of emitted values;
- `_sum`: sums all the emitted values, which must be numbers;
- `_stats`: calculates some numerical statistics on your emitted values, which must be numbers.

These functions are fast, since they are written in Erlang.

For efficiency, the key/value pairs are passed in as two parallel arrays. This reduce block just counts the number of values and returns that number as an object. We could query this view, with reduce enabled, and get the total number of phone numbers in the database. Or by specifying a key range we could find the number of phone numbers in that range, for example the number in a single area code.

Here's just the body of a reduce function that totals up numbers. (This function would belong in a different view, whose map function emitted numeric values.)

### Rereduce

The previous section ignored the boolean rereduce parameter that's passed to the reduce function. What's it for? Unfortunately, from your perspsective as a reduce-function-writer it's just there to make your job a bit harder. The reason it exists is because it's part of a major optimization that makes reducing more efficient for the query engine.

Think of a view with a hundred million rows in its index. To run a reduced query against the whole index (with no startKey or endKey) the database will have to read all hundred million keys and values into memory at once, so it can pass them all to your reduce function. That's a lot of overhead, and on a mobile device it's likely to crash your app.

Instead, the database will read the rows in chunks. It'll read some number of rows into memory, send them to your reduce function, release them from memory, then go on to the next rows. This scales very well, but now there's the problem of what to do with the multiple reduced values returned by your function. Reducing is supposed to produce one end result, not several! The answer is to reduce the list of reduced values -- to re-reduce.

The rereduce parameter is there to tell your reduce function that it's being called in this special re-reduce mode. When re-reducing there are no keys, and the values are the ones already returned by previous runs of the same reduce function. The function's job is, once again, to combine the values into a single value and return it.

Sometimes you can handle re-reduce mode exactly like reduce mode. The second reduce block shown above (the one that totals up the values) can do this. Since its input values are numbers, and its output is a number, the re-reduce is done the same way as the reduce, and it can just ignore the rereduce flag.

But sometimes re-reduce has to work differently, because the output of the reduce stage doesn't look like the indexed values. The first reduce example -- the one that just counts the rows -- is an example. To re-reduce a list of row counts, you can't just count them, you have to add them. Let's revisit that example and add proper support for re-reducing

### Rules for the reduce function

The reduce function has the same restrictions as the map function (see above): It must be a "pure" function that always produce the same output given the same input. It must not have side effects. And it must be thread-safe. In addition:

Its output should be no larger than its input. Usually this comes naturally. But it is legal to return an array or dictionary, and sometimes people have tried to make reduce functions that transform the input values without actually making them any smaller. The problem with this is that it scales badly, and as the size of the index grows, the indexer will eventually run out of memory and fail.

## Adding MapReduce functions

In the example below we created a design document `books` and we added a view called `novel`. The view uses PHP as language 
for both map and reduce functions, since PHP is the EoC default language. The map function uses 
the [nowdoc syntax](http://php.net/manual/en/language.types.string.php#language.types.string.syntax.nowdoc) to make the 
code legible.

{% highlight php %}
<?php

namespace Babylon;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username', 'password'));
$couch->selectDb('database_name');

$doc = DesignDoc::create('books');

$handler = new ViewHandler("novel");
$handler->mapFn = <<<'MAP'
function($doc) use ($emit) {
  if ($doc->type == 'book' && $doc->category == 'novel')
    $emit($doc->category, $doc->title);
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