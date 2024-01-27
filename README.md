# QST technical test

## Assesment

1. We need a cluster of 3 MariaDB database engines deployed on Kubernetes to take client connections (select & insert & delete queries). All database engines should have the same data(in sync). Of course, if one of the engines goes down, clients will reconnect and should be routed to one of the other instances that is up. 
   
2. How would you approach such a task? Please reply with a set (1 or many) YAML files plus any description to explain your approach/thoughts. Please submit your work even if it could be a better implementation of the requirements.

## Solution

### Summary

Since the database engine is MariaDB, I decided to use the Galera Cluster solution. The Galera Cluster can solve the problem of having a cluster of 3 MariaDB database engines. We can use the Galera Cluster to deploy one primary and two secondary nodes. The primary node will be the one that will receive the write requests, and the secondary nodes will be the ones that will receive the read requests. The Galera Cluster will be responsible for synchronizing the data between the nodes. 

In addition to the Galera Cluster, I decided to use ProxySQL to manage the connections to the database engines. The ProxySQL will be responsible for routing the requests to the primary or secondary nodes. If the primary node goes down, the ProxySQL will be responsible for routing the requests to one of the secondary nodes.

### Galera Cluster

The deployment consists of a stateful set with three replicas. The stateful set will create three pods; each pod will have a container with the MariaDB database engine and a container with the Galera Cluster. Additionally, the stateful set will have an init container responsible for configuring the Galera Cluster. The init container will configure the Galera Cluster, mounting the configuration files from the config map.

There is a volume claim template that will create a persistent volume claim for each pod. The PVC will be used to store the database engine's data.

The stateful set will create a headless service. The headless service is going to be used to provide a stable network identity to the pods. The headless service will give a DNS entry for each stateful set pod that ProxySQL will use.

I'm using a custom storage class for the persistent volume claims since I'm using the microk8s hostpath storage class. I would use a storage class in a production environment to create persistent volumes in a cloud provider.

### ProxySQL

The ProxySQL deployment consists of a deployment with three replicas. The deployment will create three pods, each with a container with the ProxySQL. The deployment will create a service that will be used to connect to the ProxySQL.

Each ProxySQL instance will be configured to connect to the Galera Cluster. The ProxySQL will be configured to route the requests to the primary node or to the secondary nodes. If the primary node goes down, the ProxySQL will be responsible for routing the requests to one of the secondary nodes.