# mongodb-metrics-reporter

[![Build Status](https://github.com/keawe-software/mongodb-metrics-reporter/workflows/Java%20CI%20with%20Maven/badge.svg)](https://github.com/keawe-software/mongodb-metrics-reporter/actions?query=workflow%3A%22Java+CI+with+Maven%22) [![Apache License](http://img.shields.io/badge/license-APACHE2-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0.html) ![maven status](https://img.shields.io/maven-central/v/de.keawe.metrics/mongodb-metrics.svg)

This project provides a reporter for CodaHale/DropWizard Metrics library to store the metric data in MongoDB.

## How does it work?

1. Define MetricsRegistry which is a JVM wide singleton for metric data
2. Metrics Reporter is associated with the Metrics Registry
3. Application services are instrumented with DropWizard Metrics
4. The reporter reads the data from registry at fixed defined interval and writes this data to the MongoDB datastore is the relevant collection

![MongoDB-Metrics](doc/images/MetricsMongoDB.png)

## Build Requirements:

* Apache Maven version 3.x
* Java8

## Building

Clone the repository using command

```bash
git clone git://github.com/aparnachaudhary/mongodb-metrics-reporter.git
```

To build, issue this from the command line

```bash
mvn clean install
```

## Dependencies

The module has following compile time dependencies

* io.dropwizard.metrics:metrics-core:jar:3.1.2
* org.mongodb:mongo-java-driver:jar:3.1.1
* org.slf4j:slf4j-api:jar:1.7.13

## Usage

Add the following dependency to your project's Maven POM file.

```xml
    <dependency>
        <groupId>io.github.aparnachaudhary</groupId>
        <artifactId>mongodb-metrics</artifactId>
        <version>${version.mongodb-reporter}</version>
    </dependency>
```

Usage in Java code:

```java
        // create metric registry
        MetricRegistry metricRegistry = new MetricRegistry();
        // register JVM metrics
        metricRegistry.register("jvm.attribute", new JvmAttributeGaugeSet());

        MongoDBReporter reporter = MongoDBReporter.forRegistry(metricRegistry)
                .serverAddresses(new ServerAddress[]{new ServerAddress("192.168.99.100", 32768)})
                .withDatabaseName("javasedemo")
                .prefixedWith("javase")
                .build();
        // Report metrics every 5 seconds
        reporter.start(5, TimeUnit.SECONDS);

        // register metric
        metricRegistry.counter("demo.counter").inc();

        // sleep for 10 seconds so that metric is reported to MongoDB store
        try {
            TimeUnit.SECONDS.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
```

Executing the above program creates two collections (counter and gauge) in javasedemo database.

*Counter collection:*

```json
/* 0 */
{
    "_id" : ObjectId("565b3b7ec8f3530f3a327001"),
    "name" : "javase.demo.counter",
    "count" : 1,
    "timestamp" : ISODate("2015-11-29T17:53:02Z")
}

/* 1 */
{
    "_id" : ObjectId("565b3b83c8f3530f3a327005"),
    "name" : "javase.demo.counter",
    "count" : 1,
    "timestamp" : ISODate("2015-11-29T17:53:07Z")
}
```

*Gauge collection:*

Since we also registered the _JvmAttributeGaugeSet_; JVM metrics are registered in the _gauge_ collection.

```json
/* 0 */
{
    "_id" : ObjectId("565b3b7ec8f3530f3a326ffe"),
    "name" : "javase.jvm.attribute.name",
    "value" : "69434@server.local",
    "timestamp" : ISODate("2015-11-29T17:53:02Z")
}

/* 1 */
{
    "_id" : ObjectId("565b3b7ec8f3530f3a326fff"),
    "name" : "javase.jvm.attribute.uptime",
    "value" : 5711,
    "timestamp" : ISODate("2015-11-29T17:53:02Z")
}

/* 2 */
{
    "_id" : ObjectId("565b3b7ec8f3530f3a327000"),
    "name" : "javase.jvm.attribute.vendor",
    "value" : "Oracle Corporation Java HotSpot(TM) 64-Bit Server VM 25.5-b02 (1.8)",
    "timestamp" : ISODate("2015-11-29T17:53:02Z")
}

/* 3 */
{
    "_id" : ObjectId("565b3b83c8f3530f3a327002"),
    "name" : "javase.jvm.attribute.name",
    "value" : "69434@server.local",
    "timestamp" : ISODate("2015-11-29T17:53:07Z")
}

/* 4 */
{
    "_id" : ObjectId("565b3b83c8f3530f3a327003"),
    "name" : "javase.jvm.attribute.uptime",
    "value" : 10561,
    "timestamp" : ISODate("2015-11-29T17:53:07Z")
}

/* 5 */
{
    "_id" : ObjectId("565b3b83c8f3530f3a327004"),
    "name" : "javase.jvm.attribute.vendor",
    "value" : "Oracle Corporation Java HotSpot(TM) 64-Bit Server VM 25.5-b02 (1.8)",
    "timestamp" : ISODate("2015-11-29T17:53:07Z")
}
```

## Samples

Sample applications demonstrating the use of Metrics library with MongoDB reporter.

* JavaSE application - https://github.com/aparnachaudhary/mongodb-metrics-demo/tree/master/javase-demo
* CDI application - https://github.com/aparnachaudhary/mongodb-metrics-demo/tree/master/cdi-demo
* JavaEE application - https://github.com/aparnachaudhary/mongodb-metrics-demo/tree/master/javaee-demo
