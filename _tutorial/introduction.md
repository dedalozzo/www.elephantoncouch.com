---
layout: tutorial
title: Introduction
permalink: /tutorial/introduction/
---

If you have landed on this site, you're probably a PHP developer who decided to use CouchDB for his next project.
Maybe you are simply tired to work with MySQL or you want just try something new, like I did when I approached the
first time the NoSQL world. If any or all the premises are correct, you are in the right place.
There are a bunch of client libraries for CouchDB,
so you probably want to know why I decided to write a new one and I didn't use something already in place. The answer
is pretty simple: they all suck. Yeah, I may sound a bit harsh here, but I'm not a diplomat and the truth must be said.
CouchDB is an Erlang piece of code, using the worst protocol out there, a protocol everyone, even the illiterates, uses
when surfing the web. Since CouchDB uses HTTP, you might be temped to use cURL, you'll start experiment some commands,
and finally you land here where you can find some peace and... relax.

## The three musketeers

EoC, acronym of Elephant on Couch, is a suite of tools, created to simplify your life. Once you have learned the basic 
concepts around MapReduce, you are half the way in the process to embed CouchDB in your application. In fact, you need 
to learn the protocol, dealing with cURL, and many other things you can't even imagine. EoC is gonna help you to focus 
on your application, providing everything you need to be productive.
There are three useful packages: a client, a server and a command-line interface. You are gonna use the server just in
case you decide to write the MapReduce functions in PHP, else you don't need it. The CLI, instead, is provided for your
own good, to avoid cURL and make easy the administration of your database instance. The client is the library you will
use to handle CouchDB from inside your application.

### Client

The client library is pretty cool. It can be configured to use cURL or native sockets: the first one is more stable, 
the second one is a little bit faster. I recommend the cURL adapter, since you need stability overall, but if your 
server doesn't have it installed, you can go with the socket adapter. If you are not satisfied with my own 
implementation, you are free to implement an interface to provide your own adapter. We'll talk about it later.
EoC Client is **not** like all the other libraries, just a wrapper on cURL to send commands via HTTP. EoC Client is an 
abstract layer aims to add persistence to your classes. You can extend every single class of your project, in many ways 
(we'll see this later), such as the instances of extended classes can be stored into CouchDB. Later, when you read a  
document from the database, an instance of the same class you saved before is created.
EoC Client comes with the ability to join queries and provides an interface to handle chunk responses, when you need to 
process data as soon they are read from the database. These are just a few incredible benefits of using this library. 

### Server

As I said above you need the server just in case you want implement map and reduce functions in PHP. CouchDB is a 
strange animal that comes with an embedded JavaScript query server, but you are free to configure it to use another
engine, like the one I wrote, for PHP. To be honest JavaScript is faster than PHP, but the server I have developed is 
pretty fast indeed, so give it a try. Keep your functions simple and remember that you can still rewrite the slow ones 
using Erlang.

### CLI

Like the two above, the CLI is provided for your convenience in the form of a Composer package. Like any other Unix 
command, it comes within an integrated help. Everything you do from Futon and Fauxton, can be done using the CLI. This 
library, unfortunately, uses Sh-Mop to keep the connection active and to remember the database in use.
Since CouchDB doesn't provide a tool to execute queries, you'll find a right-hand man in this command-line interface.  

## Acknowledgments

This job has been possible thank to the support of some, well known, CouchDB developers, like Robert Newson, Noah Slater, 
Alexander Shorin, and many others. A special gratitude to Tom Preston-Werner for the awesome Jekyll template, used for 
this site, proudly hosted on GitHub Pages. 
