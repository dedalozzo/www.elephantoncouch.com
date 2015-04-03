---
layout: docs
title: Getting started
permalink: /docs/connection/
---

Once you have installed it, you're ready to use the client. In the example below has shown how to create a new 
connection with the server and select a database.

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

$couch->selectDb('database_name');
{% endhighlight %}

As we said previously, if you are not quite happy with the provided adapters, you can implement the interface `IAdapter` 
to create a new one.

Now that you've finally managed to create a connection, you are ready to work with documents.