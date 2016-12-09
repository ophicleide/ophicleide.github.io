---
title: Usage
link: usage
---

Now that Ophicleide is running in your project it is time to begin training
models and executing queries against those models.

To begin with, you will need to navigate to the main web page for Ophicleide.
On the "Overview" page of you project, you will see a header for the
Ophicleide pod that should look similar to the following image:

<img src="/img/route.png" class="img-responsive">

_(Note, your route hostname should be different)_

Clicking on that link will take you to the landing page for the ophicleide-web
component. This page displays the training models that are available to run
queries against. As no models have been trained yet, it should be empty and
look like this:

<img src="/img/usage1.png" class="img-responsive usage">

To start training a model, click on the "Train Model" button. This will bring
up a dialog where you will enter the name of the model and the URLs
containing the source text corpora. Here is an example with the modal dialog
filled out:

<img src="/img/usage2.png" class="img-responsive usage">

Click on the "Train" button in the dialog to begin the process of training a
Word2vec model against the source text corpora. After starting the training
your models page will change to look like the following image, with the
exception that your status will be "training". When the model training is
complete, the status will change to "ready".

<img src="/img/usage3.png" class="img-responsive usage">

If you would like to verify that the ophicleide-training component is
running the Word2vec processing, you can use the OpenShift console to navigate
to the Pod view associated with Ophicleide and inspect the logs for the
ophicleide-training container. You should see something similar to the
following in the output:

<img src="/img/logs.png" class="img-responsive">

When the model status is "ready", you can click on the "Create Query" button
to initiate a word query against that model. Enter a word that you would like
to find synonyms for within the corpus, and then click the "Query" button.

<img src="/img/usage4.png" class="img-responsive usage">

After clicking the "Query" button, the page view will change and you will
now be looking at the queries page. This page shows all the word queries
that have been run and the top 5 results in each query. You will notice
that each result in the query contains the similar word as well as the
vector associated with that word.

<img src="/img/usage5.png" class="img-responsive usage">

If you would like to start another query, you can now use the "Create Query"
button on this page. As previously, you will enter a word to search for
similarities, and since we are now searching from the queries page you will
need to select the model to query against using the model select drop-down.
