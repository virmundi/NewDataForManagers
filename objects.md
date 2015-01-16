# Object Databases

You've probably heard of Object Oriented Programming. Object databases take the idea further by saying, "Hey, why not just save the whole complex object at once?" There is no messy ORM. The developer simply saves an object into the database and pulls it out at a later date. Exactly how all of this magic occurs is vendor specific. That is the general idea though: object.save() and find(query).

There are two terms floating around that sound similar, but for our purposes are distinct. The first is an object database. An object database is a database that stores objects. It tends to be tied to the application language. The second is object-relational database. Such a database still uses relational ideas under the covers. There is an object veneer across the top. SQL:1999 made this easier by adding custom types to the database. The fundamental difference between the two is the underlying storage model. Object databases are based on objects or if the vendor is being fancy "object calculus". Object-relational databases are based on set theory/relational theory.

Unlike many other NoSQL families, there are (mostly dead) standards available. Some provide direct interfaces for Java's JPA data abstraction layer. The Object Data Management Group created three standards before JPA. The work culminated in Java Data Objects (JDO). The Object Management Group (OMG!) tried to create spec vs 4.0. Unfortunately the economic crisis of 2008 caused the vendors in the consortium to pull out. How is this helpful? A) fun facts for a technical mixer and B) the vendors support  some level of the specification. When shopping around, you're not entirely in the dark about what they provide. 

## Architecture

Everything in a OOODB revolves around the idea of an object. An OOODB knows how to efficiently persist, retrieve and query Objects. This naturally leads to a question....

### What's an Object?

Essentially an object is what you get from your logical designs. All of those UML class diagrams map 1-for-1 into an object database. Your hierarchies, relationships and interfaces all get stored into the database meta-data. As a result, there is no disconnect between the logical and physical models.

Objects have an identity. A person's identity might be their name. It might be their government assigned identifier like their SSN. It could be a random increasing number. However its defined, once set that's how the person is referenced in the system. So to an object. 

Objects have behavior associated with them. Their behavior should be available regardless of point of origin. Behavior can come from a complex hierarchy. OODBs track and relate an object through its hierarchy.

Objects are complex. Picture of bill of materials (BOM). A product is made of up sub-products. Each sub-product is broken down into its constituent parts.  A car has a dashboard, which has a radio, etc. Two cars do not share a dashboard. The storage engine has to track and optimize such relationships.

### OODBs, How Do They Work?

There are a few models of how objects are stored and distributed. 

When storing to a physical location, an OODB can either manage the raw drive with its own file system (creating a partition just for the DB) or it can sit on top of the OS's file system. The benefit of raw storage is the database can optimize read/write performance. Often related objects are spatially co-located in chunks on the drive. The DB can then read the whole graph in one read cycle. OS backed storage is logically easier to comprehend and implement. Performance is not necessarily poor is such a system since modern file systems are fairly tuned for performance as well.

Distribution is a bit more complex. Servers may return whole object graphs. Let's say that there is a `Customer` object with an `Address` object. The server might return an instance of the object with  the customer details and the address. It might return the customer details with a pointer to `Address` that only gets address when you actually request it. A server might actually share memory pages with the client. Each distribution has it's pros and cons.

### Daily Life With OODB

OODB' are tightly coupled with one or a few programming languages. The benefit of this is that there's no mismatch between the common development tool and the database. This makes working with the storage engine mentally easier. Because there is no mapping layer, most systems are 10 to 1,000 times faster with 40% reduced code base (at least around parts of the system involving the DB)[^speed_language]. A possible issue with the coupling is that the only API in and out of the system is the target language. This may require ETL processes to move  production data from the OODB to an analytics platform.

Querying is an exercise in the host language. The client library exposes operations via an API. Simple expressions like find an object that looks like some instance of an object are normal Java to create the object and pass it to a look up method. There is no need to worry about typing of a column. Java's compiler will tell you what's needed. Complex queries are a composition of method. 

## Getting to Know the Players

### Versant
The homepage is <http://www.actian.com/products/operational-databases/versant/>

Versant is one of the oldest vendors in the list. They're database may be run ass both an embedded server or  a distributed, master-slave system. Each offers high throughput of data ingress and processing.

It is used as part of Verizon's fraud detection system. It was developed back in the MCI days. It can monitor for fraud in real-time.

### Objectivity
The homepage is <http://www.objectivity.com/products/objectivitydb/>

The maker of Objectivity is the same maker of InfiniteGraph discussed in the graph chapter. It has a proven history of rapid data ingress and analysis. It's used a variety of industries from Telecommunications to Healthcare.

A unique feature is how data distribution works. To quote from their site[^object_tech]:

> Fully Distributed, Peer-to-Peer Solution: Objectivity/DB’s uniquely distributed architecture
> supports many data models. Organizations can distribute the applications and leave the data 
> on a centralized server, or distribute the data across hundreds or thousands of computers and
> support centralized applications, or distribute the applications and the data. Developers can
>  also use placement options to position data anywhere on the network, and closer to where it
>  needs to be used. 

For case studies visit <http://www.objectivity.com/solutions/>.

## McObject

The homepage is <http://www.mcobject.com/>

Unlike the products listed, McObject is purely embedded. It is a fast, small footprint store used by multiple companies such as DirectTV, F5 Networked and Northrop Grumman. It is developed under a dual license for open-source and commercial applications.

For case studies visit <http://www.mcobject.com/who_uses_mcobject>. 

### A Moment of Silence for db40

The home page is <http://supportservices.actian.com/versant/default.html>

Sadly this is little embedded object store is no longer with us. At least not commercially. Let's all bow our heads for a moment of silence. It was a great lost to the mobile development community since it worked will even on Android.

## So How Would We Use This?

### Embedded Systems
OODBs have been the darling of CAD manufactures for year. Modeling physical objects requires complex graphs with many moving parts. The industry found that  traditional RDBMS couldn't provided the required performance of these relationships. 

F5 Networks uses McObject to manage configuration and rule execution in the F5 Big IP product family.

### High Speed, Complex, Self-Contained Data
OODBs take advantage of the single item nature of their data to land information quickly. Picture a BoM again. It is self contained. Even if the relationships inside the BoM are complex, the BoM is outwardly a single item. 

Since OODBs are a standard to themselves, each vendor implements a quick, efficient transfer protocol and object encoding. As a result then can send thousands of self-contained things around the network quickly. Since objects are locked often at the object root, there are no locks to manage for data ingress.

### General OLTP

Often such applications center around (semi) complex data with self-contained graphs. A user may update a document. Then query about some data from that document. In such a flow rapid ingress and egress are wonderful. Most OODBs support the use of indexes just like a traditional database. This makes for reasonable OLTP performance. Some OODBs even boast OLAP capabilities.

## Further Resources

* The Object-Oriented Database System Manifest. This paper was the first attempt to standardize on what an OODB is, <http://goo.gl/UpBj6D>
* Douglas K Barry's OODB Advocacy Blog, <http://goo.gl/d4tWji>

[^speed_language]: <http://goo.gl/Bzddqt>
[^object_tech]: <http://goo.gl/kOZH1v>