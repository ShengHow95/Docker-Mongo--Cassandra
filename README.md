[![Docker badge]https://img.shields.io/badge/Docker-up-green
[![Mongo badge]https://img.shields.io/badge/Mongo-up-green
[![Cassandra badge]https://img.shields.io/badge/Cassandra-up-green
[![GCP badge]https://img.shields.io/badge/GoogleCloudPlatform-up-green
[![Python badge]https://img.shields.io/badge/Python-up-green

## Docker-Mongo-Cassandra

Contributed by:
1. Kong Sheng How
2. Chu Kah Em
3. Teoh Rui Wen
4. Kng Chun Keat
5. Cheng Xi Yang

Simple Demo Project setting up MongoDB server and Cassandra cluster using Docker in Google Cloud Platform (GCP). 
Dataset Used: [Chicago Crimes](https://www.kaggle.com/currie32/crimes-in-chicago#Chicago_Crimes_2008_to_2011.csv)

Docker Image Used:  

**MongoDB**  
Image with Crimes Dataset: [jeffhow/mongo](https://hub.docker.com/r/jeffhow/mongo)  
Base Image: [mongo](https://hub.docker.com/_/mongo)  

**Cassandra**  
Image with Crimes Dataset: [jeffhow/cassandra](https://hub.docker.com/r/jeffhow/cassandra)  
Base Image: [cassandra](https://hub.docker.com/_/cassandra)  

### Project Overview
1. Build Docker Image with Dataset
2. Setup and deploy MongoDB and Cassandra using Docker container in Google Cloud Platform
3. Connect to Docker Container Using Python

#### Build Docker Image with Dataset (Optional)
1. Create a path with `dataset` and `Dockerfile` with content:  
**MongoDB**
```
# Use mongo Image as base image
From mongo

# Copy Dataset from Local Folder to Container
COPY <source path>.csv <container path>.csv
```

**Cassandra**
```
# Use cassandra Image as base image
FROM cassandra

# Copy Dataset from Local Folder to Container
COPY <source path>.csv <container path>.csv
```

2. Build the docker image. `docker build -t <username>\<image name>`
3. Login to your Docker Account. `docker login` (If you have already logged in, skip this step)
4. Push image to Docker Hub. `docker push <username>\<image name>`

#### Setup and deploy MongoDB and Cassandra using Docker container in Google Cloud Platform
**MongoDB**
1. Create a *Container OS* VM instance in Google Cloud Platform (GCP). (Make sure to turn on IP Forwarding in network settings)
2. Run the following commands in *VM Shell* to setup Mongo Server in GCP and import dataset into Mongo Server
```
# Pull Image from Docker Repos
docker pull jeffhow/mongo

# Create dedicated network for mongo container
docker network create mongo

# Run jeffhow/mongo image
docker run --name my-mongo --network mongo -p 27017:27017 -d jeffhow/mongo

# Go into Container Bash Shell
docker exec -it my-mongo bash

# Import data into Mongo Server
mongoimport -d mydb -c things --type csv --file crimes.csv --headerline

# Start Mongo Client within Container Bash Shell
mongo

# Verify dataset is completely imported
show dbs
use mydb
db.getCollectionNames()
db.things.find().pretty()
```

**Cassandra**
1. Create a *Container OS* VM instance in Google Cloud Platform (GCP). (Make sure to turn on IP Forwarding in network settings)
2. Run the following commands in *VM Shell* to setup Cassandra 2 Nodes Cluster in GCP and import dataset into Cassandra Cluster
```
# Pull image from docker repos
docker pull jeffhow/cassandra

# Create dedicated network for cassandra containers
docker network create cassandra

# Create and run First Cassandra Node Container 
docker run --name my-cassandra1 --network cassandra -p 9042:9042 -p 9160:9160 -d jeffhow/cassandra

# Create and run Second Cassandra Node Container 
docker run --name my-cassandra2 --network cassandra -e CASSANDRA_SEEDS=my-cassandra1 -d jeffhow/cassandra

# Create and run another Cassandra container as CQLSH Client 
docker run -it --network cassandra jeffhow/cassandra cqlsh my-cassandra1

# Import Dataset into Cassandra Cluster
CREATE KEYSPACE IF NOT EXISTS TestDB WITH replication = {'class':'SimpleStrategy', 'replication_factor':3};

USE TestDB;

CREATE TABLE IF NOT EXISTS crimestable (Id int PRIMARY KEY, Arrest boolean, Beat int, Block text, CaseNumber text, CommunityArea float, Date timestamp, Description text, District text, Domestic boolean, FBICode text, IUCR text, Latitude float, Location text, LocationDescription text, Longitude float, PrimaryType text, UpdatedOn timestamp, Ward float, XCoordinate float, YCoordinate float, Year int) WITH comment='Crimes_Dataset';

COPY crimestable (Id, Arrest, Beat, Block, CaseNumber, CommunityArea, Date, Description, District, Domestic, FBICode, IUCR, Latitude, Location, LocationDescription, Longitude, PrimaryType, UpdatedOn, Ward, XCoordinate, YCoordinate, Year) FROM '/crimes.csv' WITH DELIMITER=',' AND HEADER=TRUE AND DATETIMEFORMAT='%m/%d/%Y %H:%M:%S %p';
```

#### Connect to Docker Container Using Python
1. Make sure to install the following python drivers:  

**Mongo**
```pip install pymongo```

**Cassandra**
```pip install cassandra-driver```

2. Refer to Project Files to perform some simple queries. (Change the IPAddress to your *VM External IPAddress* or *localhost* if you run the contaniers in your own local computer)
