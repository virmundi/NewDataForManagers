# Key-Value Stores

## Starting with the Prokaryotes
Biology is a fascinating topic. Scientist deal with what is alive and what’s not. Viruses are some of the simplest forms of possible life. I would liken Key-Value stores to viruses, but that’s a loaded word in computer science. So let’s look to another fitting example from biology.

Bacteria are some of the simplest things out there. They are more complex than viruses. Bacteria have cell walls, cytoplasm and DNA. There are good bacteria and bad bacteria. As a result, the term is not as problematic as viruses. There are also many forms of bacteria. Prokaryotes are some of the simplest of the simple.

Prokaryotes lack a nucleus. They are, essentially, a bag of things. Each thing provides some benefit to the bacteria as a whole. Each thing has a name and a responsibility. Finding the responsibility is easy as long as you have the name. It’s a quick index lookup like in a catalogue. Finding the name from the responsibility requires reading through all of the responsibilities.

Key-value stores are essentially like bacteria. A K-V is the bacteria. The data within it is like the things. Depending on the complexity of the K-V store, you might get opaque insights into the values. You might get more complex data management like list and sets. Let’s look at the general architecture. Once we understand that, well look at the evolutionary changes found in the K-V family.

## General Architecture
### The Prokaryote
At its heart, a K-V act like a stand alone map or dictionary. Record items are added by inserting a value with a specific key. They are retrieved with the same key. They act like an old Rolodex.

Most of the time the key is a string. A string is series of bytes where each byte represents a character (unicode makes this a scoach more complex, but we’ll ignore it for now). The string is *hashed* into a number. Once hashed, the operation to insert a record into a map is really fast. If we’re using Big ‘O’ notation, inserting, finding and removing should all be a constant amount of time. The goal of a good hash algorithm is to create a number so unique that you will *probably* not have another item hash to the same number unless that other item is itself the actual key used to insert data.

The datastore is embedded. An application uses client API in its language to bind to the datastore. Depending on the embedded server, there may be only three real operations available: put, get and delete. Since the database is embedded it runs in the same memory space of the application using it.

Not all prokaryotic K-Vs are this simple. Some have more advanced features such as persistence or data structure support. As far as the K-V is concerned the *value* is nothing more than a bunch of bytes. The client might be putting in pictures, or WordDocs or anything else. The K-V thinks the user put in a blob (byte large object).

There are few benefits to this model. First, the code to support it is really simple. Even the big advanced servers in the space like Redis are only about 20k LOC. If a company using a K-V found a bug in the tool and they had a developer proficient in the K-V’s language, that company could probably patch the issue in a day. Second, they are really fast. Some of the servers boast a inserting a million records in around 1 second. Since we’re just hashing

### From Prokaryotes to Protozoans
Because the *value* is just a blog, querying is difficult on a prokaryotic K-V. The order of keys is not known. Blobs are just bytes, so the server can’t look for all instances of people whose first name starts with ‘Jo’. This moves the work onto the application and the developer. 

Due to such limitations, more advanced K-Vs arose from the programatic sludge. These tools allow for key ordering. This allows range searches against key entries. They make the content of the values semi transparent. For example the server might make it possible to store a list of blobs. It will also make simple operations available for the lists. You will probably see an “add” operation that requires a list name and a value to add.

Powerful abstractions are possible in this paradigm. They are more application focused than traditional RDMBS system, however. The developers will need to track things like changes and groupings manually.

Another evolutionary change is that K-V moves from embedded into a stand-alone server. The general idea is the same as the prokaryotic K-Vs with the minor exception that you have a bit more configuration to manage (what’s the IP/Port/Username/Etc). You also have to content with partition failures. Just as with a VS system that’s on a network, you might loose the network connection to your K-V. The benefit of server-based K-Vs is that you now can carve out a centralized location for data. You can make that server beefy so the K-V can keep most of the data in memory.

### Keeping the Values Around
Persistence is one of those complex things that more advanced bacteria have. Important information is stored in the nucleus. Having a nucleus is also a feature of the K-Vs. Some store their information. Some do not.

When evaluating how well a K-V might fit your application need, you need to question how much you need persistence as well as its type. You might find that you really only have transitory data. If you lost it at a minutes notice, you’d be fine. You might find that you need your data, but if you lost the last minute or so’s data you’d be okay. You might find that you really need the data to stick around in the case of a disaster. The K-V family has your back, but a particular instance might not. We’ll look at few of the major K-Vs as well as their characteristics.

### Multi-Cellular K-Vs
Many of the K-Vs out there are single server instances. Some allow redundancy by having a master-slave relationship. We’ve seen this before in the terminology chapter. This is where a primary datastore fails and a slave steps up to take its place.

In such a configuration, there is two or more nodes. When a client writes to a master, the write is passed along to one or more slaves. Ideally the write persists to both. If the master fails, the slave will pick up where the master left off.

How all of this happens is vendor specific. Make sure you check with the docs or the community to really understand how the K-V, if distributed, handles partitions and fail over.

## When Would I Use This?
While an inventive developer could use a K-V as the primary store for a complex e-commerce like Amazon.com (DynamoDB), most of the time a K-V is great for storing small, focused data for the application. Normally this is session data for web-based application or game.

The session is keyed off of something really unique like a user’s name or email address. Depending on the K-V type, the value is either an opaque blob (could be JSON or XML) or a complex data structure like a list or a map of values.

It can be useful in an enterprise setting to hold web session data so that you can properly handle failure of a web server. Let’s say you’re using a load balancer across three WebSphere servers. If  your application is using a session (it is if it’s old enough, like 2 years old), then your load balancer is probably using some sort of affinity. In this case if a user came into the session on Node 1, they will always return to Node 1. Even in a round robin balance, all the user return to their initial load server. All of their session state is on that *single* server.

If you were to lose a server at this point, whoops...the user’s session is lost. Now let’s be honest, many users’ sessions are lost. This can cause untold issues in your application. Probably a bunch of 500 errors, a few calls to Help Desk and bunch of re-logins. That’s if your lucky. The application might solider on with a corrupt/missing session and start to really mess up the database.

To prevent this, a K-V server can get quietly dropped in. It is beyond the scope of this book to detail how, but a simple Google search for “java web implement session management redis” brings up more than one GitHub project for Tomcat and Jetty connectors. The same is true for .NET with “iis web implement session management redis”. I’m sure that there is a similar extension for Python and Ruby (for Python you probably just “import workingSession” and you’re done). In the case of the Java servers the configuration should be fairly seemless. As a result it should be possible to integration test adding the feature in a day or two.

Since session is safely stored outside of the server, you can fail over your web applications without much concern. Depending on the implementation of the connector, any changes to the session are first written locally and then persisted through to the backing store as part of a transaction. Your developers won’t have to worry about the details. You’re OPs guys won’t have to work about a really chatty network. You won’t have to worry because you spent just $6k on two Redis servers setup in master-slave.

## Products in the Family
BerkleyDB is an embedded Key-Value store. It comes in several flavors. You can get it directly from the BerkleyDB project. You can also get it from Oracle in both C and Java builds. It's been around for 15+ years. It's battle tested. It is also under active development. Some flavors are more advanced. For example Oracle's version comes with an embedded SQLite database as well. 

Since the application is embedded it will run within the same memory space as your application. This means if you store 4 GB of data in the database, and the database decides to load all 4 GB into memory, your application profile be whatever your application uses plus the 4-6 GB of memory required by the database.

Redis[^redis] is a full featured, server based K-V. It provides multiple data structure formats, atomic counter (multi-client safe number incrementers) as well as master-slave replication. There are plugins for multiple platforms. Manning and other publishers have books available. The community is responsive to questions. Unlike BerkleyDB, if the server dies, it's possible to A) not kill the hosting application and B) failover to the slave. 



[^redis]: http://redis.io/
