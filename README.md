### Overview
camel-plantuml is a tool which helps to genereate [PlantUML](https://plantuml.com/) diagrams describing Apache [Camel](https://camel.apache.org/) routes. 

It allows to have diagrams where we can see interactions between endpoints and routes.

If you consider following routes:
```
from(timer("foo").period(5000)).id("slowTimerRoute")
        .description("Route which generates slow periodic events")
        .setBody(constant("slow"))
        .to(seda("endpoint1"));

from(timer("bar").period(1000)).id("fastTimerRoute")
        .description("Route which generates fast periodic events")
        .setBody(constant("fast"))
        .to(seda("endpoint1"));

from(seda("endpoint1")).id("mainRoute")
        .description("Route which handles processing of the message")
        .log(LoggingLevel.INFO, "${body}")
        .enrich().constant("direct://endpoint2")
        .toD(mock("mock-${body}"));

from(direct("endpoint2")).id("transformRoute")
        .description("Route which transforms the message")
        .transform(simple("${body}${body}"));
```
It will allow you generate this:

- with all endpoints:

![](images/example1.full.svg)

- with only "internal" endpoints:

![](images/example1.light.svg)

### How it works
It uses the Camel JMX MBeans (which are enabled by default in Camel), and particularly the ones related to routes and processors.

Following processors are handled:
- SendProcessor (`to`)
- SendDynamicProcessor (`toD`)
- Enricher (`enrich`)
- PollEnricher (`pollEnrich`)
- WireTapProcessor (`wireTap`)
- RecipientList (`recipientList`)

The PlantUML code is exposed through a configurable HTTP endpoint, and can be rendered afterwards as an image using PlantUML [webserver](http://www.plantuml.com/plantuml/uml "PlantUML webserver") or any other tool where PlantUML is available (VSCode, IntelliJ, your own PlantUML server...)

### Features
This tool generates PlantUML diagrams with following features:
- each route is rendered as a rectangle.
- each static endpoint base URI is rendered as a queue with a "static" layout.
- each dynamic endpoint URI is rendered as a queue with a "dynamic" layout.
- each consumer is rendered as a labelled arrow (`from` or `pollEnrich`) which connects an endpoint to a route.
- each producer is rendered as a labelled arrow (`to`,`toD`,`enrich`,`wireTap` or `recipientList`) which connects a route to an endpoint.

### Versions
There is a version for the two Camel major versions. Both versions uses Java `1.8`.

##### Camel 2.x
The jar to use is `camel2-plantuml`. It has been built with Camel version `2.20.4`.
The jar is a OSGi bundle, and can be used with Apache ServiceMix/Apache Karaf.

##### Camel 3.x
The jar to use is `camel3-plantuml`. It has been built with Camel version `3.4.4`.
The jar is a OSGi bundle, and can be used with Apache ServiceMix/Apache Karaf.

### How to use
#### 1. Add the dependency to your project
If you use Camel **2.x**:
```
<dependency>
    <groupId>fr.ncasaux</groupId>
    <artifactId>camel2-plantuml</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```
If you use Camel **3.x**:
```
<dependency>
    <groupId>fr.ncasaux</groupId>
    <artifactId>camel3-plantuml</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

#### 2. Add the route builder to your Camel context
`getContext().addRoutes(new CamelPlantUmlRouteBuilder());`

Default host is `localhost`, default port is `8090`, but you can change them:

`getContext().addRoutes(new CamelPlantUmlRouteBuilder("localhost", 8090));`


#### 3. Start your Camel context, and open a browser:
To have all the endpoints, go to:

`http://localhost:8090/camel-plantuml/diagram.puml`

To connect routes directly (and hide "internal" endpoints), go to:

`http://localhost:9090/camel-plantuml/diagram.puml?connectRoutes=true`

