---
title: Workflow management with Argo Workflows
image: images/argo.png
date: "2022-02-05T00:00:00"
tags:
  - argo
  - kubernetes
  - workflow management
  - cloud native
summary: "**Argo workflows** is part of the Argo family, promotes itself as **the workflow engine for kubernetes**. It's usage purposes can be very similar to Apache AirFlow, to setup your pipelines/transformations to follow a certain pattern (DAG - Directed Acyclic Graph)
It runs on kubernetes and is fully cloud-native"
---
[Argo workflows](https://argoproj.github.io/argo-workflows/) is part of the Argo family, promotes itself as *the workflow engine for kubernetes*. It's usage purposes can be very similar to Apache AirFlow, to setup your pipelines/transformations to follow a certain pattern (DAG - Directed Acyclic Graph)
It runs on kubernetes and is fully cloud-native. While Airflow would only run Python code, Argo WF runs containers with any language 
<!-- more -->

In this case, the client needed such a solution, where 
1. a single "order" had many steps of long running jobs 
2. the solutions had to handle scaling to a significant amount of parallel orders (eg single server with pulling queue would not work)
3. the long running jobs had dependencies on each other, but were sometimes optional
4. it would not have been cost effective to keep many servers constantly running, as orders came as swarms and were
5. depending on the size of input data different machines should be utilised for further cost optimisation
6. have multiple environments of the system, which reduces the risk for changes thus enables moving faster

As mentioned before, the combination of Argo Workflows and the flexibility of Kubernetes provided a perfect solution.
To achieve the exact rules for the business logic of dependendent jobs was very flexible. Argo WF also provided options for failed workload paths with [exit handlers](https://github.com/argoproj/argo-workflows/blob/master/examples/exit-handlers.yaml) (eg wrong input data)
With certain strategies used in Kubernetes ( [Node Selector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector) / Affinity rules combined with [Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#concepts) ) a new machine only lived until the work has been completed.
With Kubernetes (GKE) autoscaling set to "[optimised for utilisation](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler#autoscaling_profiles)" the waste of scaling down after a completed job was greatly reduced.
Additionally all the job are being completed on [preemptible](https://cloud.google.com/kubernetes-engine/docs/how-to/preemptible-vms) machines which further reduces the compute bill by close to 80%.
The [declarative nature of kubernetes](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/) made it fairly easy to run up multiple environments
