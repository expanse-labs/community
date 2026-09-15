---
title: It's 2026 and We're Still Writing SLURM Scripts by Hand
description: Why the friction between scientific intent and cluster execution remains one of the most solvable hard problems in research computing.
date: 2026-02-28
author: Ismaeel Bashir
tags: [hpc, slurm]
image: https://raw.githubusercontent.com/expanse-labs/community/main/blog/images/slurm-scripts-by-hand/header-desktop.jpg
imageAlt: A Slurm batch script with its resource requests marked as guesses
draft: false
featured: true
---

> **TL;DR**: I've personally worked within HPC teams in academia and industry and found surprisingly that we are spending more time fighting infrastructure or doing 'meta work' than doing actual research. I think it's the most solvable hard problem in science right now.

The biggest breakthroughs in modern day science: protein folding, climate modelling, drug discovery, fusion simulations, all run on massive on-premises supercomputers and data centres. Although these ideas are human, the execution is computational, and as the problems get bigger, the compute required to solve them gets bigger too. [HPC is growing 12% year-on-year](https://www.precedenceresearch.com/high-performance-computing-market) because we figured out that large-scale compute is the lever that brings us into the new age of scientific breakthroughs, from [AlphaFold's protein structure predictions](https://www.nature.com/articles/s41586-021-03819-2) to [fusion energy simulations at ITER](https://www.iter.org/sci/Goals) to [the climate models underpinning global weather forecasting](https://www.ecmwf.int/en/about/media-centre/science-blog/2022/how-we-keep-pushing-boundaries-weather-prediction).

But there is a problem that I've seen, and I believe it is *the* biggest bottleneck in scientific progress of our time: the friction between user intent and execution. After spending time working with researchers I saw that the biggest mental drain for them is going from working code to getting it onto a cluster and submitted, because you are always fighting bugs, digging through logs, transferring files back and forth, wrestling with config files. Why would anyone doing scientific research want to deal with any of these?

I've always wanted to close the gap between my ideas and actually executing them. When I first got access to [ARCHER2](https://www.archer2.ac.uk/) during my time at [EPCC](https://www.epcc.ed.ac.uk/), I spent the first two days not running anything, just configuring environments, fighting module conflicts, writing submission scripts. *Two days before a single line of my actual work ran.* That stuck with me.

One thing I've realised working in HPC is that the infrastructure has no intelligence. It doesn't know your code, it doesn't know your history, and it certainly doesn't try to help you. You are entirely on your own, and when something goes wrong at 3am after a 4-week queue wait, it feels pretty lonely. Imagine a world where you can run your code on a supercomputer as easily as running it locally. That's what I want to build. That's what Expanse is.

The thing is, HPC is stuck in the 80s, and I think we are in the perfect conditions for a change. [Moore's Law](https://en.wikipedia.org/wiki/Moore%27s_law) has levelled off, which means the next gains come from software intelligence, not faster chips. Funding into supercomputing is at an all-time high, so any improvement compounds. And with capable AI models now available, there's finally a real opportunity to automate the admin work that's been eating researcher time for decades. *The hardware got a revolution. The tooling never did.*

After conversations with various people running [HPC](https://en.wikipedia.org/wiki/High-performance_computing) workloads across enterprise and national labs, we saw that it takes a huge amount of time to run things on a supercomputer. Look at this video below: submitting a single workflow on ARCHER2 takes roughly 40 minutes of hands-on human time, even when using ChatGPT to guide you along the way. This is not queue wait or waiting for the workload to complete. It's time wasted by someone who should be pushing the boundaries of their field sitting at a terminal, writing [SLURM](https://slurm.schedmd.com/overview.html) scripts, configuring resource requests, managing dependencies, transferring data, and hoping every flag is set correctly.

https://www.youtube.com/watch?v=6H9P1q7cvzQ

Instead of trying to replace the entire stack, with Expanse, we wanted to build a tool that sits cleanly between the cluster and your code. Your workload definition should be the only thing you think about. Everything else, which cluster to target, how to size resources, when to submit, what to do when something goes wrong, should be handled for you, transparently and without you having to ask.

One command. That's it.

No SLURM script to rewrite. No SSH session to open. No modules to fight with. And the researchers we've spoken to aren't junior people who don't know their way around a cluster; these are people who have spent years on these systems. They're still losing 40 minutes per submission, every time, because *that's just how long it takes.* That's not a skill gap. That's a tooling problem.

What gets me is that this is everywhere. Every lab, every research group, every quant team with an on-premises cluster. The same 40 minutes, repeated thousands of times a day, across every field that depends on large-scale compute. That's not a niche problem. That's an enormous amount of collective human time going nowhere.

And the failures are worse. When an [OOM kill](https://en.wikipedia.org/wiki/Out_of_memory) hits after a three-week queue wait on a four-week job, you don't just lose compute time. You lose the researcher's momentum, their trust in the system, and sometimes weeks of uncheckpointed progress. That happens because nobody told the job it was going to run out of memory. *We knew before it even started.* We just had no tools to act on it.

We're not trying to replace the scheduler or reinvent the cluster. The hardware is fine. What's missing is an intelligent layer between you and it: something that reads your code, knows your history, predicts what your job actually needs before it runs, and takes care of the rest. Not as a thin wrapper. As something that earns your trust over time, job by job.

That's what we're building with Expanse. And we're just getting started.

If you're running HPC workloads and any of this sounds familiar, I'd like to hear about it. Reach me at [ismaeel@expanse.org.uk](mailto:ismaeel@expanse.org.uk), or book a call at [expanse.org.uk](https://expanse.org.uk).
