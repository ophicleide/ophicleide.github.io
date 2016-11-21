---
title: Architecture
link: architecture
---

The Ophicleide application suite is primarily composed of two microservice
containers:
[ophicleide-training](https://github.com/ophicleide/ophicleide-training) and
[ophicleide-web](https://github.com/ophicleide/ophicleide-web).

The ophicleide-training microservice is a Python and PySpark application that
receives training and query requests through a RESTful interface. It uses the
[Apache Spark](https://spark.apache.org) processing framework to perform
[Word2vec](https://en.wikipedia.org/wiki/Word2vec) calculations on text corpora
supplied by the user. The results of these calculations are stored as models in
a [MongoDB](https://www.mongodb.com/) database.

The ophicleide-web microservice is a [Node.js](https://nodejs.org/en/)
application that communicates with the ophicleide-training REST interface, and
provides a graphical interface to users through their web  browser using
[AngularJS](https://angularjs.org/) and
[PatternFly](http://www.patternfly.org/). It allows users to create Word2vec
training models by supplying URLs containing the source corpora to be
processed, and to make single word similarity queries against those models.

The following diagram shows the overall architecture of the Ophicleide suite.

![Architecture drawing](/img/architecture.png)

*Please note that the number of Spark containers is to demonstrate the use of a
processing cluster and not an exact number.*
