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
4. Demonstrate simple query

#### Build Docker Image with Dataset (Optional)
1. Create a path with `dataset` and `Dockerfile` with content:
**MongoDB**
```
From Mongo
COPY <source path>.csv <dataset path>.csv
```
**Cassandra**
```
FROM Cassandra
COPY <source path>.csv <dataset path>.csv
```
2. Build the docker image with `docker build -t <username>\<image name>`
3. Login to your Docker Account with `docker login`. If you have already logged in, skip this step.
4. Push image to Docker Hub with `docker push <username>\<image name>`

#### Setup and deploy MongoDB and Cassandra using Docker container in Google Cloud Platform
**MongoDB**
1. Create a *Container OS* VM instance in Google Cloud Platform (GCP). (Make sure to turn on IP Forwarding in network settings)
2. Run the following commands in *VM Shell* to setup Mongo Server in GCP and import dataset into Mongo Server
```
docker pull jeffhow/mongo
docker network create mongo
docker run --name my-mongo --network mongo -p 27017:27017 -d jeffhow/mongo
docker exec -it my-mongo bash
mongoimport -d mydb -c things --type csv --file crimes.csv --headerline
mongo
show dbs
use mydb
db.getCollectionNames()
db.things.find().pretty()
```

**Cassandra**
1. Create a *Container OS* VM instance in Google Cloud Platform (GCP). (Make sure to turn on IP Forwarding in network settings)
2. Run the following commands in *VM Shell* to setup Cassandra 2 Nodes Cluster in GCP and import dataset into Cassandra Cluster
```
docker pull jeffhow/cassandra
docker network create cassandra
docker run --name my-cassandra1 --network cassandra -p 9042:9042 -p 9160:9160 -d jeffhow/cassandra
docker run --name my-cassandra2 --network cassandra -e CASSANDRA_SEEDS=my-cassandra1 -d jeffhow/cassandra
docker run -it --network cassandra jeffhow/cassandra cqlsh my-cassandra1

CREATE KEYSPACE IF NOT EXISTS TestDB WITH replication = {'class':'SimpleStrategy', 'replication_factor':3};
USE TestDB;
CREATE TABLE IF NOT EXISTS crimestable (Id int PRIMARY KEY, Arrest boolean, Beat int, Block text, CaseNumber text, CommunityArea float, Date timestamp, Description text, District text, Domestic boolean, FBICode text, IUCR text, Latitude float, Location text, LocationDescription text, Longitude float, PrimaryType text, UpdatedOn timestamp, Ward float, XCoordinate float, YCoordinate float, Year int) WITH comment='Crimes_Dataset';
COPY crimestable (Id, Arrest, Beat, Block, CaseNumber, CommunityArea, Date, Description, District, Domestic, FBICode, IUCR, Latitude, Location, LocationDescription, Longitude, PrimaryType, UpdatedOn, Ward, XCoordinate, YCoordinate, Year) FROM '/crimes.csv' WITH DELIMITER=',' AND HEADER=TRUE AND DATETIMEFORMAT='%m/%d/%Y %H:%M:%S %p';
```
