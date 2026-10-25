---
title: Built and measured on a national supercomputer
description: How an early version of Expanse's prediction model was built and measured against ground truth on production clusters at a national supercomputing centre, and what a month on a national facility showed about how much compute is paid for and never used.
date: 2026-10-27
author: Ismaeel Bashir
github: ismaeelbashir03
tags: [research, hpc]
images:
  alt: Ranking accuracy and median runtime error of an early Expanse model against conventional baselines on a month of real HPC workloads
  social: images/measured-on-a-national-supercomputer/social.png
  card:
    mobile: images/measured-on-a-national-supercomputer/card-mobile.png
    desktop: images/measured-on-a-national-supercomputer/card-desktop.png
  header:
    mobile: images/measured-on-a-national-supercomputer/header-mobile.png
    desktop: images/measured-on-a-national-supercomputer/header-desktop.png
draft: false
featured: false
---

> **TL;DR**: An early version of Expanse's prediction model was trained and evaluated on two production clusters at a national supercomputing centre, on a month of real workloads with measured runtime and peak memory. Against that ground truth it ranks jobs by runtime 32% better than the best conventional model and by peak memory 55% better, half of its runtime predictions land within 30.6% of the truth, and its prediction intervals contain the measured value for 86% of jobs. Running across one national facility for a month, in observe only mode, it identified nearly $8M of compute that was paid for and never used. An extension of the model with inputs from the cluster itself got within 10% on runtime and 5% on memory on the same jobs, and the version live now is a later iteration again.

Expanse predicts what every job needs before it runs, so nothing sits reserved and idle, and nothing dies halfway through. We call this compute certainty: knowing exactly what a workload needs before it runs. This post covers an early version of the model behind it, how it was built and how it was measured.

## Every job starts with a guess

Batch schedulers ask you to declare runtime, memory and cores before a job runs. Memory and cores can't be changed once it starts, so a job that asks for too little is killed. The rational response is to ask for too much, and that's what everyone does.

People have been measuring this for 25 years. [Mu'alem and Feitelson](https://doi.org/10.1109/71.932708) found users overestimate runtime by around 2x, with memory estimates no better, and the [Parallel Workloads Archive](https://www.cs.huji.ac.il/labs/parallel/workload/) records the same pattern on HPC systems worldwide. On the Blue Gene/Q Mira supercomputer, failed jobs made up over 40% of core hours during the worst periods, and timeouts alone were a third of the waste. Overestimating also defeats backfilling, so the queue gets worse for everyone.

## What goes in

Two things go into a prediction. The first is the code. We compile it with a compiler we built for HPC workloads, and what comes out is an input to the model, alongside the submission script and the job's metadata. It works on code the model has never seen, including code it can't read.

The second is the cluster. The hardware goes in, and so does live telemetry on how jobs like this one behaved on it last month. The early model in this post used the code, the script and the metadata. The extension added the cluster.

The output is a distribution with an interval. If the model says 10 GB and is wrong, you stop trusting it. If it says 10 GB with an interval, you decide how much headroom you want.

## Evaluation on a national supercomputer

The early model was evaluated against measured runtime and peak memory on a month of workloads that ran on two production clusters at a national supercomputing centre. That's 102 programs across eight kinds of workload: NPB, the CORAL/ECP proxy apps, OSU MPI, RODINIA, STREAM, BLAS, blackbox commercial solvers and a set of smaller kernels. Some are whitebox with the full source and some are blackbox binaries with none. Runtimes go from under a second to 8.7 hours and peak memory from 10 MB to over 200 GB.

Everything is five fold cross validated. The model never scores a job it was trained on, so every prediction below is out of sample. The baselines are XGBoost and MLP models on the job metadata, with and without features from the code and the text of the submission script, trained and scored the same way.

| What we measured | Early model | Best conventional baseline |
|---|---|---|
| Runtime, ranking jobs by what they'll use (Kendall's tau) | 0.815 | 0.617 |
| Runtime, median error | 30.6% | 46.7% |
| Runtime, variation explained (R²) | 0.874 | 0.531 |
| Peak memory, ranking (Kendall's tau) | 0.802 | 0.519 |
| Peak memory, median error | 34.7% | 74.5% |

Ranking is the number a scheduler cares about. If the model puts jobs in the right order by what they'll really consume, the scheduler can pack the queue tightly, and 0.815 against 0.617 on runtime is a 32% improvement on the previous state of the art. On peak memory the gap is wider, 0.802 against 0.519, a 55% improvement.

On runtime, half of all predictions land within 30.6% of what the job really took. The norm for a human estimate is overestimating by 2x, so that's around three times tighter than the job would've been sized by hand. And because the model explains 87% of the variation in what jobs consume, the accuracy holds across the whole range of jobs.

The model also comes with a distributional head that gives an interval instead of a single number. Its 10th to 90th percentile range contains the measured runtime for 86% of jobs and the measured peak memory for 83%, against the 80% an honest range should give, so the intervals are slightly cautious. The interval is what an operator uses to pick headroom.

## Against frontier language models

We also gave eight frontier models the same jobs, zero shot, with the source code and the submission script in the prompt. The best of them was off by 77% on runtime and 58% on peak memory on the median, their 10th to 90th percentile ranges contained the measured value for 23% of jobs, and the early model beat every one of them on every number. The extension, with the cluster's own history, hardware and telemetry as inputs, got within 10% on runtime and 5% on memory on the same jobs. The full benchmark is in [Frontier models guess at compute](/blog/frontier-models-guess-at-compute).

## A month on a national facility

Running across one national supercomputing facility for a month, on a well run system, Expanse found that most of the compute being paid for was never used: nearly $8M of capacity in a single month. That is capacity identified in observe only mode.

Padding the request is the rational response to being asked to guess, and it shows up on well run systems, which is exactly why it goes unnoticed. None of that capacity needs anything bought, and no user has to change how they work.

## Conclusion

Monitoring tools chart usage after the fact. Expanse works before the job starts. Runs on Slurm, Kubernetes and the major clouds, on-prem or hybrid. It sits inside your own environment, so your code and your data never leave it.

If you run a cluster, I'm curious what the number looks like on yours: [Book time with me](https://outlook.office.com/book/ExpanseDiscovery@expanse.sh).

## References

1. A. W. Mu'alem and D. G. Feitelson, "Utilization, predictability, workloads, and user runtime estimates in scheduling the IBM SP2 with backfilling", IEEE Transactions on Parallel and Distributed Systems, 12(6), 2001.
2. D. G. Feitelson, D. Tsafrir and D. Krakov, "Experience with using the Parallel Workloads Archive", Journal of Parallel and Distributed Computing, 74(10), 2014.
3. Z. Zheng, Z. Lan, R. Gupta, S. Coghlan and P. Beckman, "Characterizing and understanding HPC job failures over the Blue Gene/Q Mira", IEEE/IFIP DSN, 2018.
