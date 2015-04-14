---
layout: tutorial
title: Queries
permalink: /tutorial/queries/
---

A query is the action of looking up results from a view's index. In Couchbase Lite, queries are objects of the Query class. To perform a query you create one of these, customize its properties (such as the key range or the maximum number of rows) and then run it. The result is a QueryEnumerator, which provides a list of QueryRow objects, each one describing one row from the view's index.

There's also a special type of query called an all-docs query. This type of query isn't associated with any view; or rather, you can think of it as querying an imaginary view that contains one row for every document in the database. You use an all-docs query to find all the documents in the database, or the documents with keys in a specific range, or even the documents with a specific set of keys. It can also be used to find documents with conflicts.

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


### Querying key ranges

There are some subtleties to working with key ranges (startKey and endKey.) The first is that if you reverse the order of keys, by setting the reverse property, then the startKey needs to be greater than the endKey. That's the reason they're named start and end, rather than min and max. In the following example, note that the key range starts at 100 and ends at 90; if we'd done it the other way around, we'd have gotten an empty result set.

Objective-C	Swift	Java	Android	C#
// Set up a query for the highest-rated movies:
Query query = database.getView("postsByDate").createQuery();
query.setDescending(true);
query.setStartKey(new Integer(100));
query.setEndKey(new Integer(90));
        
Second is the handling of compound (array) keys. When a view's keys are arrays, it's very common to want to query all the rows that have a specific value (or value range) for the first element. The start key is just a one-element array with that value in it, but it's not obvious what the end key should be. What works is an array that's like the starting key but with a second object appended that's greater than any possible value. For example, if the start key is (in JSON) ["red"] then the end key could be ["red", "ZZZZ"] ... because none of the possible second items could be greater than "ZZZZ", right? Unfortunately this has obvious problems. The correct stop value to use turns out to be an empty object/dictionary, {}, making the end key ["red", {}]. This works because the sort order in views puts dictionaries last.

### Reducing

If the view has a reduce function, it will be run by default when you query the view. This means that all rows of the output will be aggregated into a single row with no key, whose value is the output of the reduce function. (See the View documentation for a full description of what reduce functions do.)

(It's important to realize that the reduce function runs on the rows that would be output, not all the rows in the view. So if you set the startKey and/or endKey, the reduce function runs only on the rows in that key range.)

If you don't want the reduce function to be used, set the query's mapOnly property to true. This gives you the flexibility to use a single view for both detailed results and statistics. For example, adding a typical row-count reduce function to a view lets you get the full results (with mapOnly=true) or just the number of rows (with mapOnly=false).


### Grouping by key

The groupLevel property of a query allows you to collapse together (aggregate) rows with the same keys or key prefixes. And you can compute aggregated statistics of the grouped-together rows by using a reduce function. One very powerful use of grouping is to take a view whose keys are arrays representing a hierarchy — like [genre, artist, album, track] for a music library — and query a single level of the hierarchy for use in a navigation UI.

In general, groupLevel requires that the keys be arrays; rows with other types of keys will be ignored. When the groupLevel is n, the query combines rows that have equal values in the first n items of the key into a single row whose key is the n-item common prefix.

groupLevel=1 is slightly different in that it supports non-array keys: it compares them for equality. In other words, if a view's keys are strings or numbers, a query with groupLevel=1 will return a row for each unique key in the index.

We've talked about the keys of grouped query rows, but what are the values? The value property of each row will be the result of running the view's reduce function over all the rows that were aggregated; or if the view has no reduce function, there's no value. (See the View documentation for information on reduce functions.)

Here's an interesting example. We have a database of the user's music library, and a view containing a row for every audio track, with key of the form [genre, artist, album, trackname] and value being the track's duration in seconds. The view has a reduce function that simply totals the input values. The user's drilled down into the genre "Mope-Rock", then artist "Radiohead", and now we want to display the albums by this artist, showing each album's running time.