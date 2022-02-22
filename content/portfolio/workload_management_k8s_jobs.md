---
title: Job management with Kubernetes
image: images/k8s_small.png
date: "2021-08-20T00:00:00"
tags:
  - kubernetes
  - long running job
  - job management
  - cloud native
  - scaling
  - python
summary: "**Kubernetes** is the leading container orchestration platform. This case study focuses on the **Job** resource kind and how it could be used to help with long running jobs."
---
Our client in focus was utilising Machine Learning models on 8x GPU machines to serve customer orders ( deepfake workloads with virtual models ). Their naive solution was working - running as a service continuously costing $10k+ per month for a single machine.
They seriously needed an on-demand scheduling solution as a startup they lacked the constant load and the machines were very underutilised.

The solution was found with the help of a Kubernetes [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) and to rewrite their old worker service to a 'work to completion' container. Their old service split in half to a simple custom workload scheduler - which only created the kubernetes jobs - and the rest of their code which was responsible for the computation.
Once an order arrived to the system, the job scheduler was able to translate it to a Job by directly talking to the [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/). The Job resource had [Node Selectors](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector) and [Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#concepts) attached to run it only on certain type of machine ([preemptible](https://cloud.google.com/kubernetes-engine/docs/how-to/preemptible-vms) 8x GPU), triggering the autoscaling on a node pool, which scaled back to zero having no workloads.
Their whole monthly GCP running cost were instantly crippled to under $400 and if more orders arrived at the same time there was no waiting for their customer as multiple Jobs could be scheduled parallel following their desired limits if needed
