# Apache Edgent Meetup: a Recap
This past week, I was happy to present at the first ever meetup of the Apache Edgent community. Hosted by Galvanize in San Francisco's SOMA district, it was attended by a diverse group of 40 developers -- some entirely uninitiated to datastreaming, and others with a long history of developing streaming solutions. 

Edgent has been incubating in the Apache community for more than a year, and it is comparatively young next to its streaming counterparts such as Apex, Kafka, or Flink. Most of the attendees I spoke with had not heard of Edgent before the meetup was scheduled, so I believe the strength of its attendance highlights a growing level of interest and importance of streaming in the industry. 

# Edgent's Positioning
Since many were unfamiliar with Edgent, I began with a quick summary of the problem it's trying to solve: Edgent is a library which performs streaming analytics on "edge" devices. An edge device can be an Android phone, a RaspberryPI, or any small constrained computing device. Most real-time streaming frameworks, for example IBM Streams, are tailored to run and scale in a datacenter or cluster. Unfortunately, this means that any produced data must be sent to that backend. This has two major drawbacks:
* Network costs. Many edge devices communicate using 3G or 4G. This can be prohibitively expensive, or simply impossible, for large amounts of data.
* Increased latency. If a decision needs to be made immediately on an edge device, it isn't feasible to incur the round trip cost of sending data to and from a backend.

Since Edgent runs on the same device that produces the data, it isn't subject to these limitations.

# Use Cases & Code Samples
Talking at a high level about new technology only goes so far, so I gave examples of use cases for which Edgent was designed. Consider an application which seeks to monitor the temperature of a car's engine and act accordingly if it begins to overheat. Most modern cars come equipped with sensors which can be accessed through a CAN bus interface, so a possible solution would be to have Edgent run on a RaspberryPI, interface with the bus, and listen for any abnormal temperature readings. Upon finding an abnormal reading, it could signal this event to a backend which could take further action (such as telling the engine to decrease its maximum RPM).

By doing this simple analytic at the edge, and only sending back the "interesting" data, Edgent can greatly reduce the overall amount data sent over the network. Additionally, when communicating with a backend becomes necessary, Edgent reduces the boilerplate code of using message brokers such as MQTT and Kafka by providing a suite of connectors. 

To illustrate this, I provided a code example of Edgent initializing a connection with the Watson Internet of Thing Foundation (a MQTT connection broker and device registry), and sending it the words "Hello Edgent!". For an application which defines a MQTT connection, creates its input data, converts it to JSON, and sends the data to a backend, its surprisingly concise:

``` Java
public static void main(String[] args) throws Exception {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        IotDevice device = new IotpDevice(top, new File("device.cfg"));
        TStream<String> helloStream = top.strings("Hello", "Edgent!");
        TStream<JsonObject> output = helloStream.map(tuple -> {
        	return new JsonObject().addProperty("value", tuple);
        });
        device.events(output, "hello", QoS.FIRE_AND_FORGET);
        dp.submit(top);
    }
```

Going line by line, the first step is to create a `Provider`. A `Provider` allows the user to create and submit an Edgent application:
```Java
DirectProvider dp = new DirectProvider();
```
With the `Provider`, a `Topology` is created. A `Topology` represents the user's application, including data sources, processing, and communication with a backend:
```Java
Topology top = dp.newTopology();
```
Using the topology, we can create an `IotDevice` to send messages to a backend. Specifically, we use an `IotpDevice` which uses MQTT under the covers:
```Java
IotDevice device = new IotpDevice(top, new File("device.cfg"));
```
We can use the `Topology#strings` method to create a stream of two Java Strings, "Hello" and "Edgent!":
```Java
TStream<String> helloStream = top.strings("Hello", "Edgent!");
```
To send the Java Strings to the backend using the `IotDevice`, we can wrap them in JSON using the `TStream#map` method and Java8 lambdas:
```Java
TStream<JsonObject> output = helloStream.map(tuple -> {
        	return new JsonObject().addProperty("value", tuple);
        });
```
We then call the `IotDevice#events` method to indicate that each JsonObject should be sent to the backend on the `hello` topic:
```java
device.events(output, "hello", QoS.FIRE_AND_FORGET);
```
Finally, we can submit our application to be run by calling the `Provider#submit` method:
```Java
dp.submit(top);
```
If the backend is configured to write the messages to standard output, it should display:
```
Hello
Edgent!
```
# Q & A
I received several questions which indicated to me that the audience had grasped Edgent's positioning, one of which related to how feasible it would be to create a c++ implementation of Edgent. Many edge devices don't come with a JVM to run Edgent's current Java implementation. A c++ version of Edgent, which could be compiled down to x86 code, would not be restricted to platforms that support a JVM.

This is a fair point. Java was chosen as the initial language because it provides a good compromise between development speed, runtime performance, and cross-platform support. Ultimately though, the language to prioritize is a decision which should stem from the community. A c++ implementation of Edgent would be welcome, and feasible, but it falls to the community to initiate its support.

# Reception
Based on the questions I received, and my interactions with the audience, the meetup seemed to spark curiosity. I'm hopeful that if we continue to host such events, it will translate into more subscribers on the mailing list and increased interaction on the github site.
