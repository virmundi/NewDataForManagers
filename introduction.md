# Welcome to the Technology Jungle

Historically picking a data store for a new application was a non-event. The paradigm was predetermined: relational database management system (RDBMS). The provider of the RDBMS was predetermined: startups used PostgreSQL or MySQL while many mid-to-large companies went with one of the Big 3 (Oracle, Microsoft or IBM). In fact, even the data modeling process was predetermined by years of experience within the RDBMS paradigm and the associated provider’s preference (like stored procs for many MS SQL Server jobs). There wasn’t any reason to question this. There were few alternatives and those weren’t cheap. Why chose something that is both exotic and expensive?

Times have changed. SQL has competitors (or companions as well see later) in the NoSQL space. Stresses like high data volumes, rapid change and developer preferences caused a flood of new paradigms. In their wake many are left confused as to the proper application of the technologies. New tech brings new terms. Sometimes the industry equivocates on terms (we prefer to call it “overloading”). Added to this is that each camp and even more granular each vendor within a camp swears that their product is the best thing since set theory. It becomes hard to tease out fact from marketing. That’s what this book proposes to do.

This book will provide a concise description and application of three major technologies related to data storage: NoSQL, New SQL and Big Data. You will learn the relative terms and theories from a 30,000 ft perspective. The goal is to give the reader enough information that they might better focus their research.

## What Did You Write This?
There are three reasons I undertook this project. First, I’ve always enjoyed watching the hipster culture from a far. These trend setters discussed the benefits of adopting some NoSQL tool long before it was cool. They extolled the virtues of a schema-less model. They revolved in the process of not having traditional ACID transactions. They seldom talked about how their new toys were a bad fit. The sheer fandom made these new ideas interesting.

Cooler heads tempered their exuberance. I got to see when there was a mismatch between problem spaces and implementation. I got to see how systems could be improved as told by random talking heads on the Internet at such websites like r/Programming or Hack News. Both sides of the argument got me really thinking about how distributed data storage could make life better/easier. They even improved my daily development against RDBMS’. 

Second, I was able to see what a non-RDBMS paradigm could do first hand. A few years back I was honored with the responsibility to extend the Medicare Fraud Detection System for the United States’ government. The challenge was “simple”: create a system that allowed the execution of an arbitrary number of fraud models that reads an arbitrary number of years worth of Medicare claims data. A given fraud model might look at just today’s data; it might look at 3 years worth of data. The models would also have to calculate an arbitrary number of statics to determine if there was fraud. Finally, all of these calculations need to run within a 24 hr cycle. Downtime is unacceptable. As a result the system has to run in under 24 hours in case one or more parts failed and require re-execution.

All of the traditional, rubber stamped data stores failed to meet these requirements. So we looked elsewhere. We found Hadoop. Hadoop is a product available from Apache labeled under Big Data. In fact it is _the_ pedagogic Big Data solution. We didn’t know Hadoop, but we dove in. Within 3 months (nice for a government job, huh?) we had a working fraud detection system. One component of the new system replaced one of the old. The old took 8 hours to execute. The new took just 45 minutes. We eventually got that down to 30 minutes.

Finally, there is a lot of hype and confusion round the topics of NoSQL, New SQL and Big Data. Hundreds of blog posts, hundreds of counter blog posts. Books and books on specific topics. There are, however, few general survey books on the paradigms. Of those (I found 2 or so), none are directed as a survey for semi-technical people or technical people wanted to get the gist of the technologies. I wanted a book I could give my team or management so we’d all be conversant in general ideas. If and when we decided to use a specific implementation of a tool, it would only be after we had discussed the other options. To properly discuss we all need a common reference.

## General Outline of the Book
The book is carved into three major portions. 

The first provides an introduction to the terms. This is where we’ll discuss topics like ACID, BASE, Master->Master, etc. Most of these terms apply to more than one of the paradigms. Rather than explaining them within a context of a particular paradigm, and thus force a reading order, they are all front loaded.

The second looks at NoSQL families. By my estimation there are 5 major families of NoSQL and some hybridized version as well. Those families are: Key-Value, Document, Columnar, Graph and Object databases. For each member we’ll discuss: 
* The architecture.
* Common use-case applications.
* Products in the family.
* Resources for future reading.

The third looks at New SQL and Big Data. The reason for grouping these two is due to the push to make Big Data more SQL friendly by both Apache and the folks at Concurrent and that NewSQL gives up some traditional SQL features in order to get the systems to scale.  

There isn’t a recommended order of reading. If you already feel comfortable with the general terms, skip it. Most people will want to at least skim the terms section. 