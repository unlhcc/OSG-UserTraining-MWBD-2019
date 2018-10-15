---
layout: lesson
root: ../..
title: Job Scheduling with HTCondor
---
<!-- <div class="objectives" markdown="1">

#### Objectives
*   Learn how to submit HTCondor jobs.
*   Learn how to monitor the running jobs.
</div> -->

## Overview

In this section, we will learn the basics of submitting and monitoring 
computing "jobs" with HTCondor, which is extremely similar to job 
submission using other queueing systems, but differs in some ways that 
make HTC a lot easier. For a great introduction to HTCondor's features
for users, see the (HTCondor Week User Tutorial)[https://agenda.hep.wisc.edu/event/1201/session/4/contribution/5/material/slides/1.pdf] 
(full documentation is available in the (HTCondor Manual)[http://research.cs.wisc.edu/htcondor/manual/current/2_Users_Manual.html]).


The typical cycle for a set of HTC jobs is:

1. Job(s) are submitted to the queue on the submit server.
2. A job is matched to an execute server and user-indicated files transfered to a temporary job working directory.
3. The job executable is executed in the job working directory on the execute server (executable needs to create output files in the 'current' or 'present' directory).
4. HTCondor captures any executable errors and terminal output/error information, and transfers all newly-created files back to the submission directory on the submit server.

In HTCondor the job submit file (separate from the job executable) communicates how many jobs to submit, what data to transfer to the job, and the resource requirements of the job(s).

<!-- ![fig 1](https://raw.githubusercontent.com/OSGConnect/tutorial-quickstart/master/Images/jobSubmit.png) -->

## Submit a 'Dummy' Job to learn HTCondor Submission

### Job Execution Script

We will retrieve data for some examples from the `tutorial` options on the OSG Connect Server. Let's get started with the `quickstart` tutorial, which will download a directory of job setup files named `tutorial-quickstart`:

    $ tutorial quickstart
    Installing quickstart (master)...
    Tutorial files installed in ./tutorial-quickstart.
    Running setup in ./tutorial-quickstart...
    $ cd tutorial-quickstart

We will look at two files in detail: `short.sh` and `tutorial01.submit`

Inside the tutorial directory, look at `short.sh` using

    $ cat short.sh

This is an ordinary shell script that will print various information about the server it runs on, but isn't really going to do any science or produce any specific output files:

    #!/bin/bash
    # short.sh: a short discovery job
    printf "Start time: "; /bin/date
    printf "Job is running on node: "; /bin/hostname
    printf "Job running as user: "; /usr/bin/id
    printf "Job is running in directory: "; /bin/pwd
    echo
    echo "Working hard..."
    sleep 20
    echo "Science complete!"

Now, add executable (x) permissions to this script, so that it can be run as an executable on Linux computers:

    $ chmod +x short.sh

Making the script executable and the "shebang" line (`#!/bin/bash`) line at the top of the script are not necessary for programs that are only run locally. However, _it is extremely important for jobs running on the various servers across the OSG_.

Since this is a simple script, we can test it on the submit server. (It's important to test 'real' job executables with one or a few submitted jobs, perhaps using an interactive job submission or on another Linux server; many real programs can overuse the resources on the submit server if tested there.)

    $ ./short.sh
    Start time: Mon Aug  6 00:08:06 CST 2018
    Job is running on node: training.osgconnect.net
    Job running as user: uid=46628(username) gid=46628(username) groups=46628(username),400(condor),401(ciconnect-staff),1000(users)
    Job is running in directory: /tmp/Test/tutorial-quickstart
    Working hard...
    Science complete!

### Job Description File

Next we will create a simple (though verbose with comments) HTCondor submit file.  A submit file tells the HTCondor _how_ to run your workload, with what properties and arguments, and, optionally, how to return output to the submit host.

    $ cat tutorial01.submit
    # Our executable is the main program or script that we've created
    # to do the 'work' of a single job.
    executable = short.sh

    # We need to name the files that HTCondor should create to save the
    #  terminal output (stdout) and error (stderr) created by our job.
    #  Similarly, we need to name the log file where HTCondor will save
    #  information about job execution steps.
    #  We'll specify unique filenames for each job by using
    #  the job's 'cluster' value.
    error = short.$(Cluster).error
    output = short.$(Cluster).output
    log = short.$(Cluster).log

    # We need to request the resources that this job will need:
    request_cpus = 1
    request_memory = 1 MB
    request_disk = 1 MB

    # The last line of a submit file indicates how many jobs of the above
    #  description should be queued. We'll start with one job.
    queue 1

## Job Submission

Submit the job using `condor_submit`.

    $ condor_submit tutorial01.submit
    Submitting job(s).
    1 job(s) submitted to cluster 1144.
    
The 'cluster' number is how HTCondor (and you) will identify this set of submitted jobs (one job for this first example).


### Job Status

The `condor_q` command tells the status of not-yet-completed jobs. We have submitted one job to the queue:

    $ condor_q

    -- Schedd: training.osgconnect.net : <192.170.227.119:9618?... @ 06/30/17 15:17:51
    OWNER    BATCH_NAME       SUBMITTED   DONE   RUN    IDLE  TOTAL JOB_IDS
    username CMD: short.sh   6/30 15:17      _      _      1      1 1144.0

    1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended 

The default `condor_q` output format "batches" similar jobs together, which will make it easier to manage multiple batches of *MANY* HTC jobs. If you want to see more details about each individual job, use the `-nobatch` option:

    $ condor_q -nobatch

    -- Schedd: training.osgconnect.net : <192.170.227.119:9618?... @ 06/30/17 15:19:21
     ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD
    1144.0   username         6/30 15:19   0+00:00:00 I  0    0.0 short.sh

    1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended 

If you want to see jobs submitted by all users on the submit server, use `condor_q -all`. (There are a number of other options to condor_q, only some of which we'll cover.)

You can also get status on a specific job `cluster`(which includes all jobs from the submission of a single submit file), even for a different user:

    $ condor_q -nobatch 1082 

    -- Schedd: training.osgconnect.net : <192.170.227.119:9419?...
     ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
    1144.0   username       3/6  00:17   0+00:00:00 I  0   0.0  short.sh

    1 jobs; 0 completed, 0 removed, 1 idle, 0 running, 0 held, 0 suspended

Note the ST (state) column. Your job will be in the `I` state (idle) if it hasn't started yet. If it's running, it will have state `R` (running). If it has completed already, it will not appear in `condor_q`.

Let's wait for our job to finish, at which point it will be no longer visible in the `condor_q` output. A useful tool for this is `watch`, which runs a program repeatedly, letting you see how the output changes. Let's submit the job again, and `watch` the condor_q output at two-second intervals:

    $ condor_submit tutorial01.submit
    Submitting job(s). 
    1 job(s) submitted to cluster 1145
    $ watch -n2 condor_q $USER 


When your job has completed, it will disappear from the list.  To close `watch`, hold down Ctrl and press C.

### Job Output

Once your job has finished, you can look at the files that HTCondor has returned to the working directory. If everything was successful, it should have returned:

* `short.1144.output`: An output file for each job's terminal-printed output ("standard output")
* `short.1144.error`: An error file for any error messages ("standard error")
* `short.1144.log`: A log file for each job's HTCondor logging information

Read the output file, which should show something like the below:

    $ cat short.1144.output
    Start time: Wed Mar  8 18:16:04 CST 2017
    Job is running on node: cmswn2300.fnal.gov
    Job running as user: uid=12740(osg) gid=9652(osg) groups=9652(osg)
    Job is running in directory: /storage/local/data1/condor/execute/dir_2031614/glide_6B4s2O/execute/dir_887949
    Working hard...
    Science complete!


### Job History

You can also get information about completed jobs; just use the `condor_history` command:

    $ condor_history 1144
    ID     OWNER          SUBMITTED   RUN_TIME     ST COMPLETED   CMD            
    1144.0   username       3/6  00:17   0+00:00:27 C   3/6  00:28 /share/training/..

You can see much more information about your job's final status using the `-long` option. To close the work of `condor_history` (as it may keep traversing the very long history of jobs on this submit server), use Ctrl-C, as you did for the `watch` command.


### Removing Jobs

You may want to remove individual or all your workloads from the queue. The command to remove jobs from the queue is `condor_rm`. It takes one or more arguments corresponding to job cluster numbers, job ID numbers, or your username (to remove ALL of *your* jobs). Providing the job ID will remove only that job:

    $ condor_submit tutorial01.submit
    Submitting job(s).
    1 job(s) submitted to cluster 1145 
    $ condor_rm 1145
    Cluster 1145 has been marked for removal.

while providing your username will remove all jobs associated with your username:

    $ condor_rm username
    All jobs of user "username" have been marked for removal

## Basics of HTCondor Matchmaking

As you have seen in the previous example, HTCondor is a batch management system that handles running jobs on a cluster. Like other full-featured batch systems, HTCondor provides a job queuing mechanism, scheduling policy, priority scheme, resource monitoring, and resource management. HTCondor places submitted jobs into a queue, chooses when and where to run the jobs based upon a policy, carefully monitors the their progress, and ultimately informs the user upon completion or failure. This lesson will go over some of the specifics of how HTCondor selects where to run a particular job.  

HTCondor selects nodes on which to run particular jobs using a matchmaking process.  When a job is submitted to HTCondor, HTCondor generates a set of attributes that the job needs in order to run. These attributes function like classified ads in the newspaper and are called "classads", indicating aspects of the job, including what it `wants`.  For example, we could ask `condor_q` for the full classad of a single job with the `-l` option (for the *long* list job attributes):

    $ condor_q -l 1082.0

It's a long list, but there are a number of useful lines you can find there. For example, look for lines beginning with "Req".

Similarly, HTCondor expresses aspects of matchable machine 'slots' and their requirements in the *machine* classads. Let's examine what a slot classad looks like. First we get a name for one of the slots with `condor_status` (like `condor_q` but for listing available machines), and then we ask `condor_status` to give us the details for that machine (with `-l`).

    $ condor_status 
    
Then select a slot name from the left column, and examine the `condor_status -l` output:
    
    $ condor_status -l slot1@scott.grid.uchicago.edu
    [...]
    HAS_FILE_usr_lib64_libgfortran_so_3 = true
    OSGVO_OS_STRING = "RHEL 7"
    [...]


HTCondor takes a list of classads from jobs and from compute nodes and then tries to make the classads with job requirements with the classads with compute node capabilities. When the two match, HTCondor will run the job on the compute node.

## A More Typical OSG Job

You can make use any of these attributes in your submit file "Requirements" line to limit where your jobs go, but know that doing so will mean that your jobs match to fewer 'slots', so you'll have less throughput across a batch of many jobs. The `osg-template-job.submit` file is an example of a fairly complete OSG job which you can use as a template submit script when getting started on OSG. Note the `Requirements` and `requests_*` lines.

For any script or job you want to run, you will usually want to do one or several of the following things: pass input parameters to a script, use input file, and produce an output file. Open the example script `short_with_input_output_transfer.sh` with `cat`:

    $ cat short_transfer.sh

This is a shell script that is similar to the above example. The main difference is that this script takes a text file as a command line argument argument, i.e. $1, and produces an output file that is the copy of the input file, i.e. cat $1 > output.txt.

    #!/bin/bash
    # short.sh: a short discovery job
    printf "Start time: "; /bin/date
    printf "Job is running on node: "; /bin/hostname
    printf "Job running as user: "; /usr/bin/id
    printf "Job is running in directory: "; /bin/pwd
    printf "The command line argument is: "; $1
    printf "Contents of $1 is "; cat $1
    cat $1 > output.txt
    printf "Working hard..."
    ls -l $PWD
    sleep 20
    echo "Science complete!"

A to run the `short_with_input_output_transfer.sh` script will need to transfer the input file to the remote worker node, pass the input filename as a command line argument, and then transfer the output file back. We do all these things in the example submit file `tutorial02.submit`:

    # We need the job to run our executable script, with the
    #  input.txt filename as an argument, and to transfer the
    #  relevant input and output files:
    executable = short_transfer.sh
    arguments = input.txt
    transfer_input_files = input.txt
    transfer_output_files = output.txt

    # This time, let's specify unique filenames for each job by using
    #  the job 'cluster' and 'process' values.
    error = job.$(Cluster).$(Process).error
    output = job.$(Cluster).$(Process).output
    log = job.$(Cluster).$(Process).log

    # The below are good base requirements for first testing jobs on OSG, 
    #  if you don't have a good idea of memory and disk usage. Though we 
    #  don't actually need them for our shell script, we'll also require
    #  servers that have the OSG software modules:
    request_cpus = 1
    request_memory = 1 GB
    request_disk = 1 GB

    # Let's queue one job with the above:
    queue 1

You can test this job by submitting and monitoring it as we did for prior jobs:

    $ condor_submit tutorial02.submit
    Submitting job(s).
    1 job(s) submitted to cluster 1152

The files for the terminal error and output have filenames including the job cluster and process numbers (which make up the unique Job ID number of each job), so that each job's files are unique from other jobs (and don't overwrite each other). You can see these new files (and contents) with

    $ ls
    $ cat job.*.output

There will also be an `output.txt` in the directory, which was created by our job from `input.txt`:

    $ ls output.txt
    $ cat output.txt

## Challenges

* What happens if we leave the `queue` line out of a submit file?

* What happens if we write only `queue`, with no argument?

* `condor_history -long username` gives a LOT of extended information about your past jobs, ordered as key-value pairs.  Try it with your a single job from your last cluster:

    $ condor_history -long ######.0

<!-- <div class="keypoints" markdown="1">

#### Key Points
*   HTCondor shedules and monitors your Jobs.
*   To submit a job to HTCondor, you must prepare the job execution and job submission scripts.
*   *condor_submit* - HTCondor's job submission command.
*   *condor_q* - HTCondor's job monitoring command.
*   *condor_rm* - HTCondor's job removal command.
</div> -->

