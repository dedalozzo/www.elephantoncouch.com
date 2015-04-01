---
layout: docs
title: MapReduce
permalink: /docs/mapreduce/
---

To create the data model, you might use a data manipulation language, also know as DDL, or some tool that does the job 
in your place. You have probably done it hundreds or thousands of times.
I'm quite sure you have also used an ORM to simplify your life, but soon or later you 
have to write some SQL. You designed entities that you call tables and you modeled their relations. Maybe you are bored 
and you dream an imaginary world lacking of a proper, well defined, structure. I'm afraid to tell you that this world 
doesn't exist and NoSQL is just a container where you can find every database management system that is not relational, 
included CouchDB.
You still have to write your "relations", you still have to design your data model. How do you do that?
Let's start from the data. CouchDB doesn't have tables, neither it is a key/value store; it is, instead, a document 
database: a place where you can store documents of any type. Unfortunately a document is not an object, but it's a JSON 
representation of a data set. Elephant on Couch threats documents like objects and "vice versa". Any instance of a 
Document subclass is automatically converted in a document when saved. A special attribute determines the class of the 
object has been saved, in such a way as to allow the creation of an instance of the same class. This job is done  "under 
the hood" by the EoC Client. The magic trick above transforms CouchDB, literally, into an object-oriented database.
This solve the data problem: you don't have to deal with tables, you don't need to map anymore the objects to your 
tables, because thanks to EoC, you can equip your objects with persistence in the blink of an eye.
How do we model the relations? We do that in the same way we always done it: through references. This is a complex topic 
I won't discuss here, since there are different strategies: sometimes you need to join data, other times you don't and 
you can store them all in a single document. Depends. Let's just say it's something must be faced case-by-case.
But you still have to query your data. How?
