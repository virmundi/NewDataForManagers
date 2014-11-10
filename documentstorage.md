# Document Storage

When developers in the OO world work with traditional databases, they will often load the data into objects. The objects model the problem at a domain level. The database models the problem in a normalized, relational way. The interaction between the two layers doesn't quite fit. This is called the *object-relational impedance mismatch*[^oo_rel_mismatch]. 

At the same time, code is more flexible than a relational database. To add a field to an object normally requires just typing in the field and auto-generating the accessors. A database requires tables to be created, altered and migrated. Great tools like the Hibernate family or Rails mitigate the issue, but the team must plan and execute migrations. Document Storage seeks to bring that ease of change into the database world.

## Architecture
The heart of a document store is the idea of a document. A document is simply a holder of one or more key-value pairs. What makes documents distinct from a K-V storage is that the datastore knows how to interact with a document structure. Unlike a K-V store, the database knows what structure a given document has. It knows document fields. As such you can query against them. 

For example, a document might have information about a person. Within one entry you'll see first name, last name. These are strings like "Patrick" and "Davenport". The document might also have a key named "address". Address is a complex value: street, city, state, zip, etc. That complex value is stored in the document itself rather than requiring a separate address table.

How the document store actually stores the data is really dependent on the implementation. Some store the documents a JSON (JavaScript Object Notation) entries. These are plain text records that look like this 

~~~~~~~~
    { 
        "first-name" : "Patrick",
        "last-name" : "Davenport",
        "address" : {
            "street": "123 Fake St",
            "city": "Springfield",
            "state: "???"
        }
    }
~~~~~~~~

though they are seldom pretty formatted when actually saved. There is a derivative of JSON called BSON (Binary JSON). They can also be shaped documents which is like JSON, but the actual field names are not stored for the document, rather document has a shape id. This kinda normalizes the shape information to reduce storage size.

Documents exist within a Collection. A collection is simply a named container for one or more documents. A collection is equatable to a table in the RDBMS world. 

A Database houses one or more collections. They are also responsible for housing meta data about user rights, indexes and other meta concerning the management of the records. 

There is no SQL like, universal query language for document stores. Each vendor provides their own unique take on the process. Each implementation views expressiveness in different ways. Some might have a SQL like language. Others might leverage a programatic interface. Some support joins; some require joins to be executed on the client side. Finally, some support transactions at the document level while others, ArangoDB notably, support batched transaction. As a result, there is a degree of vendor lock in. You will find it difficult to switch backends without having to rewrite most of your data access layer.

Replication can be either master-master or master-slave. When you're looking at replication please keep in mind your organizations need to replicate data between data centers. Master-master setups are better at this than master-slave. Most implementations allow the client to call the replicas for read operations. This allows shorter response times and distributed work.

Most document stores communicate via HTTP/REST. They support actions like POSTing a new document. GETting a document by an id or query. DELETE-ing a document. The benefit here is that as long as the programming language you use has an HTTP client, you can use the database.

Often a document store is referred to as schemaless. The reason for this is that there is no mechanism within a document store to enforce a structure on a document within a collection. It is possible to a have Person collection that has the majority of the documents looking like the one above. It can also house any other document too. For example we might have a document that looks like
  
~~~~~~~~
    {
        "reset": "2014-10-01"
    }
~~~~~~~~

Such a record is impossible in a relational model. This document obviously does not conform to the relational definition of a Person.

Another reason that a document is referred to as schemaless is that there are rarely any type constraints on a the members of a document. It is possible to have the field "first-name" with a value of "Patrick" or a value of 100.00. Adding a document like that will not throw an exception. 

The lack of typing means that there are no hard values on the length of a string. Normally one would say that a "Note" column in a relational table is VARCHAR(2000). If a user entered a note that was 2,500 chars long, the system would either fail to accept the insert or would truncate the note. Document stores will happily store the record with 2,500 characters.

The result of schemaless management mandates that the application must coordinate domain rules. If a note has to be 2,000 characters or less, the application has to enforce how to handle large notes. The application also has to manage document relationships. Even systems that support joining documents fail to enforce foreign key relationships. Document A might have a list of related document ids in collection B. If any of those documents in collection B are deleted, the application must remove the entries in Document A. Otherwise Document A will have dangling pointers.

## Getting to Know the Players

Given the relative ease of switching to developing against a document store, they are some of the most developed systems. There are a lot of different vendors from which to choose. We're going to look at three. Each provides insight to particular ideas in the space. Just because a product is not in the list does not mean I don't like it. For example, RethinkDB would have been my personal choice for a project, but it lacked geo-indexes at the time. It is however, an up-and-coming database.

### ArangoDB

This is a relatively obscure one, at least in the USA. It's an example of fine German engineering. Unlike many document stores, it is actually a document and graph database rolled into one. It offers geo-spatial indexing, document flexibility, graph relationships and scaling. 

It presently supports scaling via replication and sharding. Replication is asynchronous master-slave. This means that BASE rules apply: the slaves might be out of sync with the master for a short time [^arango_rep]  It does not presently provide automatic failover in the event of a master going down. Sharding is managed on the document's key.

The ArangoDB team wants to make rapid prototyping easy. Rather than having to have a web server and document database, you can use Foxx, an embedded, Node.js like, web environment.  This allows you to wrap your database queries with a REST interface. Now you've got less communication between the backends and shared resources too! The client doesn't know that it's calling Arango. If you find that Foxx no longer meets your needs, you can tier the application without impacting your clients.

What makes ArangoDB standout is its query language AQL. AQL provides a SQL like expressions for filtering, updating documents. Unlike other document stores, AQL allows joining documents. This helps to limit the number of return calls to the database to perform the join in the application. AQL also makes the database much more accessible to business analysts and BI professionals. While there is a learning curve to AQL, it is SQL-like. In no time your analysts should be comfortable with AQL queries as they are with SQL.

Finally, ArangoDB supports multi-document transactions. I believe this is the only such database to provide that feature. While not as advanced as an Oracle or SQL Server transaction, ArangoDB allows a client to send a batch of changes to the database. They are either all committed or they all fail.

For more information please go to [http://www.arangodb.com](http://www.arangodb.com).

### Cloudant

Cloudant is a flavor of CouchDB. Being a CouchDB means that you get an interesting solution for offline data synchronization and replication. Cloudant takes this a step further by providing a great management interface across the cluster.

Cloudant has a great history of scaling stories. When you review their case studies you'll see how they've helped companies scale from a one node RDBMS to a 200 node system with low query latency while having gigs and gigs of data at rest.

Traditionally Cloudant, as its name indicates, is a platform-as-a-service provider. Your data is hosted in a secured cluster within the Cloudant data centers. They are responsible for managing the day-to-day operations of the database. Your company can then focus on writing software. If your company prefers to manage their platforms, you're in luck. In October 2014, IBM announced a new [on-premise](https://cloudant.com/blog/introducing-cloudant-local/#.VGDX9vTF9hs) option.

Recently IBM purchased Cloudant. This might make the idea of using it more palatable in the enterprise space. I know many a manager sleeps better at night knowing there is a Batphone (Bluephone?) to IBM if they need it.

For more information visit [https://cloudant.com](https://cloudant.com)

### MongoDB

This is probably the most known of the document stores. It helped popularize the NoSQL movement (and some of the backlash against it). It is going strong through the efforts of the people at MongoDB, Inc.

MongoDB uses a proprietary storage format called BSON (Binary JSON). It allows the system to more efficiently interact with the data as well as reduce storage size. 

MongoDB is centered around the cluster. It is rare to see community discussion around a single node MongoDB deploy. Often the recommend production configuration is three nodes: one master and two slaves. As a result you might have a larger upfront cost associated with the initial deployment.
 

## So How Would We Use This?

### There Is No Spoon
Since this is a semi-technical book, I'm duty bound to have a Matrix reference. Neo talks to a little, white boy dressed as a Tibetan monk about the reality of cutlery. The child's goal was to impress upon Neo that his world view needed to shift in order to absorb reality of the Matrix. So too your baggage around modeling data from the RDBMS world.

The first thing that you have to accept is that modeling a document store is different than relational modeling. Relational modeling uses normalization to link various tables together. Let's look at a relational model for the person. There is a Person table with columns F_NAME VARCHAR(20), L_NAME VARCHAR(20, ID INT PK. There is an Address table with columns ID INT PK, STREET VARCHAR(20), CITY VARCHAR(30). Finally there is a PersonAddress table with columns P_ID FK, A_ID FK. If your application normally shows a person with their addresses, two joins are necessary Person -> PersonAddress <-Address. Given the domain rule that we show a person with their address a document might model like we saw above. Some documents might have a list of addresses. Some documents might have a single, non-list entry. The exact details are left up to the application it stores the information.

Modeling directly impacts the need for transactions. In the section on *Dropping ACID in order to FREE-BASE* we saw that ACID might not be necessary for my data functions. In the relational world adding a new address to a person, say their work address, requires modify two tables: Address and PersonAddress. To be safe, a transaction is started before the write and only completes when the database promises that it persistently wrote to the two tables. Contrast this behavior with a document change. We would essentially only PATCH (in REST parlance) the document. Transactions in every document store I've researched are ACID at the document level. Either the document is update in whole or its not. There is no partial state. 

### Daily OLTP Work
Many companies find the modeling power of documents eases the *object-relational impedance mismatch*. If your in a JavaScript-all-the-way-down shop built on Node.js, there is no mismatch. Your developers work entirely in JSON. Clojure shops will experience the same benefit due to JSON being easily mapped into MAPs and VECTORs, etc.

If your work load is heavily read oriented, clients reading from replicas will dramatically improve system responsiveness. Rather than one server handling all requests for information, the workload is distributed across the cluster. Sharding extends this benefit further by allowing clients to read replicas for a shard.

### OLAP Reporting

While many BI tools are racing to catch up with document databases, power users leverage them daily to run sophisticated reporting across the cluster. Map-Reduce found in Couchbase, Cloudant, and MongoDB allow developers to write programatic queries that run across all cluster in parallel. ArangoDB's AQL aim to provide the power of Map-Reduce with the expressiveness of SQL. 

Because the databases horizontally scale, reporting can run much more quickly than in traditional database. It is possible to setup replicas just for reporting. Let's say that one out of three nodes in a replica set are used just for reporting. The OLTP clients only know about the other two nodes. Now you can quickly manage system clients while also quickly answer business queries without having to setup an offline data warehouse. 

### Archive Management
MongoDB helped Craigslist comply with regulatory requirements. Craigslist provides online ads for a variety of consumer needs. Everyday they server thousands of ads to the Internet. They also have to track meta-data about the ad content for a period of a few years. Upgrading the scheme caused the single database system to slow due to having to update the meta-data of the historical records. At the same time Craigslist wanted to reduce database administration costs.

MongoDB's ability to shard and replicate the data allowed the rapid storage and retrieval of 10 TB of documents. The schemeless nature of documents enables easy generational versioning. If a new generation needs to add one or more fields, it can. If they wish to consolidate fields, they can. It is up to the consumer of the data to manage differences between generations.

### Mobile Development

PouchDB is a database for JavaScript apps and Node.js. In many ways it's document store version of the BerkleyDB. It replicates the API found in CouchDB. It allows an offline application to store information as it would it if was online talking to a CouchDB compatible HTTP API. When the app goes online later, PouchDB will synchronize itself with the enterprise. 

With a little technological finesse, the application does not know, nor does it care if it is online or off. The developers can focus on learning and leveraging the CouchDB API to the fullest. The app will take care of itself.

There are other PouchDB like tools that extend the mobile sync concept to native applications. 


## Sizing and Costs

The architecture of most of the document stores requires memory. The field is young. Most storage systems of this kind rely on memory-mapped files. Without getting too technical, let's say the more RAM the better. 

How much is "more RAM"? That depends on your data amount. Memory-mapped architectures prefer to keep all of the necessary data to answer a query in memory. This does not mean that you have to have equal parts RAM to stored data. The common memory configuration of at least 8 GB of RAM should suffice for data sets around 5-20 GB.

CPU is seldom the bottleneck in databases. A commodity server with an x86 architecture should suffice. Experiment with your specific workload. If there are a lot of in-memory calculations such as summations, the box might need a bigger CPU.

Document stores will run on traditional HDDs. In a commodity server world, you could have a few HDD systems in a cluster and get passable performance. If your IT policy says all servers must have HDD and only HDDs, you'll be fine. That said, everything runs better on a solid state drive (SSD).

SSDs are memory based storage rather than magnetic. While they are no where near as fast as actual RAM, they are dramatically faster than HDDs. Because documents stores tend to use memory-mapped files, the SSD speeds these operations up. If you're not using them already, you should seriously look into SSDs.

When clustering, follow the manual or community recommendations around network configuration. Some systems provide multi-datacenter support out of the box (CouchDB). 

### On Premise

A commodity server with 4-8 GB of RAM should provide the core node configuration. At the time of this writing, adding for SSDs to the box will cost about $800-$1,000 US and provide 1 TB of storage. 

### In the Cloud

A reasonable default configuration for a cloud server is at least 1 GB of RAM for a dedicated machine. As you use the system, you'll find out if you have any bottlenecks. You might have to upgrade to the next RAM size.

Check the manual of the document store you're picking. They will have different takes on how to configure a server or cluster in the cloud. Expect to get slower performance from I/O where the disk is not actually part of the server you're renting. For example EBS drives in AWS have wild performance characteristics. To even them out, you'll need RAID. If your cloud host offers physical disks like Linode or Digital Ocean you'll get better performance, but limited storage.

You'll want servers with a good amount of RAM. If the document store is able to cache most of the records in memory, you'll get great performance regardless of instance or networked physical storage.

[^oo_rel_mismatch]: http://en.wikipedia.org/wiki/Object-relational_impedance_mismatch
[^mongo_craigslist]: http://www.mongodb.com/customers/craigslist
[^arango_rep]: http://docs.arangodb.org/Replication/README.html