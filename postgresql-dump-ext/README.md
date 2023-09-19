***Status:** Work-in-progress. Please create issues or pull requests if you have ideas for improvement.*

# **Kanister blueprint for external PostgreSQL databases**
Kanister blueprint to backup PostgreSQL external database instance, running outside the Kubernetes cluster.


## Summary
In the official Kasten documentation you can find a sample Kanister blueprint to take Logical PostgreSQL Backup.  That blueprint works fine for PostgreSQL databases running in the same application's namespace, but requires some modifications to work with external PostgreSQL databases.  By default this blueprint will take a backup of all databases in the PostgreSQL instance.

In this project, I'm including two sample blueprints for external PostgreSQL databases:
* A blueprint to [backup all databases in a PostgreSQL](postgresql-ext-blueprint-alldbs.yaml) instance.
* A blueprint to [backup a specific database in a PostgreSQL](postgresql-ext-blueprint-singledb.yaml) instance.


## Disclaimer
This project is an example of an deployment and meant to be used for testing and learning purposes only. Do not use in production. 

This blueprint has been tested with **PostgreSQL 15.4 for Linux**.


# Table of Contents

1. [Prerequisites](#Prerequisites)
2. [Blueprint Variables](#Blueprint-Variables)
3. [Using the Kanister blueprint](#Using-the-Kanister-blueprint)


## Prerequisites
To use this blueprint you need the following:
1. A service of type **externalName** to connect with the external PostgreSQL instance using its FQDN.
2. A secret containing the credentials to connect with the PostgreSQL instance, usually the one used by the kubernetes application to connect with the PostgreSQL database.  A sample secret YAML file has been included in this project, and the following data should be included:

| Name                    | Type     | Default value         | Description                                                    |
| ----------------------- | -------- | --------------------- | -------------------------------------------------------------- |
| `username`              | String   | `kastenbp`            | PostgreSQL user name                                              |
| `password`              | String   | `Veeam123!`           | PostgreSQL password                                               |
| `host `                 | String   | `external-postgresql.appexternaldb.svc.cluster.local`   | FQDN for the Service of type externalName mentioned as a pre-requisite  |
| `port`                  | String   | `5432`               | TCP port used by PostgreSQL instance                              |
| `db_name`               | String   | `kastenbp`           | DB to be protected, only required for single database backup   |


3. The PostgreSQL instance should allow for remote connections and for password authentication using any of the methods supported by PostgreSQL: https://www.postgresql.org/docs/current/auth-password.html.


## Blueprint Variables

In the blueprint, it's required to provide the following data


| Name                    | Type     | Default value         | Description                                                                                                            |
| ----------------------- | -------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Secret Name             | Secret   | `postgresql-secret`      | For backup and restore actions, it's required to set the name for the Secret with the credentials to connect with PostgreSQL instance            |
| Namespace               | String   | `{{ .Deployment.Namespace }}`   | By default the namespace's name will be got from the application itself.  In this case the application was deployed as a Deployment, but it could be also a Stateful, and in that case you should modify the blueprint accordingly   |



**NOTE**: It's important to highlight that this blueprint assumes that the application running in the Kubernetes cluster, and connected with the external PostgreSQL database, has been configured as a **"Deployment"**.  In case of using another kind of workload, like a StatefulSet, you must modify the blueprint accordingly.


## Using the Kanister blueprint
In order to use this blueprint:

1. Make sure the Secret and the Service mentioned in the  [prerequisites](#Prerequisites) exist in the application's namespace.
2. Edit the Blueprint according to your needs as mentioned in [Blueprint Variables](#Blueprint-Variables)
3. Create the blueprint in the Kasten namespace
```
kubectl create -f postgresql-ext-blueprint-alldbs.yaml -n kasten-io
```

4. Annotate the application (deployment) with the correct annotation to instruct K10 to use the Blueprint (postgres-ext-bp)
```
kubectl annotate deployment application-name kanister.kasten.io/blueprint='postgres-ext-bp' --namespace=application-namespace
```
5. Use Kasten to backup and restore the application using a [Kasten Policy](postgres-ext-bp)