---
title: Frontier models guess at compute
description: We gave eight frontier models the code and submission script of 665 real HPC jobs and asked what each job would need. None of them got close, the confidence ranges they gave were made up, and Expanse was 8x more accurate than the best of them working alone.
date: 2026-06-02
author: Ismaeel Bashir
github: ismaeelbashir03
tags: [benchmark, hpc]
images:
  alt: Median error of eight frontier models and Expanse when predicting the runtime and peak memory of 665 real HPC jobs
  social: images/frontier-models-guess-at-compute/social.png
  card:
    mobile: images/frontier-models-guess-at-compute/card-mobile.png
    desktop: images/frontier-models-guess-at-compute/card-desktop.png
  header:
    mobile: images/frontier-models-guess-at-compute/header-mobile.png
    desktop: images/frontier-models-guess-at-compute/header-desktop.png
draft: false
featured: false
---

> **Update, 24 October 2026:** the latest models perform the same on this benchmark, and a new benchmark is coming soon.

> **TL;DR**: We gave eight frontier models the source code and submission script of 665 real HPC jobs and asked each one how long the job would run and how much memory it would use. The best of them was off by 77% on runtime and 58% on memory on the median, and the 10th to 90th percentile ranges they gave contained the truth 23% of the time. Expanse was within 10% and 5% on the same jobs. The models reason about the code fine. What they need is a tool built on the data.

## If the model can read the code, why not just ask it?

It's a fair question, and more teams are asking it every month. A frontier model can read a training script, reason about the kernel and give you a number. If that number was good enough, sizing a job would already be a solved problem. So we measured it properly, on real jobs, against what those jobs actually used.

## What we ran

The jobs are 665 runs of 102 programs across eight kinds of HPC workload: NPB, the CORAL/ECP proxy apps, OSU MPI, RODINIA, STREAM, BLAS, blackbox commercial solvers and a set of smaller kernels. They ran on two production clusters at a national supercomputing centre, each with several resource configurations, and we measured the runtime and peak memory of every run when it happened. Runtimes go from under a second to 8.7 hours and peak memory from 10 MB to over 200 GB.

Eight models, the frontier at the time: Claude Opus 4.7, Sonnet 4.6 and Haiku 4.5, GPT-5.5, GPT-5.5 Pro and GPT-5.3-Codex, o3 and o4-mini. Each one got the submission script with the resource lines stripped out and as much of the source as fits in a prompt, and was asked for two numbers, runtime in seconds and peak memory in GB, with a 10th to 90th percentile range on each and a confidence score. We turned reasoning on, at high effort where the model has the setting, and gave no examples and no tools, so this is what you get if you paste a job into a chat and ask.

Expanse ran on the same 665 jobs with the same five fold splits. For each held out job it could only see the jobs in the other folds as the cluster's history, so every prediction is out of sample.

We scored everything on median absolute percentage error, so a model that's wildly wrong on a few jobs doesn't hide how it does on the rest.

## What came back

| Model | Runtime, median error | Peak memory, median error |
|---|---|---|
| o3 | 77% | 92% |
| GPT-5.5 Pro | 83% | 66% |
| GPT-5.3-Codex | 85% | 58% |
| GPT-5.5 | 86% | 60% |
| o4-mini | 90% | 61% |
| Claude Sonnet 4.6 | 173% | 86% |
| Claude Haiku 4.5 | 183% | 98% |
| Claude Opus 4.7 | 228% | 92% |
| **Expanse** | **10%** | **5%** |

The best frontier model on runtime was o3, off by 77% on the median. The worst was Opus 4.7 at 228%. On memory the best was GPT-5.3-Codex at 58% and the worst Haiku 4.5 at 98%. Expanse was within 10% on runtime and 5% on memory on the same jobs, which is 8x more accurate than the best frontier model working alone.

The errors go both ways. The three Claude models asked for around 3x the runtime the job used on the median, which is the request that sits reserved and idle. o3 and o4-mini went the other way and under predicted runtime on the median, which on a real scheduler is the job that gets killed before it finishes.

Two things didn't help. Training on code didn't close the gap: GPT-5.3-Codex got 85% on runtime against 86% for plain GPT-5.5. Reasoning didn't either, with o3 at high effort the best on runtime and still 77% out.

## The ranges were made up

An honest 10th to 90th percentile range contains the truth 80% of the time. Across the eight models the runtime ranges contained it 23% of the time, and the best of them managed 30%. Memory was 27%.

| Model | Runtime in range | Memory in range | Round bounds |
|---|---|---|---|
| Claude Opus 4.7 | 29% | 43% | 99% |
| Claude Sonnet 4.6 | 29% | 35% | 100% |
| Claude Haiku 4.5 | 3% | 8% | 69% |
| GPT-5.5 | 28% | 28% | 88% |
| GPT-5.5 Pro | 30% | 31% | 85% |
| GPT-5.3-Codex | 26% | 30% | 77% |
| o3 | 28% | 18% | 99% |
| o4-mini | 12% | 27% | 99% |

The bounds themselves show where they came from. Almost every range end the models gave was a round number, one or two digits and then zeros (the last column), 45 to 260 seconds, 5 to 28 GB. That's what a number looks like when it's picked to sound plausible and not read off any data. The confidence scores barely moved with how wrong the answer was either.

## Two jobs up close

### CORAL AMG: 328x short on runtime

A frontier model got the submission script and as much of the HYPRE source as fits in a prompt (the whole thing is 91,490 lines), and was asked for the wall clock runtime. It said 95 seconds. The job took 8.7 hours. Expanse said 8.5 hours. The ranges didn't help: on that job every model's upper bound was under five minutes.

### NPB IS: 33x over on memory

Integer sort, MPI class D on 1,024 processes. The model got the source and the submission script and was asked for peak memory per node. It said 640 GB. The job used 19.5 GB. Expanse said 22.4 GB. The model had added up memory across every process instead of per node.

## Where the gap is

The models reasoned about the code fine. Read their working and they identify the solver, the problem size and the communication pattern, and they usually pick the right thing to worry about. What they can't do is turn that into a number, because the number comes from data they've never seen: how long this solver took on this cluster at this size, what the interconnect did at 1,024 processes, what jobs like this one used last month. So they do what anyone does when asked for a number they don't have. They pick a plausible one and round it.

That's a tool problem, the same one a model has with arithmetic or a database lookup, and a smarter model doesn't fix it. What fixes it is a model trained for this one task on the cluster's own data, telemetry in and a measured distribution out, that the agent calls the way it calls a compiler.

## What that's worth

Most clusters sit at around 50% utilisation. Expanse is built to take that towards 90% with no new hardware. On a 1,000 GPU cluster that's 500 GPUs reserved and idle, which is around $8.8M a year at $2 a GPU hour. At 10,000 GPUs it's $88M a year, and at 100,000 it's $880M.

## Where this goes

Frontier models are going to run more and more of the experiment loop, and I think that's the right direction. The agent writes the code, and when it needs to know what the job will take, it should ask a tool built for that question instead of guessing.

That tool is Expanse. Expanse predicts what every job needs before it runs, so nothing sits reserved and idle, and nothing dies halfway through. We call this compute certainty: knowing exactly what a workload needs before it runs. It's a two-model ensemble: precedent matching over the cluster's own history, and code reasoning over the script, dependency graph and cluster state when there is no precedent. What comes back is a full distribution over what the job will need, with an interval that comes from measurements and not from a guess, and a flag on the jobs that will fail before they start. Runs on Slurm, Kubernetes and the major clouds, on-prem or hybrid. It sits inside your own environment, so your code and your data never leave it.

If you want to run another model on the same jobs and splits, email me at [ismaeel@expanse.sh](mailto:ismaeel@expanse.sh) and I'll send you the kit. And if you run a cluster, I'm curious what the number looks like on yours: [Book time with me](https://outlook.office.com/book/ExpanseDiscovery@expanse.sh).
