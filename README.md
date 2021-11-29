[![Actions Status](https://github.com/SolaceProducts/pubsubplus-connector-kafka-sink/workflows/build/badge.svg?branch=master)](https://github.com/SolaceProducts/pubsubplus-connector-kafka-sink/actions?query=workflow%3Abuild+branch%3Amaster)
[![Code Analysis (CodeQL)](https://github.com/SolaceProducts/pubsubplus-connector-kafka-sink/actions/workflows/codeql-analysis.yml/badge.svg?branch=master)](https://github.com/SolaceProducts/pubsubplus-connector-kafka-sink/actions/workflows/codeql-analysis.yml)
[![Code Analysis (PMD)](https://github.com/SolaceProducts/pubsubplus-connector-kafka-sink/actions/workflows/pmd-analysis.yml/badge.svg?branch=master)](https://github.com/SolaceProducts/pubsubplus-connector-kafka-sink/actions/workflows/pmd-analysis.yml)
[![Code Analysis (SpotBugs)](https://github.com/SolaceProducts/pubsubplus-connector-kafka-sink/actions/workflows/spotbugs-analysis.yml/badge.svg?branch=master)](https://github.com/SolaceProducts/pubsubplus-connector-kafka-sink/actions/workflows/spotbugs-analysis.yml)

# Solace PubSub+ Connector for Kafka: Sink

This project provides a Kafka to Solace PubSub+ Event Broker [Sink Connector](//kafka.apache.org/documentation.html#connect_concepts) (adapter) that makes use of the [Kafka Connect API](//kafka.apache.org/documentation/#connect).

**Note**: there is also a PubSub+ Kafka Source Connector available from the [Solace PubSub+ Connector for Kafka: Source](https://github.com/SolaceProducts/pubsubplus-connector-kafka-source) GitHub repository.

Contents:

  * [Overview](#overview)
  * [Use Cases](#use-cases)
  * [Downloads](#downloads)
  * [Quick Start](#quick-start)
  * [Parameters](#parameters)
  * [User Guide](#user-guide)
    + [Deployment](#deployment)
    + [Troubleshooting](#troubleshooting)
    + [Event Processing](#event-processing)
    + [Performance and Reliability Considerations](#performance-and-reliability-considerations)
    + [Security Considerations](#security-considerations)
  * [Developers Guide](#developers-guide)

## Overview

The Solace/Kafka adapter consumes Kafka topic records and streams them to the PubSub+ Event Mesh as topic and/or queue data events. 

The connector was created using PubSub+ high performance Java API to move data to PubSub+.

## Use Cases

#### Protocol and API Messaging Transformations

Unlike many other message brokers, the Solace PubSub+ Event Broker supports transparent protocol and API messaging transformations. 

As the following diagram shows, any Kafka topic (keyed or non-keyed) sink record is instantly available for consumption by a consumer that uses one of the Solace supported open standards languages or transport protocols.

![Messaging Transformations](/doc/images/KSink.png)

#### Tying Kafka into the PubSub+ Event Mesh

The [PubSub+ Event Mesh](//docs.solace.com/Solace-PubSub-Platform.htm#PubSub-mesh) is a clustered group of PubSub+ Event Brokers, which appears to individual services (consumers or producers of data events) to be a single transparent event broker. The Event Mesh routes data events in real-time to any of its client services. The Solace PubSub+ brokers can be any of the three categories: dedicated extreme performance hardware appliances, high performance software brokers that are deployed as software images (deployable under most Hypervisors, Cloud IaaS and PaaS layers and in Docker) or provided as a fully-managed Cloud MaaS (Messaging as a Service). 

When an application registers interest in receiving events, the entire Event Mesh becomes aware of the registration request and knows how to securely route the appropriate events generated by the Solace Sink Connector.

![Messaging Transformations](/doc/images/EventMesh.png)

Because of the Solace architecture, a single PubSub+ Sink Connector can move a new Kafka record to any downstream service.

#### Distributing Messages to IoT Devices

PubSub+ brokers support bi-directional messaging and the unique addressing of millions of devices through fine-grained filtering. Using the Sink Connector, messages created from Kafka records can be efficiently distributed to a controlled set of destinations.

![Messaging Transformations](/doc/images/IoT-Command-Control.png)

## Downloads

The PubSub+ Kafka Sink Connector is available as a ZIP or TAR package from the [downloads](//solaceproducts.github.io/pubsubplus-connector-kafka-sink/downloads/) page.

The package includes the jar libraries, documentation with license information, and sample property files. Download and extract it into a directory that is on the `plugin.path` of your `connect-standalone` or `connect-distributed` properties file.

## Quick Start

This example demonstrates an end-to-end scenario similar to the [Protocol and API messaging transformations](#protocol-and-api-messaging-transformations) use case, using the WebSocket API to receive an exported Kafka record as a message at the PubSub+ event broker.

It builds on the open source [Apache Kafka Quickstart tutorial](//kafka.apache.org/quickstart) and walks through getting started in a standalone environment for development purposes. For setting up a distributed environment for production purposes, refer to the User Guide section.

**Note**: The steps are similar if using [Confluent Kafka](//www.confluent.io/download/); there may be differences in the root directory where the Kafka binaries (`bin`) and properties (`etc/kafka`) are located.

**Steps**

1. Install Kafka. Follow the [Apache tutorial](//kafka.apache.org/quickstart#quickstart_download) to download the Kafka release code, start the Zookeeper and Kafka servers in separate command line sessions, then create a topic named `test` and verify it exists.

2. Install PubSub+ Sink Connector. Designate and create a directory for the PubSub+ Sink Connector (here, we assume it is named `connectors`). Edit the `config/connect-standalone.properties` file, and ensure the `plugin.path` parameter value includes the absolute path of the `connectors` directory.
[Download]( https://solaceproducts.github.io/pubsubplus-connector-kafka-sink/downloads ) and extract the PubSub+ Sink Connector into the `connectors` directory.

3. Acquire access to a PubSub+ message broker. If you don't already have one available, the easiest option is to get a free-tier service in a few minutes in [PubSub+ Cloud](//solace.com/try-it-now/), following the instructions in [Creating Your First Messaging Service](https://docs.solace.com/Solace-Cloud/ggs_signup.htm). 

4. Configure the PubSub+ Sink Connector:

   a) Locate the following connection information of your messaging service for the "Solace Java API" (this is what the connector is using inside):
      * Username
      * Password
      * Message VPN
      * one of the Host URIs

   b) Edit the PubSub+ Sink Connector properties file located at `connectors/pubsubplus-connector-kafka-sink-<version>/etc/solace_sink.properties`  updating following respective parameters so the connector can access the PubSub+ event broker:
      * `sol.username`
      * `sol.password`
      * `sol.vpn_name`
      * `sol.host`

   **Note**: In the configured source and destination information, `topics` is the Kafka source topic (`test`), created in Step 1 and the `sol.topics` parameter specifies the destination topic on PubSub+ (`sinktest`).

5. Start the connector in standalone mode. In a command line session run:
   ```sh
   bin/connect-standalone.sh \
   config/connect-standalone.properties \
   connectors/pubsubplus-connector-kafka-sink-<version>/etc/solace_sink.properties
   ```
   After startup, the logs will eventually contain following line:
   ```
   ================Session is Connected
   ```

6. To watch messages arriving into PubSub+, we use the "Try Me!" test service of the browser-based administration console to subscribe to messages to the `sinktest` topic. Behind the scenes, "Try Me!" uses the JavaScript WebSocket API.

   * If you are using PubSub+ Cloud for your messaging service, follow the instructions in [Trying Out Your Messaging Service](//docs.solace.com/Solace-Cloud/ggs_tryme.htm).
   * If you are using an existing event broker, log into its [PubSub+ Manager admin console](//docs.solace.com/Solace-PubSub-Manager/PubSub-Manager-Overview.htm#mc-main-content) and follow the instructions in [How to Send and Receive Test Messages](//docs.solace.com/Solace-PubSub-Manager/PubSub-Manager-Overview.htm#Test-Messages).

   In both cases ensure to set the topic to `sinktest`, which the connector is publishing to.

7. Demo time! Start to write messages to the Kafka `test` topic. Get back to the Kafka [tutorial](//kafka.apache.org/quickstart#quickstart_send), type and send `Hello world!`.

   The "Try Me!" consumer from Step 6 should now display the new message arriving to PubSub+ through the PubSub+ Kafka Sink Connector:
   ```
   Hello world!
   ```

## Parameters

The Connector parameters consist of [Kafka-defined parameters](https://kafka.apache.org/documentation/#connect_configuring) and PubSub+ connector-specific parameters.

Refer to the in-line documentation of the [sample PubSub+ Kafka Sink Connector properties file](/etc/solace_sink.properties) and additional information in the [Configuration](#Configuration) section.

## User Guide

### Deployment

The PubSub+ Sink Connector deployment has been tested on Apache Kafka 2.4 and Confluent Kafka 5.4 platforms. The Kafka software is typically placed under the root directory: `/opt/<provider>/<kafka or confluent-version>`.

Kafka distributions may be available as install bundles, Docker images, Kubernetes deployments, etc. They all support Kafka Connect which includes the scripts, tools and sample properties for Kafka connectors.

Kafka provides two options for connector deployment: [standalone mode and distributed mode](//kafka.apache.org/documentation/#connect_running).

* In standalone mode, recommended for development or testing only, configuration is provided together in the Kafka `connect-standalone.properties` and in the PubSub+ Sink Connector `solace_sink.properties` files and passed to the `connect-standalone` Kafka shell script running on a single worker node (machine), as seen in the [Quick Start](#quick-start).

* In distributed mode, the Kafka configuration is provided in `connect-distributed.properties` and passed to the `connect-distributed` Kafka shell script, which is started on each worker node. The `group.id` parameter identifies worker nodes belonging the same group. The script starts a REST server on each worker node and PubSub+ Sink Connector configuration is passed to any one of the worker nodes in the group through REST requests in JSON format.

To deploy the Connector, for each target machine, [download]( https://solaceproducts.github.io/pubsubplus-connector-kafka-sink/downloads) and extract the PubSub+ Sink Connector into a directory and ensure the `plugin.path` parameter value in the `connect-*.properties` includes the absolute path to that directory. Note that Kafka Connect, i.e., the `connect-standalone` or `connect-distributed` Kafka shell scripts, must be restarted (or an equivalent action from a Kafka console is required) if the PubSub+ Sink Connector deployment is updated.

Some PubSub+ Sink Connector configurations may require the deployment of additional specific files, like keystores, truststores, Kerberos config files, etc. It does not matter where these additional files are located, but they must be available on all Kafka Connect Cluster nodes and placed in the same location on all the nodes because they are referenced by absolute location and configured only once through one REST request for all.

#### REST JSON Configuration

First test to confirm the PubSub+ Sink Connector is available for use in distributed mode with the command:
```ini
curl http://18.218.82.209:8083/connector-plugins | jq
```

In this case the IP address is one of the nodes running the distributed mode worker process, and the port defaults to 8083 or as specified in the `rest.port` property in `connect-distributed.properties`. If the connector is loaded correctly, you should see a response similar to:

```
  {
    "class": "com.solace.connector.kafka.connect.sink.SolaceSinkConnector",
    "type": "sink",
    "version": "2.1.0"
  },
```

At this point, it is now possible to start the connector in distributed mode with a command similar to:

```ini
curl -X POST -H "Content-Type: application/json" \
             -d @solace_sink_properties.json \
             http://18.218.82.209:8083/connectors
``` 

The connector's JSON configuration file, in this case, is called `solace_sink_properties.json`. A sample is available [here](/etc/solace_sink_properties.json), which can be extended with the same properties as described in the [Parameters section](#parameters).

Determine whether the Sink Connector is running with the following command:
```ini
curl 18.218.82.209:8083/connectors/solaceSourceConnector/status | jq
```
If there was an error in starting, the details are returned with this command. 

### Troubleshooting

In standalone mode, the connect logs are written to the console. If you do not want to send the output to the console, simply add the `-daemon` option to have all output directed to the logs directory.

In distributed mode, the logs location is determined by the `connect-log4j.properties` located at the `config` directory in the Apache Kafka distribution or under `etc/kafka/` in the Confluent distribution.

If logs are redirected to the standard output, here is a sample log4j.properties snippet to direct them to a file:
```
log4j.rootLogger=INFO, file
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=/var/log/kafka/connect.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%d] %p %m (%c:%L)%n
log4j.appender.file.MaxFileSize=10MB
log4j.appender.file.MaxBackupIndex=5
log4j.appender.file.append=true
```

To troubleshoot PubSub+ connection issues, increase logging level to DEBUG by adding following line:
```
log4j.logger.com.solacesystems.jcsmp=DEBUG
```
Ensure that you set it back to INFO or WARN for production.

### Event Processing

#### Record Processors

There are many ways to map topics, partitions, keys, and values of Kafka records to PubSub+ messages, depending on the application.

The PubSub+ Sink Connector comes with three sample record processors that can be used as-is, or as a starting point to develop a customized record processor.

* **SolSimpleRecordProcessor**: Takes the Kafka sink record as a binary payload with a binary schema for the value, which becomes the PubSub+ message payload. The key and value schema can be changed via the configuration file.

* **SolSimpleKeyedRecordProcessor**: A more complex sample that allows the flexibility of mapping the sink record key to PubSub+ message contents. In this sample, the kafka key is set as a Correlation ID in the Solace messages. The option of no key in the record is also possible.

* **SolDynamicDestinationRecordProcessor**: By default, the Sink Connector sends messages to destinations (Topics or Queues) defined in the configuration file. This example shows how to route each Kafka record to a potentially different PubSub+ topic based on the record binary payload. In this imaginary transportation example, the records are distributed to buses listening to topics like `ctrl/bus/<busId>/<command>`, where the `busId` is encoded in the first 4 bytes in the record value and `command` in the rest. Note that `sol.dynamic_destination=true` must be specified in the configuration file to enable this mode (otherwise destinations are taken from sol.topics or sol.queue).

In all processors the original Kafka topic, partition and offset are included for reference in the PubSub+ Message as UserData in the Solace message header, sent as a "User Property Map". The message dump is similar to:
```
Destination:                            Topic 'sinktest'
AppMessageType:                         ResendOfKafkaTopic: test
Priority:                               4
Class Of Service:                       USER_COS_1
DeliveryMode:                           DIRECT
Message Id:                             4
User Property Map:                      3 entries
  Key 'k_offset' (Long): 0
  Key 'k_topic' (String): test
  Key 'k_partition' (Integer): 0

Binary Attachment:                      len=11
  48 65 6c 6c 6f 20 57 6f    72 6c 64                   Hello.World
```

The desired record processor is loaded at runtime based on the configuration of the JSON or properties configuration file, for example:
```
sol.record_processor_class=com.solace.connector.kafka.connect.sink.recordprocessor.SolSimpleRecordProcessor
```

It is possible to create more custom record processors based on you Kafka record requirements for keying and/or value serialization and the desired format of the PubSub+ event message. Simply add the new record processor classes to the project. The desired record processor is installed at run time based on the configuration file. 

Refer to the [Developers Guide](#developers-guide) for more information about building the Sink Connector and extending record processors.

#### Message Replay

By default, the Sink Connector starts to send events based on the last Kafka topic offset that was flushed before the connector was stopped. It is possible to use the Sink Connector to replay messages from the Kafka topic.

To start processing from an offset position that is different from the one stored before the connector was stopped, add the following configuration entry:
```
sol.kafka_replay_offset=<offset>
```

A value of 0 results in the replay of the entire Kafka Topic. A positive value results in the replay from that offset value for the Kafka Topic. The same offset value is used against all active partitions for that Kafka Topic.

### Performance and Reliability Considerations

#### Sending to PubSub+ Topics

We recommend using PubSub+ Topics if high throughput is required and the Kafka Topic is configured for high performance. Message duplication and loss mimics the underlying reliability and QoS configured for the Kafka topic.

By default, messages are published to topics using direct messaging. To publish persistent messages to topics using a local transaction, set `sol.use_transactions_for_topics` to `true`. See [Sending with Local Transactions](#sending-with-local-transactions) for more info.

#### Sending to PubSub+ Queue

When Kafka records reliability is critical, we recommend configuring the Sink Connector to send records to the Event Mesh using PubSub+ queues at the cost of reduced throughput.

A PubSub+ queue guarantees order of delivery, provides High Availability and Disaster Recovery (depending on the setup of the PubSub+ brokers) and provides an acknowledgment to the connector when the event is stored in all HA and DR members and flushed to disk. This is a higher guarantee than is provided by Kafka, even for Kafka idempotent delivery.

The connector uses local transactions to deliver to the queue by default. See [Sending with Local Transactions](#sending-with-local-transactions) for more info.

Note that generally one connector can send to only one queue.

#### Sending with Local Transactions

By default, only sending to a queue uses local transactions. To use the transacted session to send persistent messages to topics, set `sol.use_transactions_for_topics` to `true`.

The transaction is committed if messages are flushed by Kafka Connect (see [below how to tune flush interval](#recovery-from-kafka-connect-api-or-kafka-broker-failure)) or the outstanding messages size reaches the `sol.autoflush.size` (default 200) configuration.

#### Recovery from Kafka Connect API or Kafka Broker Failure

Operators are expected to monitor their connector for failures since errors will cause it to stop. If any are found and the connector was stopped, the operator must explicitly restart it again once the error condition has been resolved.

The Kafka Connect API automatically keeps track of the offset that the Sink Connector has read and processed. If the connector stops or is restarted, the Connect API starts passing records to the connector based on the last saved offset.

The time interval to save the last offset can be tuned via the `offset.flush.interval.ms` parameter (default 60,000 ms) in the worker's `connect-distributed.properties` configuration file.

Multiple retries can also be configured using the `errors.retry.timeout` parameter (default 0 ms) in the PubSub+ Sink Connector `solace_sink.properties` configuration file. Please refer to the [Kafka documentation](https://kafka.apache.org/documentation/#connect_errorreporting) for more info on retry configuration options.

Recovery may result in duplicate PubSub+ events published to the Event Mesh. As described [above](#record-processors), the Solace message header "User Property Map" contains all the Kafka unique record information which enables identifying and filtering duplicates.

#### Multiple Workers

The Sink Connector can scale when more performance is required. Throughput is limited with a single instance of the Connect API; the Kafka Broker can produce records and the Solace PubSub+ broker can consume messages at a far greater rate than the connector can handle them.

You can deploy and automatically and spread multiple connector tasks across all available Connect API workers simply by indicating the number of desired tasks in the connector configuration file. There are no special configuration requirements for PubSub+ queue or topics to support scaling.

On the Kafka side, the Connect API automatically uses a Kafka consumer group to allow moving of records from multiple topic partitions in parallel.

### Security Considerations

The security setup and operation between the PubSub+ broker and the Sink Connector and Kafka broker and the Sink Connector operate completely independently.
 
The Sink Connector supports both PKI and Kerberos for more secure authentication beyond the simple user name/password, when connecting to the PubSub+ event broker.

The security setup between the Sink Connector and the Kafka brokers is controlled by the Kafka Connect libraries. These are exposed in the configuration file as parameters based on the Kafka-documented parameters and configuration. Please refer to the [Kafka documentation](//docs.confluent.io/current/connect/security.html) for details on securing the Sink Connector to the Kafka brokers for both PKI/TLS and Kerberos. 

#### PKI/TLS

The PKI/TLS support is well documented in the [Solace Documentation](//docs.solace.com/Configuring-and-Managing/TLS-SSL-Service-Connections.htm), and will not be repeated here. All the PKI required configuration parameters are part of the configuration variable for the Solace session and transport as referenced above in the [Parameters section](#parameters). Sample parameters are found in the included [properties file](/etc/solace_sink.properties). 

#### Kerberos Authentication

Kerberos authentication support requires a bit more configuration than PKI since it is not defined as part of the Solace session or transport. 

Typical Kerberos client applications require details about the Kerberos configuration and details for the authentication. Since the Sink Connector is a server application (i.e. no direct user interaction) a Kerberos _keytab_ file is required as part of the authentication, on each Kafka Connect Cluster worker node where the connector is deployed.

The included [krb5.conf](/etc/krb5.conf) and [login.conf](/etc/login.conf) configuration files are samples that allow automatic Kerberos authentication for the Sink Connector when it is deployed to the Connect Cluster. Together with the _keytab_ file, they must be also available on all Kafka Connect cluster nodes and placed in the same (any) location on all the nodes. The files are then referenced in the Sink Connector properties, for example:
```ini
sol.kerberos.login.conf=/opt/kerberos/login.conf
sol.kerberos.krb5.conf=/opt/kerberos/krb5.conf
```

The following property entry is also required to specify Kerberos Authentication:
```ini
sol.authentication_scheme=AUTHENTICATION_SCHEME_GSS_KRB
```

Kerberos has some very specific requirements to operate correctly. Some additional tips are as follows:
* DNS must be operating correctly both in the Kafka brokers and on the Solace PS+ broker.
* Time services are recommended for use with the Kafka Cluster nodes and the Solace PS+ broker. If there is too much drift in the time between the nodes, Kerberos will fail.
* You must use the DNS name and not the IP address in the Solace PS+ host URI in the Connector configuration file 
* You must use the full Kerberos user name (including the Realm) in the configuration property; obviously, no password is required. 

## Developers Guide

### Build the Project

JDK 8 or higher is required for this project.

First, clone this GitHub repo:
```shell
git clone https://github.com/SolaceProducts/pubsubplus-connector-kafka-sink.git
cd pubsubplus-connector-kafka-sink
```

Then run the build script:
```shell
./gradlew clean build
```

This script creates artifacts in the `build` directory, including the deployable packaged PubSub+ Sink Connector archives under `build\distributions`.

### Test the Project

An integration test suite is also included, which spins up a Docker-based deployment environment that includes a PubSub+ event broker, Zookeeper, Kafka broker, Kafka Connect. It deploys the connector to Kafka Connect and runs end-to-end tests.

1. Install the test support module:
    ```shell
    git submodule update --init --recursive
    cd solace-integration-test-support
    ./mvnw clean install -DskipTests
    cd ..
    ```
2. Run the tests:
    ```shell
    ./gradlew clean test integrationTest
    ```

### Build a New Record Processor

The processing of a Kafka record to create a PubSub+ message is handled by an interface defined in [`SolRecordProcessorIF.java`](/src/main/java/com/solace/connector/kafka/connect/sink/SolRecordProcessorIF.java). This is a simple interface that creates the Kafka source records from the PubSub+ messages. This project includes three examples of classes that implement this interface:

* [SolSimpleRecordProcessor](/src/main/java/com/solace/connector/kafka/connect/sink/recordprocessor/SolSimpleRecordProcessor.java)
* [SolSimpleKeyedRecordProcessor](/src/main/java/com/solace/connector/kafka/connect/sink/recordprocessor/SolSimpleKeyedRecordProcessor.java)
* [SolDynamicDestinationRecordProcessor](/src/main/java/com/solace/connector/kafka/connect/sink/recordprocessor/SolDynamicDestinationRecordProcessor.java)

You can use these examples as starting points for implementing your own custom record processors.

More information on Kafka sink connector development can be found here:
- [Apache Kafka Connect](https://kafka.apache.org/documentation/)
- [Confluent Kafka Connect](https://docs.confluent.io/current/connect/index.html)

## Additional Information

For additional information, use cases and explanatory videos, please visit the [PubSub+/Kafka Integration Guide](https://docs.solace.com/Developer-Tools/Integration-Guides/Kafka-Connect.htm).

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Authors

See the list of [contributors](../../graphs/contributors) who participated in this project.

## License

This project is licensed under the Apache License, Version 2.0. See the [LICENSE](LICENSE) file for details.

## Resources

For more information about Solace technology in general, please visit these resources:

- How to [Setup and Use](https://solace.com/resources/solace-with-kafka/kafka-source-and-sink-how-to-video-final)
- The [Solace Developers website](https://www.solace.dev/)
- Understanding [Solace technology]( https://solace.com/products/tech/)
- Ask the [Solace Community]( https://solace.community/)
