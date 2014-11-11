# Key-Value Stores
Key-Value stores (K-V) are perhaps the simplest of the NoSQL solutions, at least logically. Most developers use maps or dictionaries in their everyday coding. Key-Value stores are an expansion of this idea. Another term for Key-Value is a Distributed Hash Table. As well see, a K-V is a lot like an old Rolodex. The key is the person's last name. The value is their contact information. Knowing the last name allows you to quickly jump to the proper location in the Rolodex. 

K-Vs are everywhere. You can get them for mobile, for the server side and even in HTML5. Where they lack in complexity, K-Vs make up for it with their ubiquity. Depending on your application or system needs using K-Vs through the stack can lead to less mental overhead since developers won't have to jump between different persistence models.

## Architecture
At the core is an incredibly simple, yet powerful abstraction: keys and their values. A key is a set of bits that uniquely identifies a thing. That thing is called the value. It too is simply a set of bits. The most rudimentary key-value stores don't attempt to know what's in the key or in the value. Both are opaque to the storage mechanism. A key might be the string "name" with a value of "Patrick". Another key might be the hex number 0xBEEF with the value of "It's what's for dinner". The data store doesn't force any representational semantics on the developer.

The key's content is hashed. This means that it's turned into a new, fixed size data used to pick a slot in a table for the value. Production hash functions are fast. Looking up a value via the hash is fast. Two fasts equal a fast system.

Given the logical simplicity of the paradigm, the simple K-V provides the operations of `GET`, `PUT`, and `DELETE`. `GET` retrieves a value from the store. `PUT` inserts a value at a key if it doesn't exists or overwrites the value if does. `DELETE` removes the value from the data store.

As stated earlier, most programming languages have the concept of a K-V as a map or dictionary.  For example, Java looks like this.

~~~~~~~~ 
    HashMap<String, Integer> cache = new HashMap<String, Integer>();
    cache.put("user.age", 21);
    System.out.println("User age is " + cache.get("user.age")); // Prints User age is 21
    cache.remove("user.age"); // Removes the value 21, and returns it.
~~~~~~~~

From the examples above one thing should be obvious, key uniqueness matters. If we used "user.age" for all of our users, we would overwrite the values and get it wrong. Instead we could model it with "&lt;username&gt;.user.age" where &lt;username&gt; is the id for the present user.

While basic CRUD could be enough for some implementations NoSQL variants go beyond it. Many remove the opacity of the values. Rather than treating them as blobs, the K-V knows they are a set of items or a list of items or a counter. As a result the developer/modeler can safely add to a list in a concurrent environment. They can universally and atomically get the next value of an ever increasing integer. 

So far we've really just talked about a single node system. In the early days, K-Vs largely were a single node system. Time has marched forward. Many K-Vs support clustering. The development team is still using a logical hash table. That table is spread across multiple nodes. The hash function often performs double duty in a distributed system. Not only does it pick the slot for a record, but it also picks the node. 

As we'll see in the Product Overviews section, a K-V can store its information in memory or persistently to a disk. There are tradeoffs and design considerations with either mechanism. Pure memory options are useful for pure speed. They are also purely transient. If the box suffers a power outage, everything in memory will probably be lost. This might not be a bad thing&trade;. When the K-V persists it must  slow down some in order write. How much of a slow down is product specific. The benefit is that while slower, you're system can survive a reboot.

Most of the K-Vs are transactional at the key level. This means that when you put a value for a key, you either put the whole value or you won't. It is not possible for a partial write to occur. The same is true for a read. You cannot read a partially added value. For example, say you wanted to `put` the value `Bob` in the key "&lt;usersession&gt;.firstname". It is not possible even during failure to write `Bo` or `B`. When a client of a K-V requests &lt;usersession&gt;.firstname the system will either return an empty response (if it hasn't got a value or if the value is removed) or it will return `Bob`. It can't return `B`.

## Getting to Know the Players

### Berkeley DB
The home page is 
http://www.oracle.com/technetwork/database/database-technologies/berkeleydb

This is the granddaddy of the NoSQL systems. It is so old (conceptually) as to not even qualify as NoSQL according to some members of the community [^kv_bdb]. To others it is a NoSQL database [^kv_bdb_how_to]. I think it is too, so I included it. 

It is an embeddable K-V. It runs as part of the memory space of the host applications. The host application accesses it via a C-lang binding. The C-lang binding allows many languages to use it outside of just C. 

It is an impressive little system. Each release brings a new feature built on the previous layers. As a result there is a SQLite facade. This allows developers used to working with SQL to continue using SQL over BDB. Behind the scenes the SQL concepts are mapped to simple K-V entries.

Recent improvements allow Berkeley DB to scale beyond a single host. Know as the Berkely DB HA, you get all of the distributed goodness on top of a simple programming API. It is a master-slave setup with automatic master selection.

Depending on how you use it, Berkeley DB is immediately consistent since it is not, by default distributed. When running in HA, you can flip many switches to adjust consistency and read throughput. 

Oracle provides many white papers and blog posts on BDB. Here a just a few.

* SQLite API - A Technical Evalutaiton http://www.oracle.com/technetwork/database/berkeleydb/learnmore/bdbvssqlite-wp-186779.pdf

* Using Oracle Berkeley DB Java Edition as a Persistence Manager for the Google Web Toolkit http://www.oracle.com/technetwork/articles/audet-bdb-gwt-096313.html

* Berkeley DB Java Edition on Android http://www.oracle.com/technetwork/database/berkeleydb/bdb-je-android-160932.pdf

### Amazon DynamoDB

The home page is http://aws.amazon.com/dynamodb/ 

This is the fount of all modern NoSQL K-Vs. When the paper describing it came out in late 2007, it was not  available to public. Since then it was promoted to public service. You can't download and install Dynamo in your data servers.

What's special about Dynamo is its management. Since it is tightly coupled with the AWS infrastructure, a database owner can automate processes like adding a new node to the cluster on demand through a nice web interface. It is also inexpensive to spin up a cluster to kick the tires then kill it off. 

The API is more complex than the logical K-Vs we've discussed. Developers can query over an id. They can also scan over ids. This allows range searches.

There are multiple implementations of the Amazon's paper that allow you get the similar features, but within your data center. Riak and Voldemort inspired by DynamoDB.

### Redis
The home page is http://redis.io/.

Redis is a K-V in the traditional sense. It is also horizontally scales with failover. What makes Redis stand out is its speed and advanced features. Values are not just transparent bits. They can be strings, hashes, lists, sets, sorted sets, bitmaps and hyperloglogs. Changes to any of the data structures are atomic. If you add an item to a list, it either commits or it doesn't. Such types allow for a more expressive data model than in traditional K-Vs. Some indie developers use Redis for all of their data needs.

Redis takes types to the extreme by providing a fast pub/sub fire hose message queueing system. Clients can listen on a channel. Publishers can publish on the channel. It even provides simple routing on channel names.

### Other Players
* Riak - http://basho.com/riak/ 
A replicated K-V with a MapReduce chaser. 
* Memcached - http://www.memcached.org/ 
Old standby for caching services. Large knowledge base and community support.
* Project Voldemort - http://www.project-voldemort.com/voldemort/ 
Apache implementation of the DynamoDB concepts.

## So How Would We Use This?

### Generally Anything Transient Looked Up By Key
This is something that can get lost in the simplicity of it all. If you have information, text, pictures, serialized objects, that has a unique key and doesn't have complex queries required for it, a K-V is probably a good bet. A RDBMS is a great tool, but heavy when it comes to simple queries like `SELECT VALUE FROM TABLE WHERE ID= ?`. Using a K-V in the stack will reduce database load which frees it up for more complex service elsewhere. This can include saving database results into a K-V. Your application would check the K-V for an answer. Only if it doesn't exist will it go to the database.

### Session Management
Perhaps the most quintessential application of K-Vs is session management. User session information is a natural fit here. There is a session id. That is the key or part of the key. The value is whatever is needed. For example, a Java developer could use the session id as the key and store a complex Java object in the value. 

The lifecycle of a session starts when a user logs in (or perhaps when they simply connect to your site). This creates an initial value or set of values on the application server. The value is pushed to the K-V. Whenever the session information is accessed, the K-V is read. User actions cause various changes to the session, like updating a shopping cart or increasing analytical values. When the session ends on the app server, it is possible to trigger an auto delete or session ETL into another data store like MySQL or MongoDB and then delete the session from the cache.

Open source helps organization leverage caching systems seamlessly. Multiple projects provide a means of integrating web session storage into Riak or Memcached near the beginning of the request pipeline. As a result, most developers will not have to worry about where their session is stored. This frees them up to focus on the important things: the business problems.

### Fast Paced Data Landing
Some applications generate high volumes of data. It could be user clicks. It could be requests for ads. K-Vs provide fast lookups and inserts. Distributed solutions provide the ability to grow your memory space to whatever size you require. Once the data is in the cluster a background service can ETL it into a permanent storage if necessary. 

Some of the K-Vs support MapReduce to perform (relatively simple) analytics in the K-V itself. As a result it's possible track real-time or near real-time information like leader boards and dashboards.

### Oddly, Messaging
Since Redis is kinda sorta a K-V, we'll look at one of its standout features: fast messaging between components using the Pub/Sub paradigm. Normally when one thinks messaging they think RabbitMQ, IBM MQ or MSMQ. Most of these are a complex protocol, often times binary. Redis' protocol is fairly straight forward and text based. Clients register to a channel to publish. Other clients register on that channel to listen. The system is a firehose. If a client disconnects from the channel, it loses all of its messages. It doesn't guarantee delivery either. If you're willing to live within these parameters, you can create chat clients for your customer facing web sites with ease. Internally you can communicate anything with any component.

## Sizing and Cost Considerations
When considering sizing and cost one must, now a days, look to Cloud vs Local. We'll first look at sizing locally within a company. Then we'll look at what major companies provide by way of cloud hosting.

### On Premises
K-Vs are memory centric. The more memory you give them, the better they run. They are rarely CPU or local IO centric. As a result, you should put your money into RAM. 

Many companies find that a single K-V or K-V cluster can provide caching value to multiple applications. Another scenario is using the K-V for high volume writes and reads. In either case a fast network card or cards help.

After that, sizing varies by need. Essentially, you should get a server with 8 GB of RAM and a 100BASE-T network card. Depending on your level of failure response, you might consider adding a second NIC. Fortunately RAM is fairly cheap. Getting a single server with 16 GB should be cost effective.

If the K-V you're looking at supports sharding or replication, you might want to use it. You'll get better read throughput and possibly redundancy for fail over. Both are a good thing to have. If you go down the replication path, multiple your base server cost by the number of nodes.

### In the Cloud
Presently memory on AWS and Google hosts is rather expensive. If you want to standup your own Redis, Memcached, etc, you'll want to pick a configuration that supports high RAM, but doesn't cost you too much. An AWS `r3.large` presently offers 15 GB of RAM at $0.175 per hour used. Assuming your instance is on 24/7 with 30 average days per month, you'll spend $126/month to host the server. Costs go up as with storage fees. Fortunately most of the IO will be within AWS so you probably won't have to pay transfer fees. Google's `n1-highmem-2` offers 13 GB of RAM at $0.164/hr. So the average monthly cost is $118.08 with a bit less head room. Keep in mind these numbers are per instance.

Now the question is _should you stand up your K-V?_ On AWS you can use DynamoDB. Their pricing model shows that a small to mid-size site should probably cost around $7.50 [^kv_aws_ddb_pricing]/month. Since DynamoDB is persistent with more advanced modeling features, it could be your only datastore. Google offers a similar service partially for free using Memcache in the AppEngine space. The free version gives you unlimited storage in a Memcache pool. Sadly, nothing is ever free-free. Your items in the Memcache pool may be evicted at anytime depending on Google's needs. Might not sound useful, but every cache hit is a performance gain. You can also pay $0.06 per GB per hour for a dedicated Memcache pool. So a 24/7 30 days/month instance runs about $43.20.


## Further Resources
* _The Architecture of Open Source Applications_ has a chapter on the Berkeley DB found at http://aosabook.org/en/bdb.html.

* Data Modeling with Key Value NoSQL Data Stores - Interview with Casey Rosenthal found at
http://www.infoq.com/articles/data-modeling-with-key-value-nosql-data-stores

* Dynamo: Amazon's Highly Available Key-value Store found at http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf 

* Comparison of ZeroMQ and Redis by Stephen McDonald 
http://blog.jupo.org/2013/02/23/a-tale-of-two-queues/

[^kv_bdb]: http://www.oracle.com/technetwork/database/database-technologies/berkeleydb/overview/index.html

[^kv_bdb_how_to]: http://www.oracle.com/technetwork/articles/cloudcomp/berkeleydb-nosql-323570.html

[^kv_aws_ddb_pricing]: http://aws.amazon.com/dynamodb/pricing/