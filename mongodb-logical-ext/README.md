***Status:** Work-in-progress. Please create issues or pull requests if you have ideas for improvement.*

# **Kanister blueprint for external MongoDB databases**
Kanister blueprint to backup MongoDB external database instance, running outside the Kubernetes cluster.


## Summary
In the official Kasten documentation you can find a sample Kanister blueprint to take Logical MongoDB Backup.  That blueprint works fine for MongoDB databases running in the same application's namespace, but requires some modifications to work with external MongoDB databases.  By default this blueprint will take a backup of all databases in the MongoDB instance.

In this project, I'm including two sample blueprints for external MongoDB databases:
* A blueprint to [backup all databases in a MongoDB](mongodb-ext-blueprint-alldbs.yaml) instance.
* A blueprint to [backup a specific database in a MongoDB](mongodb-ext-blueprint-singledb.yaml) instance.


## Disclaimer
This project is an example of an deployment and meant to be used for testing and learning purposes only. Do not use in production. 

This blueprint has been tested with MongoDB 7.0.


# Table of Contents

1. [Prerequisites](#Prerequisites)
2. [Blueprint Variables](#Blueprint-Variables)
3. [Using the Kanister blueprint](#Using-the-Kanister-blueprint)


## Prerequisites
To use this blueprint you need the following:
1. A secret containing the credentials to connect with the MongoDB instance, usually the one used by the kubernetes application to connect with the MongoDB database.  A sample secret YAML file has been included in this project.
1. A service of type **externalName** to connect with the external MongoDB instance using its FQDN.
1. (Just when trying to backup all MongoDB databases) MongoDB Oplog must be enabled.

## Blueprint Variables

Some of the tasks or commands run in the blueprint require the use of some variables, which should be properly set according to your needs:

| Name                    | Type     | Default value         | Description                                                                                                            |
| ----------------------- | -------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `mongosecret`           | Secret   | `mongodb-secret`      | For backup and restore actions, it's required to set the name for the Secret with the credentials to connect with MongoDB instance             |
| `host `                 | String   | `external-mongodb.externaldb.svc.cluster.local`   | FQDN for the Service of type externalName mentioned in the  [prerequisites](#Prerequisites)   |
| `port`                  | String   | `27017`               | TCP port used by MongoDB instance                                                                      |
| `user`                  | String   | `username`            | String containing the user name in the Secret  mentioned in the  [prerequisites](#Prerequisites)   |
| `password`              | String   | `password`            | String containing the password in the Secret  mentioned in the  [prerequisites](#Prerequisites)   |


**NOTE**: It's important to highlight that this blueprint assumes that the application running in the Kubernetes cluster, and connected with the external MongoDB database, has been configured as a **"Deployment"**.  In case of using another kind of workload, like a StatefulSet, you must modify the blueprint accordingly.


## Using the Kanister blueprint
In order to use this blueprint:

1. Make sure the Secret and the Service mentioned in the  [prerequisites](#Prerequisites) exist in the application's namespace.
2. Edit the Blueprint according to your needs as mentioned in [Blueprint Variables](#Blueprint-Variables)
3. Create the blueprint in the Kasten namespace
```
kubectl create -f mongodb-ext-blueprint-alldbs.yaml -n kasten-io
```

4. Annotate the application (deployment) with the correct annotation to instruct K10 to use the Blueprint (mongodb-ext-blueprint)
```
kubectl annotate deployment application-name kanister.kasten.io/blueprint='mongodb-ext-blueprint' --namespace=application-namespace
```
5. Use Kasten to backup and restore the application using a [Kasten Policy](mongodb-ext-blueprint)