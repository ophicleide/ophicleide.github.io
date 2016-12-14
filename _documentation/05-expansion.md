---
title: Expansion
link: expansion
---

Although Ophicleide is functional and performs the tasks it was designed for,
there is always room for improvement and expansion. The following are a few
ideas for how Ophicleide could be expanded. These are suggested as possible
exercises for the reader and as a starting point to discuss how this type of
application can evolve.

1. Use Spark to process the queries. Currently, the vectors associated with
  each processed word are stored in a dictionary that the Ophicleide training
  component uses to return query results. There are facilities in the Word2vec
  package to use a Spark context for processing these type of searches.
  Adding this functionality would allow for the lookup workload to be taken
  off the training component, and provide a platform for deeper introspection
  of query results.

2. Separate the query engine into a service. A prominent consideration when
  designing cloud native applications is scale. How will an application grow
  to accomodate larger user bases. In the case of Ophicleide, separating out
  the query engine into a service of its own would give a graceful path to
  growth. By creating a new service specifically for queries it will become
  easier to add horizontal scalability by identifying the portions of the
  application which are being used the most and then replicating them.
