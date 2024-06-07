***Status:** Work-in-progress. Please create issues or pull requests if you have ideas for improvement.*

# **Kanister blueprint for MariaDB Galera databases**
Kanister blueprint to backup MariaDB Galera database instance.


## Summary
In the official Kasten documentation you can find a sample Kanister blueprint to take Logical MariaDB Backup.  That blueprint works fine for MariaDB databases running in a single node, bit it requires some modifications to make it work when running in a cluster with MariaDB Galera.  By default this blueprint will take a backup of all databases in the MariDB Galera cluster.

In this project, I'm including a sample blueprint and a blueprint binding:
* A blueprint to [backup all databases in a MariaDB Galera Cluster](bp-mariadb-galera-dump.yaml) instance.
* A blueprint binding to [apply the Blueprint to required Applications](bpb-mariadb-galera-dump.yaml) instance.


A MariaDB Galera cluster works in active-active mode, so all the nodes of the cluster are active, and all of the have the same data.  So, the backup of the database can be done from any of these nodes, as the data is always replicated.

As any MariaDB instance, you can use mysqldump to try to backup the MariaDB Galera database consistenly, but this alone isn't enough as the backup data may be not consistent and the process will slow the node and thereby affect the overall performance of the cluster.

The recommended method is to instruct MariaDB Galera to desynchronize the node you will use to run mysqldump and create the backup. This is done by setting globally the **wsrep_desync** parameter to **ON**, as you see here:
```
mysql -p -u root --execute "SET GLOBAL wsrep_desync = ON"
```

This will stop the node from processing new transactions, although it will remain part of the cluster. It will ensure consistency of data across tables while we run the backup, but continue to receive transactions from the other nodes for when we’re finished the backup process.  When it’s done, we’ll set **wsrep_desync** back to **OFF**. The node will then process any transactions that are queued and waiting to be executed on the node.

Of course, this means that the whole process must be done in a single node of the cluster:
* Set wsrep_desync to ON
* Run mysqldump
* Set wsrep_desync back to OFF

This means, you can't use the Service used to connect with MariaDB Galera pods, as this Service could connect us to different pods every time we run a mysql command from the Kanister Pod.  So, a Service must be created to connect with a single pod of the cluster, instead of connecting randomly to any of the nodes in the cluster.

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
2. You have to label the StatefulSet in order to use the Blueprint Binding with the following label: database-type=mariadb-galera.  Alternatively you can add an annotation to the StatefulSet to use the specific Blueprint without requiring a Blueprint Binding.

```
kubectl label statefulset mariadb-galera database-type=mariadb-galera -n mariadb-galera
```

**NOTE**: Replace the statefulset name and namespace according to your needs.


## Using the Kanister blueprint
In order to use this blueprint:

1. Make sure the Service mentioned in the  [prerequisites](#Prerequisites) exist in the application's namespace.  This service is called mariadb-galera-backup and it will be referenced in the blueprint.
2. Edit the Blueprint according to your needs.  Make sure the blueprint is set to connect with the Service's FQDN created in the previous step.
The sample blueprint uses the Service name "mariadb-galera-backup", so the FQDN used to connect with database will be **mariadb-galera-backup.{{ .StatefulSet.Namespace }}.svc.cluster.local**.  In case the Service has a different name, please change also the FQDN in the blueprint accordingly.

3. Create the blueprint in the Kasten namespace
```
kubectl create -f bp-mariadb-galera-dump.yaml -n kasten-io
```

4. Create the blueprint binding to use this Blueprint without the need of annotations in the application
```
kubectl create -f bpb-mariadb-galera-dump.yaml -n kasten-io
```

5. Use Kasten to backup and restore the application using a Kasten Policy and making sure you set a profile for Kanister.