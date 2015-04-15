---
layout: tutorial
title: Queries
permalink: /tutorial/queries/
---

Queries are the primary mechanism for retrieving a result set from a view. The result of a query is an instance of  
`QueryResult`, a class that implements the [IteratorAggregate](http://php.net/manual/en/class.iteratoraggregate.php), 
[Countable](http://php.net/manual/en/class.countable.php) and [ArrayAccess](http://php.net/manual/en/class.arrayaccess.php) 
interfaces, so you can use the result set as an array.

Elephant on Couch provides three methods to query information from a database, but `queryView()` is all you need on a 
daily basis.
`queryTempView()` is there for debugging purpose, since the view doesn't exist and its index is generated from scratch 
at the time of the query. I recommend you don't use it, ever, neither during the development phase.

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>queryAllDocs()</code></p></td>
      <td><p>Returns a built-in view of all documents in this database. If keys are specified returns only certain rows.</p></td>
    </tr>
    <tr>
      <td><p><code>queryView()</code></p></td>
      <td><p>Executes the given view and returns the result.</p></td>
    </tr>
    <tr>
      <td><p><code>queryTempView()</code></p></td>
      <td><p>Executes the given view, both map and reduce functions, for all documents and returns the result.</p></td>
    </tr>
  </tbody>
</table>
</div>

Let's see how to query all the novels.

{% highlight php %}
<?php

namespace MyPress;

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username', 'password'));
$couch->selectDb('database_name');

$opts = new ViewQueryOpts();
$opts->doNotReduce();

$result = $couch->queryView("books", "novel", $keys, $opts);

$this->view->setVar('usersCount', $result->getTotalRows());

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



Creating and configuring queries

Query objects are created by a View's createQuery method, and by a Database's createAllDocumentsQuery method. In its default state a Query object will return every row of the index, in increasing order by key. But there are several properties you can configure to change this, before you run the query. Here are the most basic and common ones:


startKey: the key to start at. The default value, null, means to start from the beginning.
endKey: the last key to return. The default value, null, means to continue to the end.
descending: If set to true, the keys will be returned in reverse order. (This also reverses the meanings of the startKey and endKey properties, since the query will now start at the highest keys and end at lower ones!)
limit: If nonzero, this is the maximum number of rows that will be returned.
skip: If nonzero, this many rows will be skipped (starting from the startKey if any.)
Some more advanced properties that aren't used as often:

keys: If provided, the query will fetch only the rows with the given keys. (and startKey and endKey will be ignored.)
startKeyDocID: If multiple index rows match the startKey, this property specifies that the result should start from the one(s) emitted by the document with this ID, if any. (Useful if the view contains multiple identical keys, making .startKey ambiguous.)
endKeyDocID: If multiple index rows match the endKey, this property specifies that the result should end with from the one(s) emitted by the document with this ID, if any. (Useful if the view contains multiple identical keys, making .startKey ambiguous.)
indexUpdateMode: Changes the behavior of index updating. By default the index will be updated if necessary before the query runs. You can choose to skip this (and get possibly-stale results), with the option of also starting an asynchronous background update of the index.
There are other advanced properties that only apply to reducing and grouping:

mapOnly: If set to true, prevents the reduce function from being run, so you get all of the index rows instead of an aggregate. Has no effect if the view has no reduce function.
groupLevel: If greater than zero, enables grouping of rows. The value specifies the number of items in the value array that will be grouped.


### Key ranges

Those methods are used to return documents in a key range.
There are some subtleties to working with key ranges (startKey and endKey.) The first is that if you reverse the order of keys, by setting the reverse property, then the startKey needs to be greater than the endKey. That's the reason they're named start and end, rather than min and max. In the following example, note that the key range starts at 100 and ends at 90; if we'd done it the other way around, we'd have gotten an empty result set.

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>setStartKey()</code></p></td>
      <td><p>Defines the first key to be included in the range.</p></td>
    </tr>
    <tr>
      <td><p><code>setEndKey()</code></p></td>
      <td><p>Defines the last key to be included in the range.</p></td>
    </tr>
  </tbody>
</table>
</div>

### First and last document identifiers

First and last documents to be included in the output.
If you expect to have multiple documents emit identical keys, you'll need to use `startDocId` in
addition to `startKey` to paginate correctly. The reason is that `startKey` alone will no longer be
sufficient to uniquely identify a row. Those parameters are useless if you don't provide a `startKey`. In fact,
CouchDB will first look at the `startKey` parameter, then it will use the `startDocId` parameter to further
redefine the beginning of the range if multiple potential staring rows have the same key but different document IDs.
Same thing for the `endDocId`.
 
<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>setStartDocId()</code></p></td>
      <td><p>Sets the ID of the document with which to start the range.</p></td>
    </tr>
    <tr>
      <td><p><code>setEndDocId()</code></p></td>
      <td><p>Sets the ID of the document with which to end the range.</p></td>
    </tr>
  </tbody>
</table>
</div>

### Reverse order of results

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>reverseOrderOfResults()</code></p></td>
      <td>
        <p>
          Reverses order of results.
          Note that the descending option is applied before any key filtering, so you may need to swap the values
          of the start key and end key options to get the expected results.
        </p>
      </td>
    </tr>
  </tbody>
</table>
</div>

### Limit the number of results

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>setLimit($value)</code></p></td>
      <td>
        <p>
          Restricts the number of results.
        </p>
      </td>
    </tr>
  </tbody>
</table>
</div>
 
### View refresh controls

CouchDB defaults to regenerating views the first time they are accessed. This behavior is preferable in most cases
as it optimizes the resource utilization on the database server. On the other hand, in some situations the benefit
of always having fast and updated views far outweigh the cost of regenerating them every time the database server
receives updates. You can chose CouchDB behaviour using one of the following methods.
Those methods essentially tell CouchDB that if a reference to the view index is available in memory (ie, if
the view has been queried at least once since couch was started), go ahead and use it, even if it may be out of
date. The result is that for a highly trafficked view, end users can see lower latency, although they may not get
the latest data. However, if there is no view index pointer in memory, the behavior with this option is that same
as the behavior without the option.

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>doNotRefreshView()</code></p></td>
      <td>
        <p>
          CouchDB will not refresh the view, even if it's stalled.
          This is useful in case you chose to not refresh a view when a query is performed on it, because you want
          faster results. Remember, in case, to use an updater script that calls the views periodically, for example using a
          cron. You can find the implementation in Ruby or Python in the document
          [Update views on document save](http://wiki.apache.org/couchdb/Regenerating_views_on_update).
        </p>
      </td>
    </tr>
    <tr>
      <td><p><code>queryThenRefreshView()</code></p></td>
      <td><p>CouchDB will update the view after the query's result is returned.</p></td>
    </tr>
  </tbody>
</table>
</div>

        
        
        
Second is the handling of compound (array) keys. When a view's keys are arrays, it's very common to want to query all the rows that have a specific value (or value range) for the first element. The start key is just a one-element array with that value in it, but it's not obvious what the end key should be. What works is an array that's like the starting key but with a second object appended that's greater than any possible value. For example, if the start key is (in JSON) ["red"] then the end key could be ["red", "ZZZZ"] ... because none of the possible second items could be greater than "ZZZZ", right? Unfortunately this has obvious problems. The correct stop value to use turns out to be an empty object/dictionary, {}, making the end key ["red", {}]. This works because the sort order in views puts dictionaries last.

### Reducing

If the view has a reduce function, it will be run by default when you query the view. This means that all rows of the 
output will be aggregated into a single row with no key, whose value is the output of the reduce function.
If you don't want the reduce function to be used, call the method `doNotReduce()`. This gives you the flexibility to use 
a single view for both detailed results and statistics. For example, adding a typical row-count reduce function to a 
view lets you get the full results. Remember that you can't obtain both the results and the count with a single query. 
If you need both, you have to run the query twice!

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>reduce()</code></p></td>
      <td>
        <p>
          Calls the reduce function is defined.
        </p>
      </td>
    </tr>
    <tr>
      <td><p><code>doNotReduce()</code></p></td>
      <td>
        <p>
          Even if a reduce function is defined for the view, doesn't call it.
          If a view contains both a map and reduce function, querying that view will by default return the result
          of the reduce function. To avoid this behaviour you must call this method.
        </p>
      </td>
    </tr>
  </tbody>
</table>
</div>

## Inclusions and exclusions

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>includeDocs()</code></p></td>
      <td>
        <p>
          Automatically fetches and includes full documents.
          However, the user should keep in mind that there is a race condition when using this option. It is
          possible that between reading the view data and fetching the corresponding document that the document has changed.
          You can call this method only if the view doesn't contain a reduce function.          
        </p>
      </td>
    </tr>
    <tr>
      <td><p><code>excludeResults()</code></p></td>
      <td>
        <p>
          Don't get any data, but all meta-data for this View. The number of documents in this View for example.
        </p>
      </td>
    </tr>
    <tr>
      <td><p><code>excludeEndKey()</code></p></td>
      <td>
        <p>
          Tells CouchDB to not include end key in the result.
        </p>
      </td>
    </tr>
    <tr>
      <td><p><code>includeConflicts()</code></p></td>
      <td>
        <p>
          Includes conflict documents.
        </p>
      </td>
    </tr>
    <tr>
      <td><p><code>includeUpdateSeq()</code></p></td>
      <td>
        <p>
          Includes an `update_seq` value indicating which sequence id of the database the view reflects.
        </p>
      </td>
    </tr>
  </tbody>
</table>
</div>

### Grouping results

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>groupResults()</code></p></td>
      <td>
        <p>
          Results should be grouped.
          The group option controls whether the reduce function reduces to a set of distinct keys or to a single
          result row. This will run the rereduce procedure. This parameter makes sense only if a reduce function is defined
          for the view.          
        </p>
      </td>
    </tr>
    <tr>
      <td><p><code>setGroupLevel($value)</code></p></td>
      <td>
        <p>
          Level at which documents should be grouped.
          If your keys are JSON arrays, this parameter will specify how many elements in those arrays to use for
          grouping purposes. If your emitted keys are not JSON arrays this parameter's value will effectively be ignored.
          Allowed values: positive integers.
        </p>
      </td>
    </tr>
  </tbody>
</table>
</div>


The groupLevel property of a query allows you to collapse together (aggregate) rows with the same keys or key prefixes. And you can compute aggregated statistics of the grouped-together rows by using a reduce function. One very powerful use of grouping is to take a view whose keys are arrays representing a hierarchy — like [genre, artist, album, track] for a music library — and query a single level of the hierarchy for use in a navigation UI.

In general, groupLevel requires that the keys be arrays; rows with other types of keys will be ignored. When the groupLevel is n, the query combines rows that have equal values in the first n items of the key into a single row whose key is the n-item common prefix.

groupLevel=1 is slightly different in that it supports non-array keys: it compares them for equality. In other words, if a view's keys are strings or numbers, a query with groupLevel=1 will return a row for each unique key in the index.

We've talked about the keys of grouped query rows, but what are the values? The value property of each row will be the result of running the view's reduce function over all the rows that were aggregated; or if the view has no reduce function, there's no value. (See the View documentation for information on reduce functions.)

Here's an interesting example. We have a database of the user's music library, and a view containing a row for every audio track, with key of the form [genre, artist, album, trackname] and value being the track's duration in seconds. The view has a reduce function that simply totals the input values. The user's drilled down into the genre "Mope-Rock", then artist "Radiohead", and now we want to display the albums by this artist, showing each album's running time.

## Making joins

In SQL a join clause combines records from two or more tables, creating a single result set. As you know, CouchDB doesn't 
support joins, since it's not a relational database management system.
When using CouchDB, a join is a matter for the application to deal with. Of course, a join can't be done through a SQL 
statement, but it's still an easy task.
A great limitation I found during the development of EoC is that CouchDB returns only the rows if a match for a key is 
found. This behaviour precludes any LEFT or RIGHT join. To overcome this limitation EoC does a post operation on the 
result set, adding a new row for every key hasn't been matched.

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Method</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><p><code>includeMissingKeys()</code></p></td>
      <td>
        <p>
          Includes all the rows, even if a match for a key is not found.
        </p>
      </td>
    </tr>
  </tbody>
</table>
</div>

