#Columnar Stores

Of all the NoSQL families, Columnar Storage is the most RDBMS-ish. There are rows and there are columns. The database knows about column types. The key difference is how one has to relearn what a row is.

Columnar stores have the reputation of being obtuse. I think this stems from their general appearance. It took me multiple tries to understand how they actually work. My issue, and I think a lot of people's, is approaching a columnar store as a table store in the RDBMS sense. You really have to suspend this mapping.

Suspended? Good. Let's look at the system.

## Architecture
Suspending false equivalences is not the same as forgetting the prior model. Most RDBMS are row-centric. An entity is the table. A user is the `USER` table. Read and write operations occur against the entire row. There are three ramifications.

First, let's say you have a demographic table with 100 columns about your users. You want to know the average age of all the men that purchased an item within the last six months. You only need 3 columns of data: `GENDER`, `BIRTHDATE`, and `LAST_PURCHASED_DT`. Because it's row-centric, the database will actually read in all 100 columns, discard 97 and calculate against the three. That is a lot of wasted I/O. Second, every row has to have 100 columns. This causes wasted space due to nulls[^oracle_nulls].  Third, it also misrepresents the state of the data from a modeling perspective. A user might only provide a subset of fields. The null fields make the user look incomplete. In reality the user is not incomplete, but a different type of user stuck in a fixed model.

Now that we've outlined the relational database storage model. Let's switch to the columnar model.

### Building Blocks
The smallest unit of storage is a column. 

A column has three pieces of data: column name, column value, and column timestamp. A value is typed like `INTEGER` or `STRING`. Of these the timestamp is the most interesting. The system determines the data's age from the timestamp. Age helps the system determine conflict resolution like the most recent write wins.

Columns are grouped into column families. A family has a row key. Depending on the actual system used, you can have either relatively static or dynamic columns within the family. 

By "relatively static" I mean that a column family can say, "I have these possible columns, but not every row in the family provides them all. They just can't add extras." Dynamic means just that: no fixed schema. Dynamic columns enable you to essentially create pre-calculated rows just as you would with a materialized view in RDBMS land. Let's say you have a project submission  request with comments. A dynamic row might have the submitted document, its meta-data for the evaluation process and each comment by the reviewers represented by the dynamic columns where the column name is the reviewer id and the column value is the comment.

Super column families are like a view over multiple column families. If you need to analyze two or more column families at once, you should look at using super columns. **Quick Note:** super column families might be deprecated in some engines; RTFM before committing to a modeling paradigm.

### Storage and Distribution

Column families are written in contiguous chunks on the hard drive. This allows the database to optimize read operations by grabbing a large number of records per read cycle. 

Every system leverages replication at a column family level. When one defines a column family, the number of replicas is specified, or a default, like three, is used. 

At this point our friend the CAP theorem comes in. Some systems choose CP. Others will choose AP. Some systems allow you dial up or down your needs between the two extremes. This is a quorum. Such systems allow you to poll the replicas to see what the common answer is. If two nodes say a row is "X, Y, Z" while a third says "X,C,B' and the quorum setting is 50% or more, the client will get a row of "X,Y,Z". 

The more nodes involved in the quorum, the slower the read operation because it has to wait for each node's response. If the quorum is set to one node, that puts you squarely in the AP camp. If the quorum is set to all the nodes, you're in the C camp because you can't necessarily survive a network partition if one of your replicas is on the other side of the partition. (Quick note, all nodes only means all of the nodes that have a replica of the data you're looking for.)


### Joins
There are no joins in such systems. Columns can point to other columns in other tables. It's up to the application to perform another query to get the information from the second table. At first blush this appears limiting. In practice, denormalization is your friend. Rather than storing information in another table, duplicate data into a column family. This gives you essentially materialized views in the row itself.

## Getting to Know the Players
Google's BigTable pushed this concept. They've used it with multiple products like Google Earth and Google Finance[^bigtable_google]. Others have used it as well. It's part of the PaaS offerings by Google[^google_cloud_data]. None one has used it outside of the Google server environment. People liked the idea though, so they worked to replicate it.

### Cassandra
The homepage is http://cassandra.apache.org/. 

Cassandra is a top-level project from Apache by way of Facebook. It is living largely in the AP camp. 

In 2008 Facebook was looking for a way to build out its data collection infrastructure.  Their solution was a hybrid between BigTable and Amazon DynamoDB. A little while after creating the first pass of Cassandra, Facebook pretty much abandoned the project in favor of HBase. Fortunately, large companies like Apple and Netflix picked up where Facebook left off[^fb_abandon].

Cassandra is a complex, powerful storage system. It consistently wins performance shootouts focused on I/O speed[^cassandra_shootout]. It is eventually consistent. Quorums allow the developer to strike the right balance between consistency and speed.  Unlike other NoSQLs, there is no single point of failure in the entire system. It uses a gossip protocol to figure out the structure of the cluster.  

A gossip protocol starts with a set of known nodes. The new node calls up the known nodes to see how they're doing. Those nodes say, "I'm fine. Have you met Ted?" Ted is then added to the list of possible node. If the node's friends are unavailable, it learns to ignore them until contacted later. 

Below are some features that make it stand out.

* Adjustable quorum size.
* Secondary indexes.
* True linear scaling (2 boxes run twice as fast as 1).
* Resiliency during node failures.


If you're looking for a Bat Phone to a support company, never fear, DataStax[^datastax] is here. Recently called a Visionary[^gartner_datastax] by Gartner, they are the premier Cassandra consulting group.

### HBase

The homepage is http://hbase.apache.org/.

HBase is another Apache top-level project. Originally developed by Powerset, it's now used by multiple Fortune 500 companies. Notably Facebook built its messaging platform on it.

HBase sits atop Hadoop's HDFS. While Hadoop gets special treatment later in the book, now is a fine time to take a sneak peak. HDFS is Hadoop's distributed file system. HBase only writes data to HDFS (excluding the local drive). Under the covers HDFS takes care of replication and all of the other bookkeeping associated with tracking data.

Below are some features that make it stand out. 

* Easier Hadoop integration. HBase carves out its own little fiefdom in HDFS. You still have to ETL in and out of it.
* Bloom filters. Quickly tells if a thing is *not* in a set of data. False positives are possible. False negatives are not.
* Advanced in-memory operations.
* Simpler development model when compared to Cassandra.

If you're looking for a Bat Phone to a support company, never fear, Hortonworks[^hortonworks] is here. Hortonworks has the largest number of contributors to the core Hadoop space than any other company. If you want to use HBase or any other Apache tool to the fullest by working with a great partner, I know from personal experience, Hortonworks is the way to go.

## So How Would We Use This?

### Rapid Data Landing
Systems create more data per year than ever existed in the prior years combined. Sensors on an assembly line can spew all sorts of wonderful snippets of data 24/7. Enterprise web apps can monitor real-time mouse movements to see where which screen has the most mouse movements or longest pauses. Twitter and Facebook make data fall like the rain.

All of this data can come at your systems like a flood. Traditional databases, even large Oracle installations, fail to keep up. Often this is due to both the volume of the data as well as the shape of the data. Columnar stores are geared to just this type of ingest. 

### Simple, Integrated Data Centers
Data warehousing is an arduous task. Multiple systems have to push their information to the warehouse. One or more ETL processes massage the data, denormalize it when needed and land it. Systems don't have the same sync windows. Often correcting system failures is a manual process that's fraught with data and productivity loss.

Products like Cassandra are a data warehouse in a box. You can use them as an OLAP and OLTP center. Analytics can run against one specific replica. Real-time transactions can run against the other replicas. These replicas can be stored between two data centers. Its eventually consistent model allows data to flow freely between centers.

## Sizing and Cost Considerations

When considering sizing and cost one must look to Cloud vs Local. We'll first look at sizing locally within a company. Then we'll look at what major companies provide by way of cloud hosting.

### On Premises
Sizing depends on which system you're going with. HBase shares resources on a Hadoop cluster. Cassandra often runs on its own dedicated cluster.

The folks at DataStax provide a thorough breakdown of what you should get for Cassandra. You can find it in the Further Resources section below. I'll highlight the their recommendations.

First, as with every other storage system out there, the more memory you have the better. Aim for 16 to 32 GB per node with the bare minimum of 8 GB for a production box.

Second, Cassandra is surprisingly CPU bound. Its tweaked I/O to the point where it spends more time hashing to find records than waiting for data.

Third, get SSDs. They provide great random reads while allowing reasonable block write performance.  Even if you go with HDDs, avoid raid. Both HBase and Cassandra work in contiguous writes so you won't get much from RAID. You also have replicated data on other nodes so you don't need RAID for that either.

### In the Cloud

If you're going to deploy to AWS, your opening move is to boot up an m3.large, r3.large or ix.large. Any one of these models has local SSDs, 7.5 GB of RAM and decent CPUs. The difference between them is the bottleneck you're optimizing for: memory, CPU or disk. The lower class servers are fine for experimentation and evaluation. You don't want to walk into production with them though. They either lack CPU, physical local disks or memory.

Keep in mind that AWS local disks are presently transient. If the server stops or terminates, you will loose all of the information. You will have to pay for EBS backup storage as well as plan backup processes. This is somewhat mitigated by the replication process of a columnar store. If you trust that you've got enough nodes to survive a collapse, your data should be fine. This is especially true if you put one of your replicas in another region.

Linode and Digital Ocean offer SSD backed units with 8GB or more for around $80/month. 

Google's pricing model is a bit denser than AWS or Linode. Start with an n1-standard-2 or with the n1-highmem-2. You can provision local SSDs at $0.22-ish/GB.

## Further Resources
Below are some links to interesting topics for continued reading. 
* Secondary indexes in Cassandra - http://www.wentnet.com/blog/?p=77
* HBase Official Free Book - http://hbase.apache.org/book/book.html
* DataStax on sizing - http://www.datastax.com/documentation/cassandra/1.2/cassandra/architecture/architecturePlanningAbout_c.html
* Cassandra on Google's Cloud - https://cloud.google.com/solutions/cassandra/
* DataStax's Guide for AWS - http://www.datastax.com/documentation/cassandra/2.0/cassandra/install/installAMI.html
* Google's BigTable Paper - http://research.google.com/archive/bigtable.html

[^bigtable_google]: http://research.google.com/archive/bigtable.html
[^oracle_nulls]: https://community.oracle.com/thread/855964
[^google_cloud_data]: https://cloud.google.com/datastore/
[^fb_abandon]: https://www.facebook.com/note.php?note_id=454991608919
[^cassandra_shootout]: http://planetcassandra.org/what-is-apache-cassandra/
[^datastax]: http://www.datastax.com/
[^gartner_datastax]: http://www.datastax.com/2014/10/gartner-names-datastax-a-visionary-in-the-2014-magic-quadrant-for-operational-database-management-systems
[^hortonworks]: http://hortonworks.com/hadoop/hbase/