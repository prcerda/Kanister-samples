# Kanister-samples
***Status:** Work-in-progress. Please create issues or pull requests if you have ideas for improvement.*

# **Kanister blueprint samples for consistent backups with Kasten**


## Summary
This project include some basic samples of Kanister Blueprints for consistent backups using Kasten.  This blueprints are in addition to the ones already published in:
* [Official Kasten documentation](https://docs.kasten.io/latest/kanister/testing.html#installing-applications-and-blueprints)
* [Kanister Github Repository](https://github.com/kanisterio/kanister/tree/master/examples)

If you want to learn about how to create your own Kanister Blueprints, you can read the [Conquer Data Consistency with Kasten K10 by Veeam and Kanister.io Blueprints](https://www.kasten.io/kubernetes/resources/how-to-guides/blueprints) whitepaper written by Michael Courcy, Kasten Solutions Architect.

## Blueprints

| Name                    | Description                                                                                                            |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| [MongoDB External DB](mongodb-logical-ext/README.md)   | Kanister blueprint to backup MongoDB external database instance, running outside the Kubernetes cluster |
