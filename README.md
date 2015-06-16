# Kafka Development Environment
This repository gives you the ability to create a local kafka cluster for development and experimentation

# Setup
1. Install [Docker](https://docs.docker.com/installation/#installation) onto your system 
2. Install [Docker-compose](https://docs.docker.com/compose/install/)
3. Download [0.8.2.1](http://kafka.apache.org/downloads.html) version of Kafka and unzip in a location of your choice, you'll need this mainly for the kafka command/console tools
4. Erm, just in case you forgot, you'll need to clone this repo :)

# Running 
1. You'll need to create local directories (under ~/tmp/docker) that are linked to directories internally used by the containers. Set up all associated local volumes/directories by running this command
``
mkdir -p `grep /tmp/docker docker-compose.yml | cut -d' ' -f6 | cut -d':' -f1 | cut -c 2- | sort | xargs -i echo $HOME{}`
  ``
  
2. Ensure Docker is running (Mac users follow these [steps](https://docs.docker.com/installation/mac/#from-your-command-line))
3. Start all the containers, for Linux users run ``docker-compose up -d`` and Mac users run ``docker-compose -f docker-compose-osx.yml up -d``
4. Running ``docker-compose ps`` or ``docker ps`` should show at least a Zookeeper Container and a Kafka Container

# Stopping
To stop all the containers running ``docker-compose stop`` 

# Kafka Commands
The following commands are in the bin directory of the Kafka package you downloaded in the Setup steps
Replace REPLACE-ME with the Docker (Macs boot2docker) ip address.

- To create a topic (with 1 partition and one replication/copy) ``./kafka-topics.sh --zookeeper [REPLACE-ME]:2181 --create --topic test
--partitions 1 --replication-factor 1`` 

- To see a list of available topics ``./kafka-topics.sh --zookeeper [REPLACE-ME]:2181 --list``

- To get a detailed list of topics, partitions and partition leaders ``./kafka-topics.sh --zookeeper [REPLACE-ME]:2181 --describe``

- To send a message to a topic ``echo 'Fear is the mind killer' | ./kafka-console-producer.sh --broker-list
[REPLACE-ME]:9092 --topic test``

- To send a bunch of messages via the console (press enter to send each message) ``./kafka-console-producer.sh --broker-list
[REPLACE-ME]:9092 --topic test``

- To continuously consume the latest messages from a topic and all it's partitions ``./kafka-console-consumer.sh --zookeeper [REPLACE-ME]:2181 --topic test`` 

- To consume ALL the messages from a topic and all it's partitions from the beginning (CAREFUL, there can be a lot of messages) ``./kafka-console-consumer.sh --zookeeper [REPLACE-ME]:2181 --topic test
--from-beginning``

- To consume A message from A particular topic and A particular partition ``./kafka-run-class.sh kafka.tools.SimpleConsumerShell --broker-list "[REPLACE-ME]:9092"
--max-messages 1 --topic test --partition 1 --offset 2657028``


# Notes
1. The associated local volumes/directories contain logs and data that persist across docker restarts. You'll have to delete and re-create them if you want the containers to run with a clean data/logs


# TODO
1. Currently this "Kafka DEV environment" is made of a single node zookeeper cluster and a single kafka node cluster. This is enough to play around with, but ideally this should be made of 3 node zookeeper cluster and 3 node kafka cluster

2. Expose/link the configuration directories of zookeeper and kafka containers the same way the logs and data directories are exposed. This will allow people to further experiment with various configurations
