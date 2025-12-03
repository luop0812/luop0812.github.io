---
title: "Checkpointing and Restarting"
permalink: /docs/misha-checkpoint/
excerpt: "Checkpoint and restart on Misha"
last_modified_at: 2021-06-07T08:48:05-04:00
#redirect_from:
#  - /theme-setup/
toc: true
---

## Introduction

wait all

## Checkpointing a program 

We have configured a one-hour grace period in SLURM to allow a preempted job to wrap up its work. 
With the grace period, when a job is selected for preemption, it is immediately sent SIGCONT and SIGTERM signals, 
followed by the SIGCONT, SIGTERM, and SIGKILL signal sequence upon reaching its new end time(<a href="#slurm_conf">ref</a>). The first SIGTERM 
is sent to all job steps (a job step is started with <b>srun</b> in SLURM)(<a href="#footnote">*</a>). In order to catch the first SIGTERM
to do checkpointing, a user program must be launched with <b>srun</b>.

The following simple example shows how to catch the SIGTERM and call a signal handler. 

<b>simple.job</b>
```<bash>
#!/bin/bash
#SBATCH -p gpu 
#SBATCH --gres=gpu:a100:1
#SBATCH -n 1 --ntasks-per-node=1 

date
srun ./simple.py
echo "finished"
```
<b>simple.py</b>
```<phthon>
#!/usr/bin/env python3.11

import signal
import sys
import time

SIGTERM = 15
def signal_handler(sig, frame):
    print('A signal has been caught.')
    time.sleep(10)

signal.signal(SIGTERM, signal_handler)
time.sleep(600)
print('Done')
```

A more comprehensive example of checkpointing can be found at <a href="https://github.com/YaleWTI-CNMI/slurm-checkpoint-pytorch">this GitHub repository</a> of the Center for Neurocomputation and Machine Intelligenc
e of WTI.


## Requeuing

If you have checkpoint-and-restart implemented in your code, you can add the SLURM option `--requeue` to your job. This option ensures that 
when your job is preempted, it will return back to the queue automatically. 
 When the resources are available for the job again, it will restart from the last checkpoint without needing any user intervention.

## Automtic checkpoint-and-restart

We now have all the ingredients to implement automatic checkpoint-and-restart. 
Here is a complete example demonstrating checkpointing at preempiton and automatic restarting when resources become available again. 

## References

<div id="slurm_conf">
<a href="https://slurm.schedmd.com/slurm.conf.html">slurm.conf (search GraceTime)</a>
</div>

## Footnotes

<div id="footnote">
<p style="font-size:14px; ">
* <b>Note</b>: This is not clear in the SLURM documentation. We only found it out through trial and errors. 
</p>
</div>


