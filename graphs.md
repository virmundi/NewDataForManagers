# Graph Storage
Relational databases use set theory to model entities and their relationships.  Each entity is related to another via a column or a group of columns. For example, person's home ownership might be modeled by a **PERSON** table with an **ID** column, an **ADDRESS** table with an **ID** column and a junction table with a **PERSON_ID** and **ADDRESS_ID** column. The junction table is the relation.

Graph databases at first glance look similar. There are entities and relationships. Users move from one to another using a query language. The difference lies in how relationships are modeled. The result is massive speedup in queries around relationships when compared to the relational model.

## Architecture

The core concept of a graph database is so simple as to be laughable. There are only two things involved in modeling: nodes and edges (or vertices and relationships depending on whose literature you read). As you can use points and lines to draw everything from a stickman to the Sistine Chapel, graphs can model simple relations to complex distribution webs.

A node is a just a container for properties like `name` or `birthdate`. A node can have as many properties as you need. An edge is the thing that connects two nodes. These are named concepts. Getting back to our home ownership above, one node would be **"Patrick Davenport"**. Another is **"The Lone Palm"**. The relationship of **"Owns"** not only links me to the house but explicitly declares my ownership of it. 

Edges can have properties too. The "Owns" relationship might have the date I purchased the home. It could be marked as my primary house (as opposed to my other vast real-estate holdings). Edges also have a direction. I *Own* Lone Palm; the house doesn't own me.

An important rule for graphs is that you cannot have dangling edges. Every graph database will delete a node and all the edges pointing to or from it. No more worrying about orphan documents or values.

Under the covers graph databases store the nodes and edges efficiently. They can plow through a fairly complex query like "who are my friends' friends' friends that drive a car and enjoy llama videos?" A RDBMS using a simple two table design of Person and Friends would take a while to get this answer. It would have to join back onto the table twice to find the relationships, then apply a filter on them to find the individual that matched. Graph databases, on the other hand, can spit out "Carl" in a few milliseconds.

Distributed graph traversal is the brass ring for the industry now. Some vendors have supported the operation for a few years. Others are just getting into the feature. Distribution allows the graphs to grow larger than what one box could handle.

Like other NoSQLs there is no universal query language. Unlike most NoSQLs there are two on the horizon: Gremlin and SPARQL. Checkout the roadmaps of any product you're investigating to see if it supports either. Hopefully more will. Enterprises and developers can sleep a little easier knowing they are not locked in to a particular vendor.

## Getting to Know the Players

Wikipedia provides a list of over 20 products in the graph storage space[^wiki_graphs]. They run the gambit from GPL open-source, to open-sourced community edition with closed-source enterprise edition to fully closed source/paid license edition. That is a lot to choose from. The selected projects cover the licensing and notoriety spectrum. 

### ArangoDB

The homepage is http://www.arangodb.com/.

If you've read the document store chapter, you'll be familiar with ArangoDB. It's a multi-model, distributed system. It's licensed under Apache 2.0. There are no limitations to the number of nodes you can run in production. You can use it to back open-source and closed-source projects. 

The graph feature is in its nascent stage. You can add nodes and edges. You can query on it. You can do all this on only one server. The road map has distributed graphs targeted by the end of 2014. If you need to house a large graph, you can either size a single box appropriately or look at alternatives.

ArangoDB's flexibility means your developers don't need to know two different server technologies for graphs and document storage. DevOps can learn how to manage ArangoDB and only ArangoDB.

### Neo4J

The homepage is http://neo4j.com/.

Probably the most well known of the graph offerings. Written in Java (hence the J in Neo4J), it provides rapid execution of queries. Licensing is complex. There are four options: community, personal, startup and enterprise [^neo_license]. Community is free. You get access to community support and public documentation. The other license models add support tiers and add features like high availability.

Neo4J supports three query languages: SPARQL, Gremlin and Cypher. The first two are open languages. Cypher is Neo4J's proprietary language. Cypher often provides better performance. The others provide decoupling from the underlying vendor.

An interesting feature of Neo4J is embeddability[^neo_embed]. Any application that targets the Java Virtual Machine (JVM) can call into Neo4J. The result is that the database is fast. It's in the same process space as the application itself. There is no network communication causing worry about performance and security.

### InfiniteGraph

The homepage is http://www.objectivity.com/infinitegraph/.

Of all the systems reviewed for this book, InifiniteGraph is one of the most mature and complex graph databases on the market. It provides a truly distributed graph space. You can manage data locality and other aspects from a beautiful admin system. 

It is a closed-source system. Licensing is more like Oracle in that you can pay per site or processor. You get a 60 day free trial. 

This is the go to system if you know you'll have large, even approaching ***big*** data. 

### Special Mention: Apache Giraph

If you've ever wondered what graph system supports Facebook [^fb_giraph] (and possibly the NSA!), wonder no more: Apache Giraph. This graph system sits atop Hadoop. Facebook's used it to hold over a trillion edges. Using, and improving, Giraph, Facebook can analyze the interrelatedness of its 1 billion users.

If you have such a dataset and you want to stick to open-source systems, Giraph could be interesting. You will require people versed in Hadoop YARN, Hive, and MapReduce. Don't worry, your people can learn it. I learned Hadoop and Cascading in a 7 week production release.

Because of its Hadoop roots, Giraph is batch oriented. It has to boot up the application you want to run, prep memory and a lot of other steps to get it to run nicely in the MapReduce environment. The community implements ways to make distributed graph traversal easier and more performant, but it is still nearer the batch side of the spectrum than the real-time. If you can use a powerful, scalable, **batch oriented graph database**, Giraph is for you.

## So How Would We Use This?

### Deep Relationship Analysis

I figure the best place to start is the obvious use case: relationship analysis. Facebook uses Giraph to figure out which people are related to each other. They do this either by straight links like Bob `KNOWS` Sally and Sally `KNOWS` Jim. Or through interests like Bob `LIKES` Cats, Stan `LIKES` the musical Cats, thus Bob might like it too? Then Facebook waits for Bob to like the feast for the eyes and ears (as long as you're within the first 8 rows), __ Andrew Lloyd Webber's Cats__. 

But you might be thinking, "I work at an insurance company. We don't care about **likes** and who tweets what, and kids with their __Rock'em Sock'em Robots__. How can I use this?" My response is that you've got relationships coming out your ears. You have policy holders. They have information like what city they live in. Where they commute. What they have. Pouring all of that information from a data warehouse  into a graph system could help either a) improving standard relational queries performance or b) empowering your analysts to find new relationships that no one knew were there. It could be that people with labradoodles and yellow VW microbuses have an increased likelihood to have there "sun lamps" catch fire. 

### CRUD Applications
As we saw in the architecture section, a graph is a named node which has properties and edges are named relationships between nodes that also can have properties. This looks an awful lot like the ERD diagrams or analysis diagrams drawn up at the begining of a project or iteration. 

Traditionally those diagrams are called a logical model. Teams talk in the logical model. They might say that a `Policy Holder` has a `Life Insurance` policy with seven `Riders`. Lines go between each entity with names and possible annotations like `Expires On`.

Sadly, logical diagrams are like cattle. They are blissfully unaware that they'll soon be butchered into a physical model that carves them until their more nuanced relationships are nothing more but pink slime on the development floor. At this point the logical model is merely an abstraction that gets in the way. A BA will say, "A policy holder has such and such." The developer will have to say (politely of course) that the policy holder doesn't have that in the physical model. They might have to go further and say that the physical model needs a good amount of work to support what the BAs asking.

Graph databases avoid this issue. They look and are stored as logical diagrams. Since physical storage matches logical modeling, there is little impedance between the OO or even functional language of the development team and the data itself. If at some point a new relationship springs forth, the team should have little problem adding it; it's just another edge.

All this being said, you don't want to do heavy computations in a graph database. If you have tabular data, with tabular math, leave it in a relational database. They are optimized for that task. If you only need to run heavy calculations semi-some-time, ETLing from a graph database to a RDBMS is fairly simple. 

### Fraud Detection and Logistics

Logistics can use graph databases to find shortest routes for a series of pickups and drop offs. Each drop off is a point. The roads are edges. You might not need to have this data yourself. MapQuest provides an API [^mapquest] (some paid, some free). Under the covers they leverage Neo4J calculate routes.

Fraud is about relationships. There is the relationship such as  the impacted person and the fraudster or fraudster with other fraudsters. From personal experience it is difficult to get timely fraud analytics even with Hadoop when the data queries against itself. 

For example, say you've computed some fraud likeliness score for a medical provider. If the person's score is greater than some value, you want to flag him or her as a fraudster. From there you want to see who the person has done business with. That join is fairly easy. But say you want to expand the fraud detection net. You want to find fraudsters three times removed. This search space gets larger and larger with every relationship. It gets even more complex when you want to see if fraudsters are sharing patients. 

There are two issues with that type of query in RDBMS. 1) They become complex to maintain. Joins of joins of joins are hard to read and easily horked. 2) They are slow, even with indexes. Indexes can help with the initial selection of fraudsters. The constant re-cycling on the same table can cause indexes to be missed. You can get around this by having intermediate tables, but then you're back to issue 1. 

Graph databases bounce from node to node just like a dedicated follow of fashion. Indexed relationships provide rapid navigation even as the nodes are self referencing. As a result you might see a complex relationship query complete in a few seconds in a graph db where it might take hours in a RDBMS. 

## Sizing and Cost Considerations

Graph databases are system intensive. They prefer to keep the entire graph in memory. Calculating node relations can eat the CPU. If the data doesn't fit into memory, they might have to go to disk a lot. As a result, you'll need to start with a fairly large, commodity server.

### On Premise

The form factor of the server doesn't really matter. Choose whatever fits your deployment best. At its heart the server should have a Xeon with 8 GB of ECC RAM. You'll have fewer worries about data corruption this way (if you're not that worried about corruption, an i7 is fine too). 

Prefer SSDs over magnetic disks. When a graph database starts cold, it has to pull the entire graph into memory. SSDs will make this much faster. The size is dependent on your storage needs. You can get a production SSD with 500 GB of storage for around $300.

The best thing to do is test, test and test some more. Load a portion of the dataset. Watch the system metrics like RAM consumption or CPU as you perform different analytic or transactional processes against them. From there, extrapolate the metric of the final data set.  You'll gain precious knowledge about not only your system demands, but how to interact with the new data store.

### In the Cloud

Cloud instances require you to know your actual workload a bit more than you might on premise. Since graph databases can bottleneck anywhere, a general compute node with a reasonable amount of memory like an **m3.large** on AWS or a **n1-standard-2** on Google Compute Engine is a good starting point. 

An interesting thing to consider is you can use smaller nodes in a distributed system. It might be cheaper to use three small nodes rather than one large node. Graph databases may or may not allow sharding the data. If it does, take some time to experiment with distributing.

Storage is hard. Google provides essentially SAN disks as local storage for instances. If a server is terminated, the data will survive (unless you configured it to die with the instance). This is great for databases from a durability perspective. It's problematic from a performance perspective: you've got network latency. AWS' approach is to have actual physical drives locally, but they are transient. To quote, "data on instance store volumes is lost under the following circumstances: Failure of an underlying drive[, s]topping an Amazon EBS-backed instance[,] Terminating an instance. Therefore, do not rely on instance store volumes for valuable, long-term data"[^aws_bye_bye]. You'll need to replicate the data out of the local, fast storage into slower, and erratic S3 or EBS.

Softlayer[^softlayer], Linode[^linode] or Digital Ocean[^digital_ocean] might be better. You get performant instances or bare-metal servers all remotely racked. Baseline configurations are SSDs. Their storage is more expensive gigabyte for gigabyte than either AWS or Google. You do get arguably more with them since the guest OS sits on physical SSDs. **Quick note on pricing:** Linode (and probably the others) has more play in system configurations than their slick landing page lets on.  Once you create an account you can slide the storage allocation to a 2 GB machine up and down. The numbers they quote on their public site are for easy consideration (much like a value-meal at McDonald's). Like AWS and Google, they might have free ingress of data. Check the details for limitations.

## Further Resources

* For an in-depth look at graph theory, checkout the free ebook at http://graphdatabases.com. It's from the makers of Neo4J so you know it's got to be good.

* DZone's Amazingly Cool Graph DB Use Case Whiteboarding at http://java.dzone.com/articles/amazingly-cool-graph-db-use. A list of projects that experiment with the idea of graph storage.

[^wiki_graphs]: http://en.wikipedia.org/wiki/Graph_database#Graph_database_projects
[^neo_license]: http://neo4j.com/subscriptions/
[^neo_embed]: http://neo4j.com/docs/stable/tutorials-java-embedded.html
[^fb_giraph]: https://www.facebook.com/notes/facebook-engineering/scaling-apache-giraph-to-a-trillion-edges/10151617006153920
[^mapquest]: http://developer.mapquest.com/web/products/dev-services/directions-ws
[^aws_bye_bye]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html
[^softlayer]: http://www.softlayer.com/bare-metal-servers
[^linode]: https://www.linode.com
[^digital_ocean]: https://www.digitalocean.com