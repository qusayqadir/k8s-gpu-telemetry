# k8s-gpu-telemetry

End-to-end AWS telemetry pipeline that monitors a fleet of simulated ML training
jobs, optimizes them for performance, cost, and efficiency, and monitors itself.


## Why

GPUs are expensive, and a common way to waste them is **starvation**: the input
pipeline can't feed data fast enough, so the GPU sits idle while you pay for it.
Raw GPU metrics don't tell you *why* that's happening or *what it costs*. This
project ties  GPU metric to its workload context and a cost model, then
automatically scales the input pipeline to minimize cost per training step.

## Goals 

answer the following questions in real time

1. **Performance:** How well is each GPU performing? Model FLOPs utilization (MFU)
   tracks useful compute against the GPU's peak, alongside data-wait time,
   utilization, and temperature.
2. what is the cost of having idle GPU time, and the tradeoffs required to reduce the cost 
3. how can i improve the efficiency of the gpu required workload 
4. how to build an optimizer that can fix the GPU starvation by autoscaling data-server replicas when GPU data-wait time exceeds a threshold (@ three differnet layers)

### Optimizer 

The goal is **lowest cost per training step**, not maximum GPU utilization. Extra
capacity costs money, so it's only worth adding if it recovers more idle GPU cost
than it adds. The optimizer acts in three levels, cheapest and fastest first:

| Level | Action | Speed | When |
|---|---|---|---|
| 1. Application | Data server adds worker threads and prefetch depth | Instant | Data-wait high, server CPU below its limit |
| 2. Pods | KEDA (kubernetes event driven autoscaling) adds data-server replicas on existing nodes | Seconds | Data-wait high, server CPU at its limit |
| 3. Nodes | Karpenter adds a CPU node for pending pods | Minutes | Pods pending, no node capacity left |


### Dashboard (Grafnaa)
- **Fleet performance**: MFU and data-wait time per trainer, plus GPU utilization,
  memory, temperature, power, and ECC (error-correcting code) errors
- **Cost**: cost of gpu wait-time, cost of scaling timeline, cost of infrastructure 
- **scaling timeline** how long does it take for the infrastructure to catch up to the lag of data the trainer is processing fmor the datas server. 
- **pipeline health**: the aws services needs to have cloudwatch to measure the pipleine health iteself. 


## Usage

    make up      # terraform apply + deploy pods
    make job JOB=resnet50-sim    # start a simulated training job
    make experiment # baseline vs optimizer run 
    make down    # tear everything down

**Stretch**
- Windowed aggregations / anomaly detection with Flink
- Validate the mock schema against a real GPU node running NVIDIA DCGM exporter


**Goals for learning**
- building and tearing down aws infrastructure using terraform 
- design a private VPC, VPC endpoints, subnets, securirty groups 
- learn K8s:
    - running workloads 
    - how does cross pod communication work in K8s 

- stream telemetry of data servers nodes and trainer nodes for observability and design kafka topics to ingest those while handling Dead-Letter Queue, Idempotency. 

- Observability / site reliability 
    - monitoring a training pipeline, and then monitoring the pipeline thats built on top of it 
    - learn latency optimization techniques

- Reliability 
    - run failure drills, load tests
    - design controlled experiment 

