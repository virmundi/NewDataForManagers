# Welcome to the Technology Jungle

Historically, picking a data store for a new application was a non-event. The paradigm was predetermined: relational database management system (RDBMS). The provider of the RDBMS was predetermined: startups used PostgreSQL or MySQL while many mid-to-large companies went with one of the Big 3 (Oracle, Microsoft or IBM). In fact, even the data modeling process was predetermined by years of experience within the RDBMS paradigm and the associated provider’s preference (like stored procs for many MS SQL Server jobs). No one had reason to question this. Few alternatives existed, and those weren’t cheap. Why choose something that is both exotic and expensive?

Times have changed. SQL has competitors (or companions as we'll see later) in the NoSQL space. Stresses like high data volumes, rapid change and developer preferences caused a flood of new paradigms. In the wake of these new paradigms, many are left confused as to the proper application of the technologies. New tech brings new terms. Sometimes the industry equivocates on terms (we prefer to call it “overloading”). Added to this, each camp and, even more granular, each vendor within a camp swears that its product is the best thing since set theory. It becomes hard to tease out fact from marketing. This book proposes to do just that.

This book will provide concise descriptions and applications of three major technologies related to data storage: NoSQL, New SQL and Big Data. You will learn the relative terms and theories from a 30,000 ft perspective. The goal is to give readers enough information to better focus their research.

## Why Did You Write This?
There are three reasons I undertook this project. First, I’ve always enjoyed watching the hipster culture from afar. These trend setters discussed the benefits of adopting some NoSQL tools long before they were cool. They extolled the virtues of a schema-less model. They reveled in the process of not having traditional ACID transactions. They seldom talked about how their new toys were a bad fit. The sheer fandom made these new ideas interesting.

Cooler heads tempered their exuberance. I saw this when there was a mismatch between problem spaces and implementation. I saw how random talking heads on the Internet at such websites like r/Programming or Hack News thought systems could be improved. Both sides of the argument made me consider how distributed data storage could make life better/easier. They even improved my daily development against RDBMS’. 

Second, I was able to see what a non-RDBMS paradigm could do first hand. A few years back, I was honored with the responsibility of extending the Medicare Fraud Detection System for the United States’ government. The challenge was “simple”: create a system that allowed the execution of an arbitrary number of fraud models that reads an arbitrary number of years worth of Medicare claims data. A given fraud model might look at just today’s data or possibly at 3 years worth of data. The models would also have to calculate an arbitrary number of statics to determine if fraud existed. Finally, all of these calculations needed to run within a 24 hr cycle. Downtime was unacceptable. As a result the system had to run in under 24 hours in case one or more parts failed and require re-execution.

All of the traditional, rubber stamped data stores failed to meet these requirements. So we looked elsewhere. We found Hadoop. Hadoop is a product available from Apache labeled under Big Data. In fact it is _the_ pedagogic Big Data solution. We didn’t know Hadoop, but we dove in. Within 3 months (I know: blindingly quick for a government job) we had a working fraud detection system. One component of the new system replaced one of the old. The old took 8 hours to execute. The new took just 45 minutes. We eventually reduced that time to 30 minutes.

My third reason is simple. A lot of hype and confusion surrounds the topics of NoSQL, New SQL and Big Data, from hundreds of blog and counter blog posts to numerous books on specific topics. However, few general survey books are written on the paradigms. Of those (I found only a couple), none are constructed as a survey for semi-technical people or technical people who want to get the gist of the technologies. I wanted a book I could give my team or management so we’d all be conversant in general ideas. If and when we decided to use a specific implementation of a tool, it would only be after we had discussed the other options. To have a proper discussion, we all need a common reference.

## General Outline of the Book
The book is divided into three major sections. 

The first section provides an introduction to the terms. Here, we’ll discuss topics like ACID, BASE, Master->Master, etc. Most of these terms apply to more than one of the paradigms. Rather than explaining them within a context of a particular paradigm, and thus forcing a reading order, they are all front-loaded.

The second section looks at NoSQL families. By my estimation, NoSQL has five major families in addition to some hybridized versions. Those families are: Key-Value,  Columnar, Document, Graph and Object databases. For each member we’ll discuss: 

+ The architecture
+ Common use-case applications
+ Products in the family
+ Resources for future reading

The third section looks at New SQL and Big Data. These two are grouped together because of the push to make Big Data more SQL friendly by both Apache and the folks at Concurrent and because NewSQL gives up some traditional SQL features in order to get the systems to scale.  

There isn’t a recommended order of reading. If you already feel comfortable with the general terms, skip it. Most people, however, will want to at least skim the terms section. 
