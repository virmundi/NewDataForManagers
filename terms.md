# So Many Terms, So Little Time
One of the fun parts about researching all of these technologies is learning their underlying theory. Authors go on and on about “My Other Cap is Theorem." They talk about the difference between ACID and BASE. Everything is about being distributed. It’s enough to make one’s head spin. Even series of Master level courses discuss these topics. Some might want to skip the terms and just see the ROI. However, you'll benefit by examining these terms first. Understanding the ideas is both part of the fun and part of the foundation of larger concepts. It is hard for a manager or developer to fully grok a tool or paradigm without first obtaining at least a basic understanding of that paradigm’s terminology. Fortunately, the terms are often shared across paradigms.

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

## Dropping ACID to Free<sup>2</sup>-BASE

1 - http://en.wikipedia.org/wiki/Moore's_law
2 - This presumes that the data store doesn’t have a single management node. If it does, then you’re in as much trouble as in a Vertically Scalable system.
3 - Free as in Beer, of course.
