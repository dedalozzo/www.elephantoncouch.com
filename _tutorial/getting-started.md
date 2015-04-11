---
layout: tutorial
title: Getting started
permalink: /tutorial/getting-started/
---

Once you have installed it, you're ready to use the client. In the example below has shown how to create a new 
connection with the server.

### Connect using the cURL adapter

{% highlight php %}
<?php

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\CurlAdapter('127.0.0.1:5984', 'username','password'));

$couch->selectDb('database_name');
{% endhighlight %}

### Connect using the native adapter

Alternatively, you can chose the `NativeAdapter`:

{% highlight php %}
<?php

use EoC\Couch;
use EoC\Adapter;

$couch = new Couch(new Adapter\NativeAdapter('127.0.0.1:5984', 'username', 'password'));
{% endhighlight %}

<div class="note">
  <h5>ProTip™: Implement your own adapter</h5>
  <p>
    As we said previously, if you are not quite happy with the provided adapters, you can implement the interface `IAdapter` 
    to create a new one.
  </p>
</div>

Now that you've finally managed to create a connection, you are ready to work with databases.

## Working with databases

A database is a physical file where CouchDB stores documents. EoC supports the typical CouchDB's database operations: 
select, create, delete, compact. In addition EoC supports replication, that we'll discuss in a dedicated chapter.

### Creating a database

{% highlight php %}
<?php

$couch->createDb('database_name');
{% endhighlight %}

### Selecting a database

{% highlight php %}
<?php

$couch->selectDb('database_name');
{% endhighlight %}

### Deleting a dabatase

{% highlight php %}
<?php

$couch->deleteDb('database_name');
{% endhighlight %}

Now that you've finally managed to create a connection, you are ready to work with documents.

### Compacting a database

{% highlight php %}
<?php

$couch->selectDb('database_name');
$couch->compactDb(); // Compacts the current database.
{% endhighlight %}
