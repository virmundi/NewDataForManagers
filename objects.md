# Object Databases

You've probably heard of Object Oriented Programming. Object databases take the idea further by saying, "Hey, why not just save the whole complex object at once?" There is no messy ORM. The developer simply saves an object into the database and pulls it out at a later date. Exactly how all of this magic occurs is vendor specific. That is the general idea though: object.save() and find(query).

There are two terms floating around that sound similar, but for our purposes are distinct. The first is an object database. An object database is a database that stores objects. It tends to be tied to the application language. The second is object-relational database. Such a database still uses relational ideas under the covers. There is an object veneer across the top. SQL:1999 made this easier by adding custom types to the database. The fundamental difference between the two is that underlying model. Object databases are based on objects or if the vendor is being fancy "object calculus". Object-relational databases are based on set theory/relational theory.

Unlike many other NoSQL families, there are (mostly dead) standards available. The Object Data Management Group created three standards. The work culminated in Java Data Objects (JDO). The Object Management Group (OMG!) tried to create spec vs 4.0. Unfortunately the economic crisis of 2008 caused the vendors in the consortium to pull out. How is this helpful? A) fun facts for a technical mixer and B) the vendors support  some level of the specs. When shopping around, you're not entirely in the dark about what they provide.

## Architecture

Everything in a OODBMS revolves around the idea of an object. An OODBMS knows how to efficiently persist, retrieve and query Objects. This naturally leads to a question....

### What's an Object?

Essentially an object is what you get from your logical designs. All of those UML class diagrams map 1-for-1 into an object database. Your hierarchies, relationships and interfaces all get stored into the database meta-data. As a result, there is no disconnect between the logical and physical models.

Objects have an identity. A person's identity might be their name. It might be their government assigned identifier like their SSN. It could be a random increasing number. However its defined, once set that's how the person is referenced in the system. So to an object. 

Objects have behavior associated with them. Their behavior should be available regardless of point of origin. 

Objects are complex. Picture of bill of materials (BOM). A product is made of up sub-products. Each sub-product is broken down into its constituent parts. Either implicit or explicit is a relationship of those parts. A car has a dashboard, which has a radio, etc. Two cars do not share a dashboard. The storage engine has to optimize these relationships.

### ODBMS, How Do They Work?
There are a few models of how objects are stored and distributed. 

When storing to a physical location, an ODBMS can either manage the raw drive with its own file system (creating a partition just for the DB) or it can sit on top of the existing OS's file system. The benefit of raw storage is the database can optimize read/write performance. Often related objects are spatially co-located in chunks on the drive. The DB can then read the whole graph in one seek cycle. OS backed storage is logically easier comprehend and implement. Performance is not necessarily poor is such a system since modern file systems are fairly tuned for performance as well.

Distribution is a bit more complex. Servers may return whole object graphs. Let's say that there is a `Customer` object with an `Address` object. The server might return an instance of the object with just the customer details and the address. It might return the customer details with a pointer to `Address` that only gets address when you actually request it. A server might actually share memory pages with the client. 

### Daily Life With ODBMS

ODBMS' are tightly coupled with one or a few programming languages. The benefit of this is that there's no mismatch between the common development tool and the database. This makes working with the storage engine mentally easier. Because there is no mapping layer, most systems are 10 to 1,000 times faster with 40% reduced code base (at least around parts of the system involving the DB)[^speed_language]. A possible issue with the coupling is that the only API in and out of the system is the target language. This may require ETL processes to move  production data from the ODBMS to an analytics platform.

Querying is an exercise in the host language. The client library exposes operations via an API. Simple expressions like find an object that looks like some instance of an object are normal Java to create the object and pass it to a look up method. There is no need to worry about typing of a column. Java's compiler will tell you what's needed. Complex queries are a composition of method. 

## Getting to Know the Players

### Caché
The homepage is http://www.intersystems.com/our-products/cache/cache-overview/.

### Versant
The homepage is http://www.actian.com/products/operational-databases/versant/.

### Objectivity
The homepage is http://www.objectivity.com/products/objectivitydb/.

### A Moment for db40
The home page is http://supportservices.actian.com/versant/default.html

Sadly this is nice, little embedded object store is no longer with us. At least not commercially. Let's all bow our heads for a moment of silence. It was a great lost to the mobile development community since it worked will even on Android.

## So How Would We Use This?


[^speed_language]: http://www.service-architecture.com/articles/object-oriented-databases/odbms_faq.html