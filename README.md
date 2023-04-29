# k8s-mongo-replicaset
MongoDB Replicaset on kubernetes using statefulset with auto-failover and  auto-scaling capacity.

## About:

A working solution on how to deploy MongoDB:V6.0 ReplicaSet on Kubernetes with Auto-failover and Auto-scaling capacity.

The MongoDB server runs as 1 primary mongo node (k8s pod) and multiple secondary nodes providing auto-failover capacity if the primary node fails or to use the multiple secondary nodes (k8s pods) in a load-balanced manner for reading data when load increases.


## How Replication happens ?

+ The rs.initiate() command runs the 1st time the 1st mongoDB node (k8s pod) is deployed on kubernetes. This initiates replication of mongo.
+ In addition, a user with root privilige is created to authenticate on the replica nodes.
+ All the pods that are deployed subsequently are assigned as secondary mongo nodes.
+ The rs.add() command is used to add this secondary nodes as a part of the replicaset.
+ The rs.remove() command is used to remove a secondary mongo node from the replicaset configuration before the k8s pod terminates.


## Pre-requisities:

+ An AWS EKS cluster is preferred but can also be used on other platforms
+ An EFS Volume in the same VPC of EKS and with EKS node's Security Group added to EFS network
+ EFS CSI Driver installed on EKS

* The EFS will be used as the persistent storage for the MongoDB statefulset

## Changes that needs to be made:

_On Kustomization yaml make the following changes:_
+ Replace the filesystemID with that of the EFS you crreated
+ Replace the namespace of your choice
+ Add your preferred name-suffix
+ Set values for the min and max replica of the kubernetes HorizontalPodAutoscaling

_There are 2 secrets manifest files:_
1. Username & password For the mongoDB users. Make changes to those values as needed.
2. Credentials for the Mongo-express console and the credentials to connect to the MongoDB server.
   + Make sure the username and passwd added for connecting to the mongoDB server matches the creds of the users in mongodb.


## How to deploy ?

After making all the changes run the following command:
> kubectl apply -k .
on the base path of the repository.


## Mongo-express

I have added the manifest files needed to deploy mongo-express on kubernetes.
For more details on the environment variables please refer to this github repo:
https://github.com/mongo-express/mongo-express


## Mongo Connection String:

The MongoDB connection string to connect from within the k8s cluster will be of the format:

_mongodb://\<username\>:\<password\>@deploymongodb-0.mongodb-svc.\<namespace\>.svc.cluster.local,deploymongodb-1.mongodb-svc.\<namespace\>.svc.cluster.local,deploymongodb-2.mongodb-svc.\<namespace\>.svc.cluster.local:27017/<db_name>_

> Add ?readPreference=secondaryPreferred as query string to connect to secondary nodes for read operations.


## For more detailed understanding of this deployment porcess of MongoDB please refer to the medium doc I created:
-----------------------------------------------------------------------------------------
_https://medium.com/@ankan-devops/mongodb-replicaset-on-kubernetes-with-auto-failover-auto-scaling-d15901658136_
-------------------------------------------------------------------------------------