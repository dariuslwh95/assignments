# Describe the architecture of a minimal high-availability web server. Explain the mechanism by which the setup will handle infrastructure failures.

_This answer is focused on the scenario of a stateless web server application where there are no application errors, failures relate to the kubernetes platform and the underlying infrastructure, which is deployed on-prem using VMware vSphere._

## 1. HA Cluster Architecture
As a best practice, an active passive setup of 2 clusters in separate availability zones (AZ) should be maintained in the event of entire-AZ failures and prolonged instability if the below measures are ineffective. Traffic to the web application can be routed to the other cluster via an external load balancer e.g. F5 for on-prem applications. 

While AZs are feature of the cloud services, in an on-prem deployment, AZs can be created as logical groups of MachineSets (), matching node labels. The adding of additional nodes to a cluster must be done manually in an on-prem deployment as compared with true cluster-autoscaling in cloud service deployments.

The manual activity of adding a new node to a cluster, be it worker or control-plane can be scripted and executed as a jenkins pipeline as well. 

Monitoring components such as the node-exporter, kube-state-metrics and prometheus should be deployed in the cluster for observability and setting up of triggers to replicate the cluster-autoscaling functionality.

An external health check script can be run from a jenkins worker every set interval e.g. 5 mins to query the prometheus endpoint for metrics, and given specific conditions e.g. capacity, node irresponsiveness, trigger an alert to seek approval to add nodes to the cluster. 
```
# promql for cpu % usage by node
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
# promql for mem % usage by node
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
# node status
up{job="node_exporter"}
```

## 2. Handling Worker Node Failures
After a configured timeout where a node is unresponsive, the node controller marks the node as NotReady and begins evicting pods from the problematic worker node. Given this, ensure that the web server is deployed using a Deployment object to allow automatic rescheduling on healthy worker nodes.

For worker node failures, if the application is still able to cope with the load and there are no flipped requests, cluster admins should first attempt manual remediation through logging and debugging given the cluster is still accessible, checking for stuck clusteroperators, jobs on the NotReady node, perform cleanups and restarts if applicable before uncordon-ing the node.

In the event the external script is triggered to seek approval, and all attempts at manual remediation have failed, a new worker node can be approved to be added as a replacement and pods will be scheduled on the new worker node.

## 3. Handling Control Plane Node Failures
etcd backups should be taken on a regular basis via a cronjob and stored in an accessible location such as NAS backed sdb storage or an on-prem MinIO deployment. 

In the event of control plane node failures, the cluster may be inaccessible due to kube-apiserver pods being affected, as such, direct ssh will be required with the kubeconfig being available in an available location to perform debugging activities.

In the event the external script is triggered to seek approval, and all attempts at manual remediation have failed, a new control-plane node can be approved to be added as a replacement. 

After, if there is etcd data corruption or loss of namespace data observed, restore etcd from the snapshot. 

## 4. Handling Persistent Storage Failures
Ensure redundancy through backup solutions such as velero or ceph and make use of volumesnapshots which can be executed via cronjobs at regular intervals. 
