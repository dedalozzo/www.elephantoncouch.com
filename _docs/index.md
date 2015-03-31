---
layout: docs
title: Welcome
permalink: /docs/home/
---

This site aims to be a comprehensive guide to Elephant on Couch. We’ll cover topics such
as configuring the EoC Server, writing views directly in PHP, querying them, creating persistent objects,
working with attachments, dealing with complex joins, and we'll give you some advice on participating in the future
development of EoC itself.

## So what is Elephant on Couch, exactly?

Elephant on Couch is a suite of PHP tools to interact with [CouchDB](http://couchdb.apache.org/). The main component is 
the [EoC Client](http://daringfireball.net/projects/markdown/) library, a 
[Composer](https://getcomposer.org/) package, that's really simple to include in your project, to query documents, 
store your objects, etc. Constructed upon the client, there is the [EoC CLI](http://daringfireball.net/projects/markdown/), 
which help you to deal with day-to-day tasks, without the need to know the complex cURL's syntax.
Least but not last, if you are not a JavaScript type, you can benefit of [EoC Server](http://daringfireball.net/projects/markdown/), 
a PHP implementation of a CouchDB [query server](http://docs.couchdb.org/en/latest/config/query-servers.html) 
(or view server), which allows the writing of your views in pure PHP.

## Helpful Hints

Throughout this guide there are a number of small-but-handy pieces of
information that can make using EoC easier, more interesting, and less
hazardous. Here’s what to look out for.

<div class="note">
  <h5>ProTips™ help you get more from Elephant on Couch</h5>
  <p>These are tips and tricks that will help you be a EoC wizard!</p>
</div>

<div class="note info">
  <h5>Notes are handy pieces of information</h5>
  <p>These are for the extra tidbits sometimes necessary to understand EoC.</p>
</div>

<div class="note warning">
  <h5>Warnings help you not blow things up</h5>
  <p>Be aware of these messages if you wish to avoid certain death.</p>
</div>

<div class="note unreleased">
  <h5>You'll see this by a feature that hasn't been released</h5>
  <p>Some pieces of this website are for future versions of EoC that are not yet released.</p>
</div>

If you come across anything along the way that we haven’t covered, or if you
know of a tip you think others would find handy, please [file an
issue]({{ site.repository }}/issues/new) and we’ll see about
including it in this guide.
