# Kafka Development Environment
This repository gives you the ability to create a local kafka cluster for development and experimentation (NOT production)

##  Setup
1. Install [docker](https://docs.docker.com/installation/#installation) onto your system 
2. Install [docker-compose](https://docs.docker.com/compose/install/)
3. Download [0.8.2.1](http://kafka.apache.org/downloads.html) version of kafka and unzip in a location of your choice, you'll need this mainly for the kafka command/console tools
4. After cloning this repo, set up the appropriate docker-compose.yml link.   
There are 2 configurations, a single node kafka and a three node kafka (both using a single zookeeper in standalone mode)   
To use a three node kafka cluster, run ``ln -fns ./configs/three-node-kafka.yml docker-compose.yml``  
To use a single node kafka cluster, run ``ln -fns ./configs/one-node-kafka.yml docker-compose.yml``  

##  Running 
1. This step only applies to linux users, mac users should skip this. you'll need to create local directories (under /tmp/docker) that are linked to directories internally used by the containers. set up all associated local volumes/directories by running this command  
``mkdir -p `grep /tmp/docker docker-compose.yml | cut -d' ' -f6 | cut -d':' -f1 | sort` `` 
2. Ensure docker is running (mac users follow these [steps](https://docs.docker.com/installation/mac/#from-your-command-line))
3. Start all the needed containers.run ``docker-compose up -d``
4. Running ``docker-compose ps`` or ``docker ps`` should show at least a zookeeper container and 1 or 3 kafka containers

####  Set up Routing (For Mac users only)
##### The Issue
The minimal os, boot2docker, is used by osx to run docker containers. It runs as a VM on your box and then runs the docker engine/host.  
The exposed container ports can still be reached by going to the VM's ip (run `boot2docker ip`), but the issue is that Kafka nodes use a hostname/ip address to advertised themselves, this advertised address is what clients use to send and consume messages.  
Containers within boot2docker end up with ip addresses that aren't reachable from the host/local machine, this stops clients outside of the boot2docker vm from sending/consuming to and from the actual Kafka Nodes.
##### The Solution (not perfect but good enough) 
For mac users to use the kafka node hostname/ip addesses, we'll need to route the subnet ip range that the docker containers are using to the boot2docker vm ip (remember we are exposing the container ports on this ip already).  
You'll need to carry out the following steps  
1) Find out the boot2docker vm's ip by running ``boot2docker ip``   
2) Find out the subnet range being used by the containers, by running ``docker ps | cut -d' ' -f1 | tail -n +2 | xargs -I {} docker inspect --format '{{.Config.Image }} {{ .NetworkSettings.IPAddress }}' {}``  
You should see a bunch of image names and ips like
> samsara/kafka:latest 172.17.0.8  
> samsara/kafka:latest 172.17.0.7  
> samsara/kafka:latest 172.17.0.6  
> samsara/zookeeper:latest 172.17.0.5  
3) Check for any existing routing rules by running ``netstat -rn | grep 172.17``   
4) If a rule exists, just make sure you can reach the container ips and any ports.   
5) If a rule doesn't exist, create one by running ``sudo route -n add 172.17.0.0/24 $(boot2docker ip)``. Then check you can reach the container ips and ports  

To remove a routing rule run ``sudo route -n delete 172.17.0.0/24 $(boot2docker ip)`` 


##  Stopping
To stop all the containers running ``docker-compose stop``   
To remove all the containers ``docker-compose rm``  

##  Kafka commands
The following commands are in the bin directory of the kafka package you downloaded in the setup steps.

To find out the actual ips of all containers run   
``docker ps | cut -d' ' -f1 | tail -n +2 | xargs -i {} docker inspect --format '{{.config.image }} {{ .networksettings.ipaddress }}' {}``  
*In actuality these are the ip's that you should use in the following commands, but since we've exposed the ports to the docker host, we'll use the localhost for linux and $(boot2docker ip) for osx*

- **Create a topic**   
  Linux - ``./kafka-topics.sh --zookeeper localhost:2181 --create --topic test --partitions 1 --replication-factor 1``  
  Osx - ``./kafka-topics.sh --zookeeper $(boot2docker ip):2181 --create --topic test --partitions 1 --replication-factor 1``  
 you can experiment with the partition numbers and replication-factor  
 a personal rule of thumb is to have the number of partitions and replicas be multiples of the number of nodes 

- **List all available topics**   
  Linux - ``./kafka-topics.sh --zookeeper localhost:2181 --list``  
  Osx - ``./kafka-topics.sh --zookeeper $(boot2docker ip):2181 --list``  

- **Detailed list of topics, partitions and partition leaders**   
  Linux - ``./kafka-topics.sh --zookeeper localhost:2181 --describe``   
  Osx - ``./kafka-topics.sh --zookeeper $(boot2docker ip):2181 --describe``   

- **Send a message to a topic**   
  Linux - ``echo 'Fear is the mind killer' | ./kafka-console-producer.sh --broker-list localhost:9092 --topic test``  
  Osx - ``echo 'Fear is the mind killer' | ./kafka-console-producer.sh --broker-list $(boot2docker ip):9092 --topic test``  

- **To send a bunch of messages (manually) via the console (press enter to send each message)**   
  Linux - ``./kafka-console-producer.sh --broker-list localhost:9092 --topic test``  
  Osx - ``./kafka-console-producer.sh --broker-list $(boot2docker ip):9092 --topic test``  

- **To continuously consume the latest messages from a topic and all it's partitions**   
  Linux - ``./kafka-console-consumer.sh --zookeeper localhost:2181 --topic test``   
  Osx - ``./kafka-console-consumer.sh --zookeeper $(boot2docker ip):2181 --topic test``   

- **To consume ALL the messages from a topic and all it's partitions from the beginning** (CAREFUL, there can be a lot of messages)   
  Linux - ``./kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning``   
  Osx - ``./kafka-console-consumer.sh --zookeeper $(boot2docker ip):2181 --topic test --from-beginning``   

- **To consume A message from A particular topic and A particular partition from the begining**   
  Linux - ``./kafka-run-class.sh kafka.tools.SimpleConsumerShell --broker-list "localhost:9092" --max-messages 1 --topic test --partition 1 --offset -2``  
  Osx - ``./kafka-run-class.sh kafka.tools.SimpleConsumerShell --broker-list "$(boot2docker ip):9092" --max-messages 1 --topic test --partition 1 --offset -2``  

- **To consume A message from A particular topic and A particular partition from the latest**   
  Linux - ``./kafka-run-class.sh kafka.tools.SimpleConsumerShell --broker-list "localhost:9092" --max-messages 1 --topic test --partition 1 --offset -1``  
  Osx - ``./kafka-run-class.sh kafka.tools.SimpleConsumerShell --broker-list "$(boot2docker ip):9092" --max-messages 1 --topic test --partition 1 --offset -1``  


##  Notes
1. Currently the zookeeper and kafka containers use volumes that are on the local filesystem for Linux, but for Macs are inside the boot2docker vm (type `boot2docker ssh` and explore /tmp/docker). These volumes will survive container restarts and should be deleted (and recreated for linux users) if you want your containers to start with clean volumes.
2. If you need to repoint the docker-compose.yml soft link, please shutdown and remove the current configuration first ``docker-compose stop && docker-compose rm``

##  TODO
1. Add a monitoring docker (Already have the perfect candidate)
2. Expose/link the configuration directories of zookeeper and kafka containers the same way the logs and data directories are exposed. This will allow people to further experiment with various configurations
3. Explore using a data/volume container instead of volumes linked to local disk.
