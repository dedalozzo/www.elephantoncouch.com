---
layout: tutorial
title: Installation
permalink: /tutorial/client-installation/
---

## Composer Installation

To install EoC Client, you first need to install [Composer](http://getcomposer.org/), a Package Manager for
PHP, following those few [steps](http://getcomposer.org/doc/00-intro.md#installation-nix):

{% highlight bash %}
$ curl -s https://getcomposer.org/installer | php
{% endhighlight %}

You can run this command to easily access composer from anywhere on your system:

{% highlight bash %}
$ sudo mv composer.phar /usr/local/bin/composer
{% endhighlight %}


## EoC Client Installation

Once you have installed Composer, it's easy install Lint.

1. Edit your `composer.json` file, adding Lint to the require section:
{% highlight json %}
{
    "require": {
        "3f/eoc-client": "dev-master"
    },
}
{% endhighlight %}
2. Run the following command in your project root dir:
{% highlight bash %}
composer update
{% endhighlight %}

## Requirements

Installing EoC Client is easy and straight-forward, but there are a few
requirements you’ll need to make sure your system has before you start.

- PHP 5.4.0 or above.

Now that you’ve got everything installed, let’s get to work!
