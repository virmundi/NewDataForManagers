# So Many Terms, So Little Time
One of the fun parts about researching all of these technologies is learning their underlying theory. Authors go on and on about “My Other Cap is Theorem." They talk about the difference between ACID and BASE. Everything is about being distributed. It’s enough to make one’s head spin. Even series of Master level courses discuss these topics. Some might want to skip the terms and just see the ROI. However, you'll benefit by examining these terms first. Understanding the ideas is both part of the fun and part of the foundation of larger concepts. It is hard for a manager or developer to fully grok a tool or paradigm without first obtaining at least a basic understanding of that paradigm’s terminology. Fortunately, the terms are often shared across paradigms.

## Point Zero - Cluster?
This is probably readily known, but should be stated for completeness, if nothing else. A cluster is a logical set of computers. Normally you need two. However many of the NoSQL systems work just fine as a cluster of one. You can add to it as time goes by. Depending on the implementation of the data store, you don’t even have to do much to get your data balanced and the whole system running smoothly across the computers.

## Where’s My Data
The first thing that you’ll probably see in any discussion on most of these data stores is that these stores are distributed. You might have also seen it referred to as being horizontally scalable. During these conversations, you will have probably also seen a contrast to vertically scalable. Because of this, a reasonable place to start is with definitions for these terms.

A vertically scalable (VS) data store is one that’s improved by shoving more components in the box. This could mean adding more cores, more RAM, more storage or any combination thereof. This is the traditional model of database design. It’s worked well in the past. Moore’s Law<sup>1</sup> pretty much doubled computer performance every 18 months. This model gets to be fairly expensive given enough time. The effect will plateau. Soon you run out of RAM slots, drive bays and CPU sockets.

Horizontal scaling (HS) looks to bring the cost down and extend the time till plateau. In a horizontally scalable data store, you don’t add RAM, drives or CPUs to get better performance, although doing so will still yield improvements. Instead you add commodity servers as you need to expand. By doubling the number of computers in your cluster (group of computers), you should see a near doubling of performance.

Let’s compare the two designs. In HS, your data is spread across (striped) the computers (nodes) in the cluster. Depending on how the particular data store does this, it’s possible to know exactly where the data is based on the distribution algorithm. This can make lookups really quick. In a VS, all of the data is in one place. Every RDMBS worth its salt has indexes. So like HS, if your query is for an index item, the lookup is really quick. If the data is not indexed, a HS system might have to query multiple systems at once. This makes the query take as long as the slowest node in the cluster. A VS will scan local files.

In a HS system if you lose a node, you lose, at worst, the information on that node<sup>3</sup>. Depending on how your data store works, you might not have even lost that. Most HS have the ability to fail over to a hot copy of the data on the master node. In a VS, if you lose the database server, you’re out of luck. 

Quick note: HS is not only in NoSQL/New SQL/Big Data. You can get the same ability in Oracle, SQL Server and DB2. The difference between the two is that NoSQL provides the benefits of HS out of the box and for low cost.

## Failover Beethoven Tell the Client the News
Let’s take a deeper look at failover. As stated most VS servers lack fail over. If you lose the server, you’re dead in the water until you revive it or get a new one. 

HS systems provide some form of failover, but you really have to look to see what form this takes. Some allow you to lose a master and still read data. All writes get rejected until the master is back up, but you’ve got partial availability. In another form, there is a “passive master”. This server is copying all the data in the master with each transaction. In the event of a master failure, the passive master steps in to service both reads and writes. Because the passive master was involved in all of the transactions, it is fairly consistent with the state of the master.

Another point of fail over is how the clients are updated. Many of the HA solutions have a client that knows about each node or at least a majority of the nodes in the cluster. If a node goes down, the client is updated to black list the node. An algorithm then updates the client in the future when the node is healthy.

## Dropping ACID to Free<sup>2</sup>-BASE
There are two terms here. The first is ACID. The second is BASE. 

ACID stands for Atomicity, Consistency, Isolation, and Durability. Atomicity means that all of the database involved in a transaction are changed or none are. Consistency means that the database cannot get into an illegal state. Isolation means that the partial effects of a transaction can be visible or hidden depending on what they user specifies. Finally Durability means that once the database said a change occurred, it is really occurred.

Most RDBMS’ are ACID compliant (at least logically <sup>4</sup>). For most people ACID is _thought_ to be a requirement. Interestingly, most of the RDBMS do not actually provide ACID and the world still spins. 

BASE stands for Basically Available, Soft-State, Eventually Consistent. Basically Available means that some version of the data is available when requested. It’s not necessarily the true data, but it a version. Soft-State means that the data is not (necessarily) persisted to a permanent medium like a disk<sup>5</sup>. Eventually consistent means that the whole system will get into a good state given enough time (this could be a few milliseconds, vendor specific).

Let’s say you’ve got a person adding a item to their shopping cart. When they pay, the system has to decrement the number of items in your system’s inventory, create a purchase order, and possibly update the user’s financial information like adding a new credit card to their list of cards.

In an ACID world, the number of available items is locked when the user added the item to his or her cart. The lock holds until the user checkouts or removes the item from the cart. Assuming that the purchase order goes through, _isolation_ means presumes that only the whole order (header and lines) are visible to the larger system. Durable means that once you charge the credit card, the purchase is written to disk so you’re legally on the hook to provide the buyer with the items. Consistent means that any rules in the system are not violated (for example causing the available inventory of an item to go below zero). Atomic means that all of the above occurs or none of it does. You cannot partially fulfill an order.

In a BASE-ic world you don’t have these guarantees. For example, there is no item lock. So a person adds an item to their cart. You might decrement the number of items right then. That’s fine but you’re not consistent. It could be that the user abandons the cart. This means that you actually have one item available in the warehouse. The user may check out. You could have the purchase order visible to the system with only some of the lines added.

Now some people freak out about BASE. Truth is that it _might_ not be that big of a deal. Many enterprise applications don’t really leverage ACID because A) web servers make it hard to hold a transaction across page renders and B) developers are told to get and drop a database connection/transaction ASAP. So your applications in the field bringing in your monies might not really be locking and holding values as you might think. Another example is that accounting systems aren’t ACID even though they are the pedagogic example of ACID.

Picture what ever database student first learns as the example of ACID. A person withdraws money from one account and deposits it into another. This is said to occur atomically, isolated, consistently and durably. In reality the account can go into an inconsistent state. This is called overdrawn. 

When you and your team look at systems that implement BASE over ACID, ask yourself do you really need ACID. It seems comforting at first. It seems natural because we’re all taught that it is right way. But then again, it’s a tool. Do you need this tool?

## My other CAP is a Theorem<sup>6</sup>


1 - http://en.wikipedia.org/wiki/Moore's_law
2 - This presumes that the data store doesn’t have a single management node. If it does, then you’re in as much trouble as in a Vertically Scalable system.
3 - Free as in Beer, of course.
4 - http://www.bailis.org/blog/when-is-acid-acid-rarely/
5 - http://www.cs.berkeley.edu/~brewer/cs262b/TACC.pdf
6 - http://learnyousomeerlang.com/distribunomicon#my-other-cap-is-a-theorem
