***Status:** Work-in-progress. Please create issues or pull requests if you have ideas for improvement.*

# **Kanister blueprint for MariaDB Galera databases**
Kanister blueprint to backup MariaDB Galera database instance.


## Summary
In the official Kasten documentation you can find a sample Kanister blueprint to take Logical MariaDB Backup.  That blueprint works fine for MariaDB databases running in a single node, bit it requires some modifications to make it work when running in a cluster with MariaDB Galera.  By default this blueprint will take a backup of all databases in the MariDB Galera cluster.

In this project, I'm including a sample blueprint and a blueprint binding:
* A blueprint to [backup all databases in a MariaDB Galera Cluster](bp-mariadb-galera-dump.yaml) instance.
* A blueprint binding to [apply the Blueprint to required Applications](bpb-mariadb-galera-dump.yaml) instance.


A MariaDB Galera cluster works in active-active mode, so all the nodes of the cluster are active, and all of the have the same data.  So, the backup of the database can be done from any of these nodes, as the data is always replicated.  
The whole process must be done in a single node of the cluster, so a Service must be created to connect with a single pod of the cluster, instead of connecting randomly to any of the nodes in the cluster.

More information about MariaDB Galera cluster backup, can be found in the following link: [https://galeracluster.com/library/training/tutorials/galera-backup.html](https://galeracluster.com/library/training/tutorials/galera-backup.html)


## Disclaimer
This project is an example of an deployment and meant to be used for testing and learning purposes only. Do not use in production. 

This blueprint has been tested with **MariaDB Galera 11.3.2**.


# Table of Contents

1. [Prerequisites](#Prerequisites)
2. [Using the Kanister blueprint](#Using-the-Kanister-blueprint)


## Prerequisites
To use this blueprint you need the following:
1. A service of type **ClusterIP** to connect with a single pod of the MariaDB Galera Cluster. In this repository you can find a [YAML manifest to create this Service](mariadb-svc.yaml).
2. You have to label the StatefulSet in order to use the Blueprint Binding with the following label: database-type=mariadb-galera

```
kubectl label statefulset mariadb-galera database-type=mariadb-galera -n mariadb-galera
```

NOTE: Replace the statefulset name and namespace according to your needs.


## Using the Kanister blueprint
In order to use this blueprint:

1. Make sure the Service mentioned in the  [prerequisites](#Prerequisites) exist in the application's namespace.
2. Edit the Blueprint according to your needs.  Make sure the blueprint is set to connect with the Service's FQDN.

The sample blueprint uses the Service FQDN: mariadb-galera-backup.mariadb-galera.svc.cluster.local.  Please change this according to your needs.

3. Create the blueprint in the Kasten namespace
```
kubectl create -f bp-mariadb-galera-dump.yaml -n kasten-io
```

4. Create the blueprint binding to use this Blueprint without the need of annotations in the application
```
kubectl create -f bpb-mariadb-galera-dump.yaml -n kasten-io
```

5. Use Kasten to backup and restore the application using a Kasten Policy and making sure you set a profile for Kanister.