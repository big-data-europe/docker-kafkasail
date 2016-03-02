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

## Installation 

The installation of this sample application requires three steps, first the installation of kafka, second the creation of a suitable Kafka topic and thirdly the installation and bootstrapping of KafkaSail.
