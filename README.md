Dropwizard Websocket Support
==========
This project is forked from [LivePersonInc/dropwizard-websockets](https://github.com/LivePersonInc/dropwizard-websockets) with the sole gole fo keeping up with upstream Dropwizard releases, namely 1.3.x and 2.0.x.

[![Build Status](https://travis-ci.org/defer/dropwizard-websockets.svg?branch=release%2F1.3.x)](https://travis-ci.org/defer/dropwizard-websockets)
[![Jitpack](https://jitpack.io/v/defer/dropwizard-websockets.svg)](https://jitpack.io/#defer/dropwizard-websockets/)
[![Coverage Status](https://coveralls.io/repos/defer/dropwizard-websockets/badge.svg?branch=release%2F1.3.x&service=github)](https://coveralls.io/github/defer/dropwizard-websockets?branch=release%2F1.3.x)

A [3rd party Dropwizard bundle](http://modules.dropwizard.io/thirdparty/), that enhances [Dropwizard](http://www.dropwizard.io) capabilities to support not only JAX-RS resources but also websockets endpoints using the JSR-356 API.

The websockets endpoints will be instrumented the same way Dropwizards does with JAX-RS resources, and their metrics will be exposed in the same way. This includes:

* Counters of current open sessions.
* Counters and rate meters for new connections.
* Counters and rate meters for messages reviewed by the endpoint.
* Timers and statistics for session duration.

Maven
---
Add the Maven dependency:

```xml
<repositories>
  <repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
  </repository>
</repositories>
<dependency>
  <groupId>com.github.defer</groupId>
  <artifactId>dropwizard-websockets</artifactId>
  <version></version>
</dependency>
```

Gradle
---
Add the Gradle dependency: 

```groovy
allprojects {
  repositories {
    ...
    maven { url 'https://jitpack.io' }
  }
}

implementation 'com.github.defer:dropwizard-websockets:<version>'
```


Usage
---
In your code you should add the ``WebsocketBundle`` in the initialization stage of the Application. Give the bundle your endpoints classes (or ``ServerEndpoindConfig`` in case of programmatic endpoints) as parameters:

```java
public void initialize(Bootstrap<Configuration> bootstrap) {
    bootstrap.addBundle(new WebsocketBundle(MyWebSocket1.class, MyWebSocket2.class));
}
```

Or, if you prefer, you can register the endpoint before the running stage:

```java
public void initialize(Bootstrap<Configuration> bootstrap) {
    websocketBundle = new WebsocketBundle();        
    bootstrap.addBundle(websocketBundle);
}

@Override
public void run(Configuration configuration, Environment environment) throws Exception {
    // Using BasicServerEndpointConfig lets you inject objects to the websocket endpoint:
    final BasicServerEndpointConfig bsec = new BasicServerEndpointConfig(EchoServer.class, "/extends-ws");
    // bsec.getUserProperties().put(Environment.class.getName(), environment);
    // Then you can get it from the Session object
    // - obj = session.getUserProperties().get("objectName");            
    websocketBundle.addEndpoint(bsec);
}
```

That's all.
A full example can be found in the [tests classes](https://github.com/defer/dropwizard-websockets/blob/master/src/test/java/io/dropwizard/websockets/MyApp.java).

Metrics
---
In order to collect metrics on your endpoints, you should annotate them with metrics annotations:

```java
@Metered
@Timed
@ExceptionMetered
@ServerEndpoint("/annotated-ws")
public static class AnnotatedEchoServer {
    @OnOpen
    public void myOnOpen(final Session session) throws IOException {
        session.getAsyncRemote().sendText("welcome");
    }

    @OnMessage
    public void myOnMsg(final Session session, String message) {
        session.getAsyncRemote().sendText(message.toUpperCase());
    }

    @OnClose
    public void myOnClose(final Session session, CloseReason cr) {
    }
}
```

Then you'll be able to see your metrics as follows:

```json
{
  "counters" : {
    "io.dropwizard.websockets.MyApp$AnnotatedEchoServer.openConnections" : {
      "count" : 2
    }
  },
  "meters" : {
    "io.dropwizard.websockets.MyApp$AnnotatedEchoServer.OnError" : {
      "count" : 0,
      "m15_rate" : 0.0,
      "m1_rate" : 0.0,
      "m5_rate" : 0.0,
      "mean_rate" : 0.0,
      "units" : "events/second"
    },
    "io.dropwizard.websockets.MyApp$AnnotatedEchoServer.OnMessage" : {
      "count" : 3,
      "m15_rate" : 0.6,
      "m1_rate" : 0.6,
      "m5_rate" : 0.6,
      "mean_rate" : 0.3194501069682357,
      "units" : "events/second"
    }
  },
  "timers" : {
    "io.dropwizard.websockets.MyApp$AnnotatedEchoServer" : {
      "count" : 1,
      "max" : 0.101819137,
      "mean" : 0.101819137,
      "min" : 0.101819137,
      "p50" : 0.101819137,
      "p75" : 0.101819137,
      "p95" : 0.101819137,
      "p98" : 0.101819137,
      "p99" : 0.101819137,
      "p999" : 0.101819137,
      "stddev" : 0.0,
      "m15_rate" : 0.2,
      "m1_rate" : 0.2,
      "m5_rate" : 0.2,
      "mean_rate" : 0.10647618704871187,
      "duration_units" : "seconds",
      "rate_units" : "calls/second"
    }
  }
}
```
Alternatives
---
See also [dropwizard-websocket-jee7-bundle](https://github.com/TomCools/dropwizard-websocket-jee7-bundle).
