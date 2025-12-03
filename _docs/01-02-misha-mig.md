---
title: "MIG and gpu_devel"
permalink: /docs/misha-mig/
excerpt: "Multi-Instance GPU on Misha"
last_modified_at: 2021-06-07T08:48:05-04:00
#redirect_from:
#  - /theme-setup/
toc: true
---

## gpu_devel

The partition `gpu_devel` is designed for GPU code development, including tasks such as developing, testing, and debugging code. To ensure instant response time, It has relatively low per-user limits, as outlined in the table below:

| name             | limit                       |
| ---------------- | --------------------------- |
| walltime         | 6 hours                     |
| CPU core         | 4                           |
| memory           | 32 GiB                      |
| GPU              | 2                           |


## MIG

MIG (Multi-Instance GPU) is a technology that partitions a single physical GPU into multiple virtual instances, each with a smaller share of processing power and VRAM. These virtual GPUs, known as <b>MIG instances</b>, enable multiple tasks to run simultaneously on the same GPU, improving resource utilization while ensuring isolation and efficiency.

Misha's `gpu_devel` partition includes one A100 node with four A100 cards. Two types of MIG instances are configured for those A100 cards:

* <b>a100.MIG.10gb</b> – 10GB VRAM
* <b>a100.MIG.20gb</b> – 20GB VRAM, approximately twice as fast as the a100.MIG.10gb instance

Each A100 GPU is partitioned into two a100.MIG.20gb instances and three a100.MIG.10gb instances, resulting in a total of 20 MIG instances in the `gpu_devel` partition.

Requesting a <b>MIG instance</b> is similar to requesting a standard GPU - you need to specify both the <b>partition name</b> and the <b>GPU type</b>. For example, to request one a100.MIG.20gb instance, use the following SLURM options:


```<bash>
 -p gpu_devel --gres=gpu:a100.MIG.20gb:1
```

## Open OnDemand

In Open OnDemand, select `gpu_devel` for partition name, and then select a GPU type or leave it empty. Also, provide the number of GPUs (the default is 1). Usually, one is enough. Please don’t request more than you need. 
