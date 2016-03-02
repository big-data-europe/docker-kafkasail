# KafkaSail docker

Docker image containing an Apache Tomcat with OpenRDF's (rdf4j) openrdf-workbench and openrdf-sesame server pre installed. The image also contains a script to bootstrap a KafkaSail Repository (see below).

Important note: KafkaSail is meant for demonstration purposes only (as of writing).

Versions used in this docker image:
* Java 1.8.0_72
* Apache Tomcat: 7.0.59
* OpenRDF-Workbench: 2.8.9
* OpenRDF-Sesame-Server: 2.8.9
* SWC-KafkaSail: 1.0.0

Image details:
* Installation directory: /usr/local/kafkasail/apache-tomcat-7.0.59-rdf4j/
* Default OpenRDF sesame root directory: /usr/local/kafkasail/apache-tomcat-7.0.59-rdf4j/data/

## Installation on Marathon

The installation of this sample application requires three easy steps, first the installation of kafka, second the creation of a suitable Kafka topic and thirdly the installation and bootstrapping of KafkaSail.

To install Kafka

* To install Kafka follow the steps outlined in "To start Kafka Docker image on Marathon" here : https://github.com/big-data-europe/docker-kafka
* For the next step we will assume that there are three Kafka brokers running on bigdata-one.example.com:9092, bigdata-two.example.com:9092 and bigdata-three.example.com:9092. Change these urls according to your local environment.

To create a suitable Kafka topic

* To create a suitable Kafka topic follow the steps outlined in "To create a Kafka topic" here : https://github.com/big-data-europe/docker-kafka
* In the next steps it is assumed that the Kafka topic is "ksail". The KafkaSail sample has been tested with three partitions and a replication-factor of 1. 

To install KafkaSail

* Clone this repository on all nodes of the test cluster and build the image using
 
 ```bash
cd /path/to/kafkasail/cloneroot/
docker build -t turnguard/kafkasail .
```

To run and bootstrap KafkaSail example on Marathon

* For this step the following information is required:
  * broker urls (see above, we will assume bigdata-one.example.com:9092, bigdata-two...)
  * topic name (see above, we will assume ksail)
  * zookeeper url with chroot (e.g. 192.168.88.219:2181,192.168.88.220:2181,192.168.88.221:2181/kafka)
* Create a Marathon Application Setup in json like the one below, store it in a file (e.g. marathon-kafkasail.json). Do not post it yet, there are some environment variables to be set.

 ```json
 {
    "container": {
        "type": "DOCKER",
        "volumes": [
        {
                "containerPath": "/usr/local/kafkasail/apache-tomcat-7.0.59-rdf4j/data/",
                "hostPath": "/var/lib/bde/kafkasail/",
                "mode": "RW"
        }
        ],
        "docker": {
            "network": "BRIDGE",
            "image": "turnguard/kafkasail",
            "privileged":false,
            "portMappings": [
              { "containerPort": 8080, "hostPort": 0}
            ]
        }
    },
    "env": {
        "KAFKASAIL_REPOSITORY_ID": "KafkaSail",
        "KAFKASAIL_BOOTSTRAP_SERVERS": "bigdata-one.example.com:9092,bigdata-two.example.com:9092,bigdata-three.example.com:9092",
        "KAFKASAIL_ZK_HOST": "192.168.88.219:2181,192.168.88.220:2181,192.168.88.221:2181/kafka",
        "KAFKASAIL_TOPIC":"ksail"
    },
    "id":"turnguard-kafkasail",
    "cpus": 0.2,
    "mem": 512,
    "cmd": "cd /usr/local/kafkasail/apache-tomcat-7.0.59-rdf4j/kafkasail/bin && ./bootstrap && cd ../../ && ./bin/catalina.sh run",
    "instances":1,
    "constraints":[["hostname","UNIQUE",""]]
}
```

* note that in the above example KafkaSail's data directory is mounted on the host.
* change the value for KAFKA_BOOTSTRAP_SERVERS according to your local environment.
* change the value for KAFKA_ZK_HOST to your local environment.
* change the value for KAFKA_TOPIC to your local environment.
* note that the bootstrap script, that is included inside the tomcat package will create a KafkaSail repository according to those environmental variables.
* post the above Marathon Application Setup to Marathon's v2/app endpoint
* Scale the application inside the Marathon GUI to a number of servers. in this example we will assume that the app is scaled to run on three servers (bigdata-one.example.com, bigdata-two.example.com and bigdata.three.example.com with a port assigned by Mesos)
 
To verify KafkaSail (fun part)

* Open all instances of KafkaSail in different tabs in a browser of your choice, appending /openrdf-workbench to the specific url. if for example the KafkaSail docker container is running on bigdata-one.example.com:31500, then the OpenRDF Workbench' url would be bigdata-one.example.com:31500/openrdf-workbench
* You now should see OpenRDF Workbench' start page on all three tabs displaying a list of available repositories, click on KafkaSail repository (if you didn't change KAFKASAIL_REPOSITORY_ID in the previous step). This will take you the page displaying basic info about the current repository. Specifically you should find the "Number of statements" to be 0 under "Repository Size".
