---
title: "QOS-based GPU Allocation"
permalink: /docs/misha-qos/
excerpt: "Qos-based GPU allocation on Misha"
last_modified_at: 2021-06-07T08:48:05-04:00
#redirect_from:
#  - /theme-setup/
toc: true
---

## Introduction

Misha is configured with a simple partition structure, with no PI partitions to segment the larger `gpu` partition shared by all Misha users. 

To accommodate contributors' needs to run jobs "immediately" on their GPUs, we have designed and implemented 
a new SLURM resource allocation protocol that allows contributors to gain priority access to their share of
the GPU resources with the capability to preempt non-priority jobs when necessary.

The new protocol doesn't physically partition the GPU nodes based on the number of nodes each PI purchased. 
Instead, it defines a contributor's share of the GPUs based on the <b>number</b> and the <b>type</b> of GPU cards the PI 
has purchased, regardless of which node the cards are physically located in. Information about GPU card numbers and types 
is encapsulated in a <b>QOS (Quality of Service)</b> tag designed specifically for each contributor group, which can only be accessed by group members.

## How to submit a priority job
To submit a priority job, you must specify the QOS name of your PI group and also specify the <b>type</b> and the <b>number</b> of GPUs you will use. For example,

```<bash>
#SBATCH --qos=qos_lafferty,--gres=gpu:a40:2
```
Using a QOS tag without specifying a GPU type is not permitted. The following example will be rejected by SLURM:

```<bash>
#SBATCH --qos_qos_lafferty,--gres=gpu:2
```
If all the GPUs in a group are being used by the group members, a new priority job will be queued. This means that even 
if the same type of GPUs are avaialbe on the cluster, the job will be queued because it cannot surpass the group limit.

If some GPUs in the group are being used for regular jobs and preempting one or more of those regular jobs will free up enough GPUs 
for the high-priority job, then the regular jobs will be preempted to allow the high-priority job to run immediately(<a href="#footnote1">*</a>).

## Influence on regular jobs

The scheduler selects jobs for preemption in a way that minimizes disruptions to ongoing regular jobs. 
When a job is preempted, it is stopped, and its allocated resources are released for other tasks.

To mitigate potential loss of progress, you can implement checkpointing to save data and program state 
in your code and trigger it when preempiton osscures. 

When you restart your program, it can resume from the last saved checkpoint instead of starting over. 
This improves efficiency by preserving computational progress and reducing redundant processing time.

For more information about how to checkpoint and restart, please check the section "[Checkpointing and Restarting](/docs/misha-checkpoint/)"

## Available group QOSs

| QOS Name         | Group Limit                   | Example                              |
| ---------------- | ----------------------------- | ------------------------------------ |
| qos_dijk         | 4 h100s, 48 CPUs, 975G memory | `--qos=qos_dijk,--gres=gpu:h100:2`   |
| qos_zhuoran_yang | 4 h100s, 48 CPUs, 975G memory | `--qos=qos_zhuoran_yang,--gres=gpu:h100:2`   |
| qos_ma_zongming  | 4 h100s, 48 CPUs, 975G memory | `--qos=qos_ma_zongming,--gres=gpu:h100:2`  |
| qos_saxena       | 20 a100s, 160 CPUs, 5000G memory | `--qos=qos_saxena,--gres=gpu:a100:2`  |
| qos_ying_rex     | 4 a40s, 32 CPUs, 975G memory | `--qos=qos_ying_rex,--gres=gpu:a40:2`  |
| qos_yildirim     | 12 a40s, 96 CPUs, 29255G memory|`--qos=qos_yildirim,--gres=gpu:a40:2`  |
| qos_lafferty     | 4 a40s, 32 CPUs, 975G memory | `--qos=qos_lafferty,--gres=gpu:a40:2`  |


The option `--gres=gpu:type:num` specifies the number of GPUs per node, while another commonly used option 
`--gpus=type:num` specifies the total number of GPUs required for the job. 
As `--gpus=type:num` doesn't care how the GPUs requested are allocated, they may end up in a distribution you don't 
want when your job has also requested multiple nodes. So it is better to use `--gres=gpu:type:num`, or `--gpus-per-node=type:num`, or `--gpus-per-task=type:num`.

## Footnotes
<div id="footnote1">
<p style="font-size:14px; ">
* <b>Note</b>: Preemption is disruptive to the jobs that are selected to free up the resources. To reduce the loss of preempted jobs, 
SLURM is configured with a one-hour grace period for preempted jobs to continue to run. Subsequently,
a priority job may have to wait for up to one hour to start. Also, if the priority job requests more othan one GPU, it may need to wait for even longer. 
</p>
</div>


