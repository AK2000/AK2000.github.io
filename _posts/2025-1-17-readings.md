---
layout: post
title: What I've Been Reading (1/17/2025)
date: 2025-1-17 17:30:00
description: A bi-weekly recap of the papers that I've been reading
tags: readings
categories: professional
---

### Architectural Implications of Function-as-a-Service Computing

Mohammad Shahrad, Jonathan Balkind, David Wentzlaff | Micro 2019 | 1/7/2025

---
***Summary:*** Serverless computing provides a popular paradigm where developers only pay for resources they use, and providers manage provisioning and autoscaling. This paradigm creates a new kind of workload for the cloud with (1) very short functions, (2) fine-grained billing based on GB-s, (3) fine-grained inter leaving. This workload challenges some of the assumptions of OS and server architecture. They find:
  1. Cold start costs are non-negligible and require increased memory-bandwidth, a well known phenomenon for which much research has been done since
  2. Branch predictors perform badly for very short functions, causing high branch misprediction
  3. Function execution time can be on the order of OS scheduling quantum
  4. Without SLAs on latency, the provider has incentive to over sell capcity, creating slow-downs for the user, and making the provider more money.
  5. Function interfence can cause increase in page faults, context switches and LLC misses.

***Thoughts:*** Many of the things recongized in this paper see obvious, sepcifically cold starts, function interference, and the interplacy between capacity and cost. The increase in branch prediction misses is notable, more so because I thought that these kind of issues would be the same issues faced by say oversubscribing cores for multi-processing. The issue of OS scheduling for short functions is also conerning and relevant. This again plays into costs and incentives. Users are charged for the runtime of their function, which is dependent on the CPU slots allocated to them. If the OS scheduling overhead of Linux CFS is on the same order of magnitude as the function execution, users could be paying significantly more because of the scheduler configuration. This problem is addresed in a new paper: *In Serverless, OS Scheduler Choice Costs Money: A Hybrid Scheduling Approach for Cheaper FaaS* which may be a paper for a different time.

***
***

### Will Serverless End The Dominance of Linux in the Cloud

Richard Koller, Dan Williams | HotOS 17 | 1/9/2025

---

***Summary:*** Again on the serverless theme. The authors make a distinction in the requirements for "actions" which are the unit of serverless computing, and processes which are the unit the linux kernel was designed for. They lay out latency and throughput requirements for serverless based on AWS lambda cost: an action must be able to complete in 100ms, and a 4 core server must be able to support 125 actions per second. They compare different solutions fer serverless architectures: a native linux process, a container, and a unikernel. The authors show that a container takes almost 6 times longer to boot than a process, but a unikernel takes only 34\% more time. Furthermore, they find the max throughput for a container to be 24 tasks/sec, compared to 329 for a process. As possible solutions they propose we could (1) bypass the kernel, as is done in a unikernel, or (2) replace the kernel with something that provides less features, no preemptable scheduling, no IPC, and no process sychronization features.

***Thoughts:*** Its interesting to see how serverless can shape and change the direction of the OS, but my thought is that the changes proposed here are too radical. The authors undervalue the history and extensive community that linux comes with. These things provide confidence that the code in the kernel is well-tested, secure, and will be maintained over a long term. I also think that we have seen other, less radical solutions to the cold-start problem in serverless. The authors dismiss caching as a inelegant solution, but this has been used with great effectiveness. Furthermore, I am interested to know what portion of these problems are addressed with a MicroVM like firecracker.

***
***

### Unikernels: Library Operating Systems for the Clouds
Anil Madhavapeddy, Richard Mortier, Charalampos Rotsos, David Scott, Balraj Singh, Thomas Gazagnaire, Steven Smith, Steven Hand and Jon Crowcroft | ASPLOS 2013 | 1/13/25

---

***Summary:*** Cloud appliances often sepearte different components of an application into sepearte VMs (a database, a webserver, etc.). The authors propose unikernels, single application virtual machine where operating system functionality is linked directly to the application.
This ahs advantages for size, and for safety since the application can be statically type checked, privleges can be dropped from the VM to prevent executable pages and the VM does not need process level permission management. The paper containes details of the simplifications that a unikernel allows to the memory management and garbage collector of the OCaml runtime as well as details for how drivers and block devices work for the unikernel.

***Thoughts:*** The idea here is very simple, but required a lot of engineering effort to bring to fruition. Since this paper, unikernels have become an influential idea in systems, although they still remain difficult to use in practice. I still don't understand how certain parts of the unikernel/OS work. What does it mean to implement the scheduler as a library function? Why exactly were the unikernels in this paper restricted to a single heap?

***
***
### With Great Freeedom Comes Great Opportunity: Rethinking Resource Allocation for Serverless Functions
Muhammad Bilal, Marco Canini, Rodrigo Fonseca, Rodrigo Rodrigues | EuroSys 23 | 1/15/25

---

***Summary:*** In this paper, the authors examine the resource allocation model provided by current serverless platforms, which couples CPU resources to amount of allocated memory, and typically does not provide the oportunity to configure CPU types. The authors project the cost model from current offerings onto heterogeneous offerings, both in terms of instance type, and in terms of CPU vs. memory tradeoff. They show that by decoupling memory and providing different CPU options, functions can be executed cheaper and faster. They then move to the feasibility of predicting the "right" resource allocation for a serverless function using saympling, and in an online setting using bayseian optimization. They also find that the input data only has a modest affect on the best configuration for a function. Finally they consider the resrouce allocation problem from the providers perspective. They provide three different options on how to make different configurations available to the user: give users options on the predicted pareto front, provide an optimization with cost and execution time weighted differently, and provide a hierarchical optimization that minimizes cost under a constaint of execuion time.

***Thoughts:*** This is an interesting and well-done paper, but I believe it misses one of the key reasons that serverless is feasible from a providers perspective. It's unsuprising that there are better configurations that those provided by the default memory/CPU proportional design; I don't think anyone expected a static proportion of CPU to memory would be optimal for all functions. However, such a static tradeoff drastically simplifies the placement of serverless functions. Rather than having a multi-attribute bin-packing problem when shcuedling functions, the problem is reduced to fitting functions into static "slots". This can easily ensure that all space on a server is sold, both in terms of memory and CPU. From a providers perspective, there is not really an incentive to optimize this space more, because it complicates scheduling and could lead to stranded resources -- a persistent issue when allocating traditional cloud instances. The most interesting part of the paper to revisit is the bayesian optimization, and the online optimization sections, which could be relevant for task-based runtimes, or FuncX, a serverless system that doesn't deal with "selling" time.
