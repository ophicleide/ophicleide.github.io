---
title: Installation
link: installation
---

Installing the Ophicleide application suite is a relatively simple process
that mainly consists of creating a template in OpenShift and then launching
the application. There are a few prerequisites that must be accomplished
before Ophicleide can be launched.

In general, these steps can be summarized as:

1. Install prerequisities

2. Upload Ophicleide object list template

3. Launch Ophicleide

## Prerequisites

Ophicleide requires a few services to accomplish its processing: an Apache
Spark cluster, and a MongoDB datastore.

1. To install an Apache Spark cluster into your OpenShift project you can
  use a project like
  [oshinko-rest](https://github.com/radanalyticsio/oshinko-rest) to perform
  the deployment. You will need to record the Spark master URL from this
  deployment.

2. OpenShift includes a MongDB template in its projects, you can read more
  about it in the
  [OpenShift documentation on MongoDB images](https://docs.openshift.org/latest/using_images/db_images/mongodb.html).
  Be sure to create a database named `ophicleide` when you deploy the MongoDB
  as the ophicleide-training component will use this database for storing its
  models and query results. You will need to record the MongoDB connection URL
  from this installation.

## Ophicleide template

This template creates several objects in your OpenShift project. These
objects will direct the building and deployment of the entire application
suite.

For information on how to load this template into your OpenShift project,
please see the
[OpenShift documentation on templates](https://docs.openshift.org/latest/dev_guide/templates.html).

Let's examine the template and explore the various objects that are created.

<a href="{% link _documentation/ophicleide-setup-list.yaml %}" target="_blank">
ophicleide-setup-list.yaml <span class="glyphicon glyphicon-download" aria-hidden="true"></span>
</a>

This template encapsulates a `List` object which is used to bundle together
the various components necessary for building and deploying Ophicleide. The
first two items are
[Image Streams](https://docs.openshift.org/latest/architecture/core_concepts/builds_and_image_streams.html#image-streams),
which are used to store the results of building the ophicleide-training and
ophicleide-web images for the application.

```
- kind: ImageStream
  apiVersion: v1
  metadata:
    name: ophicleide-training
  spec: {}

- kind: ImageStream
  apiVersion: v1
  metadata:
    name: ophicleide-web
  spec: {}
```

After the `ImageStream` objects, the template defines two build configurations
which inform OpenShift how to perform a
[Build](https://docs.openshift.org/latest/architecture/core_concepts/builds_and_image_streams.html#builds)
of the related application components.

You will see that these `BuildConfig` objects each refer to a Git repository
containing the source code for the ophicleide-training and ophicleide-web
components. They also refer to the types of builds that will be performed and
the source Docker images to use as a starting point for those builds.

```
- kind: BuildConfig
  apiVersion: v1
  metadata:
    name: ophicleide-training
  spec:
    source:
      type: Git
      git:
        uri: https://github.com/ophicleide/ophicleide-training
    strategy:
      type: Docker
    output:
      to:
        kind: ImageStreamTag
        name: ophicleide-training:latest

- kind: BuildConfig
  apiVersion: v1
  metadata:
    name: ophicleide-web
  spec:
    source:
      type: Git
      git:
        uri: https://github.com/ophicleide/ophicleide-web
    strategy:
      type: Source
      sourceStrategy:
        from:
          kind: DockerImage
          name: centos/nodejs-4-centos7:latest
    output:
      to:
        kind: ImageStreamTag
        name: ophicleide-web:latest
```

The final section of the list defines the `Template` object that will be used
by OpenShift to display the application in the "Add to Project" section of
the console, or with the command line client. The
[OpenShift documentation on Templates](https://docs.openshift.org/latest/architecture/core_concepts/templates.html#architecture-core-concepts-templates)
provides an extended discussion of this type of object.

The template for Ophicleide defines two
[Services](https://docs.openshift.org/latest/architecture/core_concepts/pods_and_services.html#services),
a [Route](https://docs.openshift.org/latest/architecture/core_concepts/routes.html),
and a [Deployment](https://docs.openshift.org/latest/architecture/core_concepts/deployments.html).

The `Service` objects provide a useful way to expose the specific ports that
our application components need, and also define static names that can be used
as URIs within the project network.

The `Route` object associates a hostname with the service for the
ophicleide-web component's interface.

Finally, the `DeploymentConfig` instructs OpenShift how the containers of
our application should be deployed into our project. You will see that the
containers of this deployment will be based on the `ImageStreams` created
earlier, and that each container should be redeployed if either of those
images changes. You can also see how each container will need a few
environment variables and a port defined during their creation. These details
can be explored more fully by examing the source code for the Ophicleide
application components.

Finally, the `Template` contains a parameters section. This section instructs
OpenShift about variables that we may want to substitute in the final version
of the object. In the case of Ophicleide, there are 2 required and one
optional parameter. As noted earlier, the Spark master URL and MongDB
connection string are required for Ophicleide to run, the optional
`WEB_ROUTE_HOSTNAME` is used to define a custom route hostname for the
ophicleide-web component.

```
- kind: Template
  apiVersion: v1
  template: ophicleide
  metadata:
    name: ophicleide
  objects:

  - kind: Service
    apiVersion: v1
    metadata:
      name: ophicleide-web
    spec:
      ports:
        - protocol: TCP
          port: 8080
          targetPort: 8081
      selector:
        name: ophicleide

  - kind: Route
    apiVersion: v1
    metadata:
      name: ophicleide-web
    spec:
      host: ${WEB_ROUTE_HOSTNAME}
      to:
        kind: Service
        name: ophicleide-web

  - kind: DeploymentConfig
    apiVersion: v1
    metadata:
      name: ophicleide
    spec:
      strategy:
        type: Rolling
      triggers:
        - type: ConfigChange
        - type: ImageChange
          imageChangeParams:
            automatic: true
            containerNames:
              - ophicleide-web
            from:
              kind: ImageStreamTag
              name: ophicleide-web:latest
        - type: ImageChange
          imageChangeParams:
            automatic: true
            containerNames:
              - ophicleide-training
            from:
              kind: ImageStreamTag
              name: ophicleide-training:latest
      replicas: 1
      selector:
        name: ophicleide
      template:
        metadata:
          labels:
            name: ophicleide
        spec:
          containers:
            - name: ophicleide-web
              image: ophicleide-web:latest
              env:
                - name: OPHICLEIDE_TRAINING_ADDR
                  value: "127.0.0.1"
                - name: OPHICLEIDE_TRAINING_PORT
                  value: "8080"
                - name: OPHICLEIDE_WEB_PORT
                  value: "8081"
              ports:
                - containerPort: 8081
                  protocol: TCP
            - name: ophicleide-training
              image: ophicleide-training:latest
              env:
                - name: OPH_MONGO_URL
                  value: ${MONGO}
                - name: OPH_SPARK_MASTER_URL
                  value: ${SPARK}
              ports:
                - containerPort: 8080
                  protocol: TCP

  parameters:
    - name: SPARK
      description: connection string for the spark master
      required: true
    - name: MONGO
      description: connection string for mongo
      required: true
    - name: WEB_ROUTE_HOSTNAME
      description: The hostname used to create the external route for the ophicleide-web component
```

## Launching Ophicleide

With the Ophicleide objects loaded into your project, you are now ready to
begin the process of building and launching the application suite. Before
the Ophicleide components can be started though, their images must be built
and tagged as image streams in the project.

Previously, the `ImageStream` objects were created to provde a location within
the project to store the built applications. Now you must build the
ophicleide-training and ophicleide-web images. This can be done by navigating
to the build section in the web console or by using the command line. For a
thorough discussion of starting a build, please see the
[OpenShift documentation on builds](https://docs.openshift.org/latest/dev_guide/builds.html#starting-a-build).

**Note** to complete the builds within your project, you will need to have
the `system:image-pusher` role on your account.

The build time for these images should be under 5 minutes, assuming there
are no connection issues. Information about the build process can be seen by
accessing the logs of either build.

With both images successfully built, you are now ready to launch the entire
application suite. As mentioned previously, you will need two pieces of
information to complete the launch: the Spark master URL, and the MongoDB
connection string.

Ophicleide can be lauched by navigating to the "Add to Project" section of
your project, and then searching for `ophicleide` in the provided form. You
should see a screen similar to this:

<img src="/img/addtoproject.png" class="img-responsive">

Selecting the Ophicleide template will bring you to the following screen
which will allow the input of our connection strings and the actual launch:

<img src="/img/launch.png" class="img-responsive">

You should now fill in the forms for the Spark master URL and the MongoDB
connection string, you may optionally add a route hostname. By default,
OpenShift will use a preconfigured value for the hostname of the route. It
will be determined by using the application name, project name, and a value
configured by the site administrator for the domain name of the OpenShift
installation.

With everything filled in, you may now click the "Create" button and your
application pods should start launching.

For extended discussions on creating objects through templates, please see
the following OpenShift documents:
[Creating from Templates Using the Web Console](https://docs.openshift.org/latest/dev_guide/templates.html#creating-from-templates-using-the-web-console)
and
[Creating from Templates Using the CLI](https://docs.openshift.org/latest/dev_guide/templates.html#creating-from-templates-using-the-cli).
