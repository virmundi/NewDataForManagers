# Graph Storage
Relational databases use set theory to model entities and their relationships.  Each entity is related to another via a column or set of columns. For example, person's home ownership might be model by a Person table with an id column, an Address table with an id column and a junction table with a PERSON_ID and ADDRESS_ID column. The junction table is the relation.

Graph databases at first glance look similar. There are entities and relationships. Users move from one to another using a query language. The difference lies in how relationships are modeled. The result is massive speedup in queries around relationships when compared to the relational model.

## Architecture

The core concept of a graph database is so simple as to be laughable. There are only two things involved in modeling: nodes and edges (or relationships depending on whose literature you read). A node is a just a container for properties like `name` or `birthdate`. A node can have as many properties as you need. An edge is the thing that connects to nodes. These are named concepts. Getting back to our home ownership above, one node would be "Patrick Davenport". Another is "The Lone Palm". The relationship of "Owns" links me to the house. 

Edges can have properties too. The "Owns" relationship might have the date I purchased the home. It could be marked as my primary house as apposed to my vast real-estate holdings. Edges also have a direction. I Own Lone Palm; the house doesn't own me.

The trick is storing the graph efficiently. Like RDBMS, you can have indexes on relationships, nodes and their properties.

## Getting to Know the Players

Wikipedia provides a list of over 20 products in the graph storage space [^wiki_graphs]. They run the gambit from GPL open-source, to open-sourced community edition with closed-source enterprise edition to full closed source/paid license edition. That is a lot to choose from. The selected projects cover the licensing and notoriety spectrum. 

### ArangoDB

### Neo4J

### InfiniteGraph

### Special Mention: Apache Giraph

## So How Would We Use This?

## Sizing and Cost Considerations

## Further Resources

* For an in-depth look at graph theory, checkout the free ebook at http://graphdatabases.com. It's from the makers of Neo4J so you know it's got to be good.

[^wiki_graphs]: http://en.wikipedia.org/wiki/Graph_database#Graph_database_projects