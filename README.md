---
layout: lesson
root: ../..
title: Introduction to the Open Science Grid 
---
<!-- <div class="objectives" markdown="1">

#### Objectives
*   Overview of the Open Science Grid consortium and available resources.
*   Understand what computational is and isn't well-supported on the OSG.
*   Begin working with OSG software modules and tutorials in preparation for remaining exercises.

</div> -->

## The Open Science Grid

The [Open Science Grid (OSG)](http://opensciencegrid.org/) is a consortium of research communities that provides resources to use distributed [high throughput computing (HTC)](http://en.wikipedia.org/wiki/High-throughput_computing) for scientific research. The OSG:

* Enables distributed computing on backfill capacity at more than 120 institutions.
* Delivers nearly 2 million core CPU hours per day, including ~500,000 'opportunistic' hours available to campuses and individual research groups (though this varies significantly from day-to-day, and there's even more un-tapped capacity available).
* Allows campuses to join the consortium by (1) contributing their own backfill capacity (from just about any 'type' of Linux cluster), and/or (2) establishing a local submission point for local researchers to access OSG.
* Provides the OSG Connect service for individual research groups who do not have a local OSG submission point.
* Supports a variety of common research applications in a core OSG-maintained repository (called OASIS).
* Supports several options for working with large, per-job input and output.
* Leverages the HTC-optimized [HTCondor compute scheduling software](https://research.cs.wisc.edu/htcondor/) for job queueing and execution (on top of a layer of OSG-specific softwae and tools).

![fig 1](https://raw.githubusercontent.com/SWC-OSG-Workshop/OSG-UserTraining-RMACC17/gh-pages/novice/DHTC/Images/osg_job_flow.png)

### Computation that is a good match for OSG 

Computational workloads that are accelerated with an HTC execution approach are those that can parallelize naturally to numerous, completely independent 'jobs', and this turns out to be the case for VERY many research workloads, especially those typically defined as 'big data'. For example, parameter space optimizations, image processing, various forms of text processing (including much of computational genomics and bioinformatics), and sets of numerous, relatively-small simulations can all be executed as numerous, independent jobs whose results can be later combined or analyzed. Nearly every time a researcher has written their own code with an internal 'mapping' or 'loop' step that is the very time-intensive, they will see far greater throughput by using an HTC execution approach and be able to expand the 'space' of their computational work to tackle greater research challenges.

**The *best* HTC workloads for OSG** can be executed as many jobs that EACH complete on 1 CPU core in less than 12 hours, with less than 100 MB of input or output data. The HTCondor queueing system can transfer all executables, input data, and output data (up to a certain size) from the submit server to the job, as specified in job submit files created by the user. However, the technologies in OSG also support jobs needing multiple cores, GPUs, and larger per-job data. Generally, the 'less' each job needs, the more jobs a user will have running, and sooner. Jobs requesting 1 CPU core and less than 1 GB of memory will typically get hundreds or up to thousands of cores running their jobs in the OSG, and reach that maximum within several hours of initially submitting the jobs.

Another component of 'fit' for jobs running on OSG pertains to the necessary software and computing environment. Jobs submitted to the OSG will be executed at several remote clusters (OSG is a *distributed* HTC, or 'dHTC' environment). The execute servers at each of these will differ a bit from the computing environment on the submit server, but all of OSG execute servers currently run RedHat-compatible Linux of major version 6 (decreasingly) or 7. This heterogeneity means that jobs will run most consistently if they bring along the major portions of the software environment they'll need (or leverage OSG-provided software for common applications). Jobs that leverage statically-compiled binaries (which includes many genomics applications, for example) are the most robust across the environment heterogeneity in OSG, but there are many users whose jobs bring along a pre-compiled version of Python or R, or leverage Matlab's runtime interpreter for compiled Matlab code, as just a few examples.

**Consider the following dHTC guidelines:**

* Jobs should run single-threaded, individually using less than 2 GB memory and running for 1-12 hours. There is support for jobs with longer run time, more memory or multi-threading support (just fewer 'slots' regularly becoming available for them, and jobs can be evicted at any time).
* Only core utilities can be expected on the remote end. There is no standard version of software even for `gcc`, `python`, `BLAS`, or others. Consider using OSG-supported Software Modules, see below, to manage software dependencies, look through our [High Throughput Computing Recipes](https://support.opensciencegrid.org/support/solutions/5000161171), or get in touch with the OSG Connect team to discuss the best approach for 'packing up' your software dependencies.
* Input and output data for each job should be < 10 GB to allow them to be transferred in by the jobs, processed and returned to the submit node. The scheduler, HTCondor, can transfer input and output files up to 100 MB (~500 MB total per job), and other transfer methods (`scp`, `rsync`, `GridFTP`) are better suited for larger per-job data transfers. Please contact the user support listed below for more information

### Computation that is NOT a good match for OSG 

The following are examples of computations that are NOT good matches for 
OSG:

* Tightly coupled computations, for example using MPI-based communication across multiple compute nodes (due to the distributed nature of the OSG infrastructure).
* Computations requiring a shared file system, as there is no shared file system between the different clusters on the OSG.
* Computations requiring complex software deployments or proprietary software are not a good fit. There is limited support for distributing software to the compute clusters, and for complex software, or licensed software, deployment can be a nearly impossible task.

### How to get help using or joining OSG

Please contact user support staff at [user-support@opensciencegrid.org](mailto:user-support@opensciencegrid.org).


