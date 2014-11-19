# So Many Terms, So Little Time
One of the fun parts about researching all of these technologies is learning their underlying theory. Authors go on and on about “My Other Cap is Theorem." They talk about the difference between ACID and BASE. Everything is about being distributed. It’s enough to make one’s head spin. Even series of Master level courses discuss these topics. Some might want to skip the terms and just see the ROI. However, you'll benefit by examining these terms first. Understanding the ideas is both part of the fun and part of the foundation of larger concepts. It is hard for a manager or developer to fully grok a tool or paradigm without first obtaining at least a basic understanding of that paradigm’s terminology. Fortunately, the terms are often shared across paradigms.

## Point Zero - Cluster?
This is probably readily known, but should be stated for completeness, if nothing else. A cluster is a logical set of computers. You will probably call each computer a node. Normally you need two. However many of the NoSQL systems work just fine as a cluster with one node. You can add to it as time goes by. Depending on the implementation of the data store, you don’t even have to do much to get your data balanced and the whole system running smoothly across the nodes.

## I Don't Care Who Made It as Long as It Works
A common refrain you'll hear across the new data providers is "commodity hardware". These are server class computers. You should buy ECC RAM. The difference is that these boxes are a) inexpensive and b) interchangeable. They run standard linux. They don't carry special configurations like you'd see when using Oracle. Presently you can buy them in the 2k-5k range. It is certainly possible to run most of these systems on POS boxes sitting in a cubical (great for testing or boot strapping a startup). Just keep in mind that cheaper components break quicker.

One of the goals on these distributed designs is to allow a node to fail without an OPs representative having to scramble to fix it. For example, at Yahoo! if a normal nodes goes down in one of their Hadoop clusters, it gets fixed during a normal repair cycle. As a result, if your cluster is large enough, quality is less of a factor. You want to aim for mid-quality. Too high brings little value; too low brings major headaches. 

## Where’s My Data
The first thing that you’ll probably see in any discussion on most of these data stores is that these stores are distributed. You might have also seen it referred to as being horizontally scalable. During these conversations, you will have probably also seen a contrast to vertically scalable. Because of this, a reasonable place to start is with definitions for these terms.

A vertically scalable (VS) data store is one that’s improved by shoving more components in the box. This could mean adding more cores, more RAM, more storage or any combination thereof. This is the traditional model of database design. It’s worked well in the past. Moore’s Law[^moores_law] pretty much doubled computer performance every 18 months. This model gets to be fairly expensive given enough time. The effect will plateau. Soon you run out of RAM slots, drive bays and CPU sockets. You might end up with a really beefy box. Regardless, you end up with *a* beefy box. One network issue or motherboard issue and you're dead in the water.

Horizontal scaling (HS) looks to bring the cost down and extend the time till plateau. In a horizontally scalable data store, you don’t add RAM, drives or CPUs to get better performance, although doing so will still yield improvements. Instead you add commodity servers as you need to expand. By doubling the number of computers in your cluster (group of computers), you should see a near doubling of performance.

Let’s compare the two designs. In HS, your data is spread across (striped) the computers (nodes) in the cluster. Depending on how the particular data store does this, it’s possible to know exactly where the data is based on the distribution algorithm. This can make lookups really quick. In a VS, all of the data is in one place. Every RDMBS worth its salt has indexes. So like HS, if your query is for an index item, the lookup is really quick. If the data is not indexed, a HS system might have to query multiple systems at once. This makes the query take as long as the slowest node in the cluster. A VS will scan local files.

In a HS system if you lose a node, you lose, at worst, the information on that node[^free_beer]. Depending on how your data store works, you might not have even lost that. Most HS have the ability to fail over to a hot copy of the data on the master node. In a VS, if you lose the database server, you’re out of luck. 

Quick note: HS is not only in NoSQL/New SQL/Big Data. You can get the same ability in Oracle, SQL Server and DB2. The difference between the two is that NoSQL provides the benefits of HS out of the box and for low cost.

## Failover Beethoven Tell the Client the News
Let’s take a deeper look at failover. Failover is a way for the system to automatically switch to a different data source in the case of a failure. As stated most VS servers lack fail over. If you lose the server, you’re dead in the water until you revive it or get a new one. 

HS systems provide some form of failover, but you really have to look to see what form this takes. Some allow you to lose a master and still read data. All writes get rejected until the master is back up, but you’ve got partial availability. In another form, there is a “passive master”. This server is copying all the data in the master with each transaction. In the event of a master failure, the passive master steps in to service both reads and writes. Because the passive master was involved in all of the transactions, it is fairly consistent with the state of the master. You will also see some systems as being multi-master or master-master. In this instance there is no one system that is “true”. Each master can take writes. The writes are coordinated between the masters to synchronize the data. The benefit of such a system is higher throughput. It does bring issues when trying to understand which record is the “truth”.

Another point of failover is how the clients are updated. Many of the HA solutions have a client that knows about each node or at least a majority of the nodes in the cluster. If a node goes down, the client black lists the node. When the node returns to good health, the client corrects its list.

## When the Shard Hits the Fan, We Might Replicate That
Two terms that everyone must know is _shard_ or _sharding_ and _replication_. Often they are treated as if equivalent. They are not. They are orthogonal to each other. Sometimes you need one or both.

Sharding is when you split a large dataset across multiple nodes. Each data entry has a _shard key_. Often this is the Primary Key (PKs) to borrow from the RDBMS terms. The shard key identifies which of the nodes owns the data. As with PKs, _shard keys_ provide the fastest lookup mechanism for a particular document. Like PKs from their relational brothers, the key may be generated automatically or derived from the data itself.

A natural example of sharding is signing into some large event like school registration day or a conference. The reception tables are split into groups like last names A-F, G-O, P-Z. We've got three shards. Ideally this grouping handles about the same number of people per group to get the best throughput into the event.

Replication is the process of copying data to more than one place. Its the same data. Each copy is called a _replica_. Normally you want to have replicas in different nodes. While there is added cost in nodes and their respective storage, there are at least two benefits. The first is you've got a backup of the data. At fail over, one of the replicas can set in for the failed node without much interruption. The other benefit is throughput on reads. If the client and data store support it, the client can read from a replica either directly or via a proxy off of a main node. If a particular replica is busy serving a prior request, another server can happily respond to a new request. If your application is read heavy, replication like this might provide a good performance boost.

How the system replicates is a function of the data store. Some stores have all writes go to a single master. The master might concurrently write to replicas. It might write to itself and then to the replicas. It might concurrently write to itself and the replicas. The exact whys and wherefores impact data consistency. We'll cover consistency later in the chapter. Just keep an eye out the particulars when evaluating your needs and what the specific data stores provide.

## Election - I’m the President! No! I AM!
If  something happens to the President of the United States, there is a law that defines who gets the job next. Presently (2014) there are 16 possible slots with 2 slots unfilled. As per the law, vacant spots are skipped. This law is rather algorithmic. Each case is defined with a codified response. How much more important is your data? 

In the case of a master-slave system (even master-master where a master has slaves), a new master must be elected in the case of, say, a master dying from a blue tail fly. Each distributed system has its own means for doing this. Their documentation will describe the specific algorithm for election. Once a new master is elected, the clients to the system should redirect to the new master for all of the writes, if not all of the reads.

## Dropping ACID to Free[^note_on_management_nodes]-BASE
There are two terms here. The first is ACID. The second is BASE. 

ACID stands for Atomicity, Consistency, Isolation, and Durability. Atomicity means that all of the database parts involved in a transaction are changed or none are. Consistency means that the database cannot get into an illegal state. Isolation means that the partial effects of a transaction can be visible or hidden depending on what they user specifies. Finally Durability means that once the database said a change occurred, it sticks.

Most RDBMS’ are thought to be ACID compliant [^not_acid]. For most people ACID is _thought_ to be a requirement. Interestingly, most of the RDBMS do not actually provide ACID and the world still spins. 

BASE stands for Basically Available, Soft-State, Eventually Consistent. Basically Available means that some version of the data is available when requested. It’s not necessarily the true data, but it a version. Soft-State means that the data is not (necessarily) persisted to a permanent medium like a disk[^berk_pdf]. Eventually consistent means that the whole system will get into a good state given enough time (this could be a few milliseconds, vendor specific).

Let’s say you’ve got a person adding a item to their shopping cart. When they pay, the system has to decrement the number of items in your system’s inventory, create a purchase order, and possibly update the user’s financial information like adding a new credit card to their list of cards.

In an ACID world, the number of available items is locked when the user added the item to his or her cart. The lock holds until the user checkouts or removes the item from the cart. Assuming that the purchase order goes through, _isolation_ means presumes that only the whole order (header and lines) are visible to the larger system. Durable means that once you charge the credit card, the purchase is written to disk so you’re legally on the hook to provide the buyer with the items. Consistent means that any rules in the system are not violated (for example causing the available inventory of an item to go below zero). Atomic means that all of the above occurs or none of it does. You cannot partially fulfill an order.

In a BASE-ic world you don’t have these guarantees. For example, there is no item lock. So a person adds an item to their cart. You might decrement the number of items right then. That’s fine but you’re not consistent. It could be that the user abandons the cart. This means that you actually have one item available in the warehouse. The user may check out. You could have the purchase order visible to the system with only some of the lines added.

Now some people freak out about BASE. Truth is that it _might_ not be that big of a deal. Many enterprise applications don’t really leverage ACID because A) web servers make it hard to hold a transaction across page renders and B) developers are told to get and drop a database connection/transaction ASAP. So your applications in the field bringing in your monies might not really be locking and holding values as you might think. Another example is that banking systems aren’t ACID even though they are the pedagogic example of ACID. If a banking system truly checked values in both accounts during a transaction, we’d never have overdrafts. 

Picture what every database student first learns as the example of ACID. A person withdraws money from one account and deposits it into another. This is said to occur atomically, isolated, consistently and durably. In reality the account can go into an inconsistent state. This is called overdrawn. 

When you and your team look at systems that implement BASE over ACID, ask yourself do you really need ACID. It seems comforting at first. It seems natural because we’re all taught that it is right way. But then again, it’s a tool. Do you need this tool?

## And to CAP It All Off a Node Died!

Failure happens everyday. A powerful Oracle box suddenly goes offline due to a bad motherboard. Your web server, that faithful, old, beige box sitting in the closet, ground its last hard drive. Then there’s the always humorous accident where a guy accidentally sends a picture of himself dressed as a White Castle Slider to everyone in your multinational insurance company thereby bringing email down for all the agents and other company personnel including the VPs, VIPs and CEO because the picture was 2.58 MB and Exchange just couldn’t handle that load. Yep, failure happens.

Failure happens even more when you’re working in a shared/distributed system. Let’s say you’ve got a great system that has a slim chance of failing which means that it’s got a 99.9% chance of not failing. If you’ve got 40 nodes in a cluster you’ll have 3.9% chance that something will fail[^no_ca]. Now you’ve got to figure out how you’re going to react to failure. 

Fortunately the Failure Reaction Triangle exists just like the Project Management Triangle[^pm_triangle]. This triangle is CAP. C stands for Consistency. A is Availability. P is Partition tolerance (T is not capitalized because it would be the CAPT theory and NoSQL folks tend to be pacifists; I’m making this part up). Like the Project Management Triangle, you get to pick two. Unlike the Project Management Triangle, CA is not possible[^no_ca].

Consistency means that to an outside observer, like a database client, change events happen at single, logical point. This means that once a change is made to a record, all of the subsequent calls about that record reflect the change. 

Availability means that every request to a working node must be satisfied. If a client asks a node for a record on patient A, it has to return the record. If a working node tosses some sort of error from its side, the record is not considered available. Note: if a client sends an invalid request and the server simple returns a bad request error, the system is still actually available. The proper response to getting garbage is to say, “That was crap.”

Partition Tolerance deals with how the system works if one or more parts of the system can’t talk to each other. Specifically it’s concerned about how the system handles losing messages. If you’re looking at a system that says it doesn’t have to work with Partition Tolerance, you’ve got a system that doesn’t understand CAP or is one where there is no network. Anything else means the designers have bought one or more of the fallacies of distributed computing[^fallacy]. You should really look at another vendor.

Consistent systems react to partitioning different ways. Some might declare a snow day for the whole distributed system. It will reject all reads and writes just as if it were a VS system. It might allow only reads. Finally, it might allow updates based on the master data available in the currently “healthy” pool of nodes. 

You’ll need to have your team pay close attention to how the system figures out which are healthy and which are not. Let’s say you’ve got 4 nodes in a cluster. Two nodes are in one rack. Two nodes in the other. The network connection between them dies, but both subsets are accessible to some of the clients. How does the logical system determine which nodes to consider healthy? 

You might also hear the phrase “eventual consistency”. In this model, a system will allow copies of a record to become outdated. A client might not get the latest update because the change may not have percolated out to all the copied nodes. Often times such systems have quorum settings in their drivers. If they do, the client to the datastore will poll multiple nodes (this is the quorum). If X nodes come back with the same answer, the client will take that. 


[^moores_law]: http://en.wikipedia.org/wiki/Moore's_law
[^note_on_management_nodes]: This presumes that the data store doesn’t have a single management node. If it does, then you’re in as much trouble as in a Vertically Scalable system.
[^free_beer]: Free as in Beer, of course.
[^not_acid]: http://www.bailis.org/blog/when-is-acid-acid-rarely/
[^berk_pdf]: http://www.cs.berkeley.edu/~brewer/cs262b/TACC.pdf
[^no_ca]: http://codahale.com/you-cant-sacrifice-partition-tolerance/
[^pm_triangle]: http://en.wikipedia.org/wiki/Project_management_triangle
[^fallacy]: http://www.rgoarchitects.com/Files/fallacies.pdf