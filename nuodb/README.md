# NuoDB Example

## Introduction
[NuoDB](https://github.com/nuodb/nuodb-helm-charts) is a distributed relational database management system that uses
 peer-to-peer messaging protocol to route data to nodes. In this example, we will tune a NuoDB k8s deployment using
  StormForge Optimize. 

## Prerequisites

You must have a Kubernetes cluster. We recommend using a cluster with 4 nodes, 40vCPUs (10 on each node) and 80GB of
 memory (20 on each node). Additionally, you will need a local configured copy of `kubectl` as well as the StormForge
  Optimize controller in your cluster. You can download the cli 
  [here](https://docs.stormforge.io/getting-started/install/), and then run `redskyctl init` to deploy a controller
   into your cluster. Additionally, you will also need to install [Helm3](https://helm.sh/docs/intro/install/), which
    will be used to install the core NuoDB components.
   
## NuoDB Helm Chart Installation

Before running the experiment, add the NuoDB helm repository to your local helm client:

`$ helm repo add nuodb https://storage.googleapis.com/nuodb-charts`

The NuoDB stack has four charts that need to be installed into your cluster. You can follow these
[instructons](https://github.com/nuodb/nuodb-helm-charts/blob/master/stable/README.md), or simply run the following
  commands to install the core NuoDB components into your cluster:
  
  1. `$ helm install transparent-hugepage nuodb/transparent-hugepage --set cloud.provider=<your-cloud-provider> --set cloud.zones=<your-cloud-zone>`
  2. `$ helm install admin nuodb/admin --set cloud.provider=<your-cloud-provider> --set cloud.zones=<your-cloud-zone>`
  3. `$ helm install storage-class nuodb/storage-class --set cloud.provider=<your-cloud-provider> --set cloud.zones=<your-cloud-zone>`
  4. `$ helm install database nuodb/database --set cloud.provider=<your-cloud-provider> --set cloud.zones=<your-cloud-zone>`

## Resource Tuning

Our goal is to configure our NuoDB resources to achieve the optimal combination of cost and performance. In this
 experiment we will measure performance using the `duration` template function, which computes the difference between
  the `.CompletionTime` and `.StartTime` of the Trial job. For our cost metric, we define a template function that
   maps resource usage of the NuoDB database pods to a monthly US dollar amount. Documentation on StormForge Optimize
    metrics can be found [here](https://docs.stormforge.io/metrics/).

The NuoDB database chart exposes a host of parameters that can be tuned
([chart documentation](https://github.com/nuodb/nuodb-helm-charts/blob/master/stable/database/README.md)); in this
 example we are going to tune: `persistence_size`, `sm_cpu`, `sm_memory`, `sm_replicas`, `te_cpu`, `te_memory
 `, `te_replicas`, where `sm` and `te` refer to the storage and transaction layers of the the NuoDB architecture
 , respectively.

## Trial Job

Now that we have our NuoDB components deployed in our cluster, we can proceed with running the experiment. The
 `experiment.yaml` file contains the full specification used by StormForge to optimize the appliction. Each trial
  consists of a few steps. First, the resources are patched with the updated trial parameter assignments. Once those
   resources pass readiness checks, we use the Yahoo! Cloud Serving Benchmark (YCSB) tool suite to apply an SQL
    workload against the database consisting of a 95/5% read/write split.

For more information on running, monitoring and maintaining experiments, please refer to our [quickstart](https://docs.stormforge.io/getting-started/quickstart/) and [experiment lifecycle](https://docs.stormforge.io/lifecycle/) documentation.
