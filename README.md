Run Kafka Locally: A Step-by-Step Guide
This repository provides a comprehensive, step-by-step guide to downloading, installing, and running a single-node Apache Kafka cluster on your local machine. This setup is ideal for development, learning, and testing Kafka-based applications without the complexity of a multi-broker cluster.
Table of Contents
Prerequisites
Step 1: Download Apache Kafka
Step 2: A Quick Look at Configuration
Step 3: Start the Servers
Start ZooKeeper
Start the Kafka Broker
Step 4: Interact with Your Kafka Cluster
Create a Topic
Describe a Topic
Produce Events
Consume Events
Step 5: Shutdown Procedure
Beyond the Basics: Next Steps
Running Kafka with Docker
Management UIs
Common Issues
Prerequisites
Before you begin, ensure you have a Java Development Kit (JDK) installed. Apache Kafka is built on the JVM and requires a compatible JDK to run. Version 11 or 17 is recommended.
Verify your installation by running:
Generated bash
java --version
Use code with caution.
Bash
You should see output specifying your Java version.
Step 1: Download Apache Kafka
Navigate to the official Apache Kafka Downloads page.
Download the latest binary download (not the source). Choose a release with a recent Scala version (e.g., 2.13). The file will be a .tgz archive.
Extract the archive to a location of your choice.
Generated bash
# Example for version 3.6.1. Replace with your downloaded version.
tar -xzf kafka_2.13-3.6.1.tgz
cd kafka_2.13-3.6.1
Use code with caution.
Bash
All subsequent commands in this guide assume you are running them from within this kafka_2.13-3.6.1/ directory.
Step 2: A Quick Look at Configuration
The default configuration files are located in the config/ directory and are sufficient for a local setup. The two most important files are:
config/zookeeper.properties: Configures the ZooKeeper instance that Kafka uses for metadata management (e.g., tracking broker status, topic configurations). The key settings are:
dataDir=/tmp/zookeeper: Where ZooKeeper stores its data.
clientPort=2181: The port ZooKeeper listens on.
config/server.properties: Configures the Kafka broker itself. Key settings include:
broker.id=0: A unique ID for this broker in the cluster.
listeners=PLAINTEXT://localhost:9092: The network address the broker will listen on for client connections.
log.dirs=/tmp/kafka-logs: The directory where Kafka will store topic data (the "logs").
Note: Modern Kafka versions are moving away from ZooKeeper in favor of a self-managed metadata quorum called KRaft. This guide uses the traditional, widely-supported ZooKeeper setup.
Step 3: Start the Servers
You will need two separate terminal windows to run the Kafka cluster.
Start ZooKeeper
Open your first terminal window and run the following command to start the ZooKeeper service.
Generated bash
bin/zookeeper-server-start.sh config/zookeeper.properties
Use code with caution.
Bash
You will see a series of startup logs. Leave this terminal running.
Start the Kafka Broker
Open a new, second terminal window and run the following command to start the Kafka broker. The broker will connect to the ZooKeeper instance you just started.
Generated bash
bin/kafka-server-start.sh config/server.properties
Use code with caution.
Bash
You will see another series of startup logs. Leave this terminal running as well.
Your single-node Kafka cluster is now running!
Step 4: Interact with Your Kafka Cluster
Open a third terminal window to execute the following command-line tools.
Create a Topic
A topic is a named stream of events. Let's create a topic named order-events with a single partition and a replication factor of 1 (since we only have one broker).
Generated bash
bin/kafka-topics.sh --create \
--topic order-events \
--bootstrap-server localhost:9092 \
--partitions 1 \
--replication-factor 1
Use code with caution.
Bash
--bootstrap-server localhost:9092: Specifies the address of a broker to connect to for cluster metadata.
--partitions 1: Splits the topic into one log.
--replication-factor 1: How many copies of the data to keep. In a single-broker cluster, this must be 1.
You can list all topics in the cluster to verify its creation:
Generated bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
Use code with caution.
Bash
Describe a Topic
To get more detailed information about a topic, use the --describe flag. This shows the partition layout, the leader broker for each partition, and the set of in-sync replicas.
Generated bash
bin/kafka-topics.sh --describe --topic order-events --bootstrap-server localhost:9092
Use code with caution.
Bash
Expected Output:
Generated code
Topic: order-events   TopicId: <some_id>   PartitionCount: 1   ReplicationFactor: 1    Configs:
  Topic: order-events   Partition: 0    Leader: 0   Replicas: 0 InSyncReplicas: 0
Use code with caution.
This confirms that partition 0 is led by broker 0 and its data resides on that same broker.
Produce Events
Kafka provides a simple console producer script to send messages to a topic from the command line.
Generated bash
bin/kafka-console-producer.sh --topic order-events --bootstrap-server localhost:9092
Use code with caution.
Bash
The command will give you a > prompt. Type your messages (events) here, pressing Enter after each one. The messages can be simple strings or structured data like JSON.
Generated code
>{"orderId":"101", "product":"Laptop", "quantity":1}
>{"orderId":"102", "product":"Mouse", "quantity":2}
>
Use code with caution.
Keep this producer terminal running to send more events later.
Consume Events
Finally, let's read the events from the topic using the console consumer. Open a fourth terminal window for this.
Generated bash
bin/kafka-console-consumer.sh --topic order-events --bootstrap-server localhost:9092 --from-beginning
Use code with caution.
Bash
--from-beginning: This flag tells the consumer to read all events in the topic from the start. If you omit this, the consumer will only show new events that are produced after it starts.
You will immediately see the JSON events you produced. Now, go back to your producer terminal and send a new message. Watch it appear instantly in the consumer terminal, demonstrating Kafka's real-time capabilities.
Step 5: Shutdown Procedure
To stop the local cluster, press Ctrl+C in each of your open terminals. Shut them down in the reverse order you started them to ensure a clean exit:
Stop the Console Consumer (Terminal 4)
Stop the Console Producer (Terminal 3)
Stop the Kafka Broker (Terminal 2)
Stop the ZooKeeper Server (Terminal 1)
It is important to stop the broker before ZooKeeper, as the broker needs to de-register itself from ZooKeeper cleanly.
Beyond the Basics: Next Steps
Running Kafka with Docker
For a more isolated and reproducible environment, consider running Kafka with Docker. The Confluent Platform Docker Compose setup is a popular choice that includes Kafka, ZooKeeper, and other useful components like Control Center and Schema Registry.
Management UIs
While the command line is powerful, UIs can make management and monitoring much easier. Popular options include:
Confluent Control Center: A comprehensive, web-based UI for managing and monitoring Confluent Platform and Confluent Cloud.
AKHQ (formerly KafkaHQ): An open-source Kafka GUI for clusters, topics, consumer groups, and more.
UI for Kafka: Another popular open-source web UI.
Common Issues
Port Conflict: If you see an error like Address already in use for ports 9092 (Kafka) or 2181 (ZooKeeper), it means another process is using that port. You can either stop the other process or change the port in the respective .properties file (listeners in server.properties and clientPort in zookeeper.properties).
java.lang.OutOfMemoryError: The default heap size might be too small for your machine. You can increase it by setting the KAFKA_HEAP_OPTS environment variable before starting the server (e.g., export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G").
