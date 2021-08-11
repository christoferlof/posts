---
title: Querying AKS pod perf counters by Kubernetes namespace in Azure Log Analytics
layout: post
tags: azure
excerpt: If you're using Azure Kubernetes Service (AKS), chances are you've also setup Azure Monitor to understand the performance and health of your Kubernetes cluster and the container workloads. Learn how to query perf counters for a pod filtering by namespace
---

If you're using Azure Kubernetes Service (AKS), chances are you've also setup [Azure Monitor to understand the performance and health of your Kubernetes cluster and the container workloads](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-overview).

To increase the density of your deployments, changes are also that you're deploying multiple versions of your application into the same Kubernetes cluster and [separating these deployments by leveraging namespaces.](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

Now, when you want to query Log Analytics to understand resource utilization such as CPU and Memory by pod, you also need to take the namespace into consideration which the pod is deployed into or you'll get a total for the pod across all namespaces where it's deployed.

Performance counters are collected into the `Perf` table in Log Analytics. Unfortunately the namespace name is not part of the record why you can't directly filter the perf records by Kubernetes namespace. But if you look at the `InstanceName` and the URI segment just before the pod name, is the pod id. Using this, you can join with the `KubePodInventory` table in which you can filter by namespace.

To show a graph like this one, displaying your pod memory usage and limits..  
![Azure Dashboard Graph showing pod memory usage and limit](/assets/aks-monitor.png)

.. the query can look something like this

```plaintext
Perf
| where InstanceName contains "the-service" and CounterName contains "memory"
| project TimeGenerated, CounterName, CounterValue, InstanceName, PodUid=tostring(split(InstanceName, "/", 9)[0])
| join kind= inner
(
    KubePodInventory
    | project PodUid, ContainerName, Namespace
)
on PodUid
| where Namespace == "the-namespace-of-feature-branch-one"
| summarize avg(CounterValue) by bin(TimeGenerated, 10m), CounterName
| render timechart
```
