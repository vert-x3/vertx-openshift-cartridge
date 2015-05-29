OpenShift Cartridge for Vert.x 3

This project provide a _cartridge_ for OpenShift to run Vert.x 3 applications. The _cartridge_ is based on a _fat jar_ approach, meaning that you provide a _fat jar_ embedding your application.

## Getting started

If you don't have an account, or aren't familiar with OpenShift you can go to [Getting Started with OpenShift](https://www.openshift.com/get-started/) to help guide you through setting up your environment.

Once our environment is setup we can create our first application (we'll call it demo) using the `rhc` client tools.

The cartridge is not available from OpenShift directly, so you need to pass a couple of parameters. First create your application with

```
rhc create-app demo https://raw.githubusercontent.com/cescoffier/vertx-cartridge/master/metadata/manifest.yml
```

To create an application that scale:

```
rhc create-app demo https://raw.githubusercontent.com/cescoffier/vertx-cartridge/master/metadata/manifest.yml -s
```

This will create a directory named `demo` which contain the Openshift template for your application. The cartridge does not provide any application by default.

## Deploy your application

Your application needs to be prepared to run on OpenShift. Your HTTP server must listen on the port and address given by the following system properties:

* `http.port` - the port on which your server need to bind
* `http.address` - the address on which your server need to bind

Here is an example:

```
vertx.createHttpServer()
    .websocketHandler(ws -> { ws.handler(ws::writeMessage);})
    .requestHandler(req -> {
            if (req.uri().equals("/")) req.response().sendFile("ws.html");
    }).listen(
        Integer.getInteger("http.port"), System.getProperty("http.address"));
```

Important to know is that your application is served on port 80/433 for HTTP, but web sockets are on ports 8000/8433. So you may need to change your web socket clients.

Once your code is ready, build a fat jar, and copy this `-fat.jar` file into the `/demo/application` directory created by `rhc`.

To deploy your application, just launch:

```
git add -A
git commit -m "deploy my application"
git push
```

Your application should now be running !

## Vert.x run options

All Vert.x run options are configured with the `RUN_ARGS` variable in the `configuration/vertx.env` file. For example if you want to specify the number of instances to deploy:

```
export RUN_ARGS="-instances 3"
```

## JVM Parameters and System Properties

You can configure the JVM parameters from the `configuration/vertx.env` file. These parameters are configured from the `VERTX_OPTS` variable. Here is an example:

```
export VERTX_OPTS="-Dfoo=bar"
```

## Retrieving the log and thread dump

You can retrieve the log file of your application using:

```
rhc tail demo -o "-n 200"
```

A thread dump can be generated using:

```
rhc threaddump demo
```

## Clustering

Clustering is supported when the application is scaled through OpenShift when you create your application `rhc create-app <my-app> <url> -s`.

Clustering is enabled from the `configuration/vertx.env` by setting:

```
export HAZELCAST_CLUSTERING=true
```

This generates the cluster metadata and provide a default `cluster.xml` file. However, if you need to customize this file copy the original [cluster](https://raw.githubusercontent.com/vert-x3/vertx-openshift-cartridge/initial-work/usr/shared/conf/cluster.xml) file to the `configuration` directory. The cartridge is going to replace the `${env.xxx}` variables.
