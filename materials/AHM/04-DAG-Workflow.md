---
layout: lesson
root: ../..
title: Workflows with DAGMan
---
<!-- <div class="objectives" markdown="1">

#### Objectives
*   Describe the components that go into a DAG workflow.
*   Identify DAG keywords and the appropriate arguments for them.
*   Create a simple two step workflow.

-->

### DAG Components

A "DAG" is a "Directed Acyclic Graph", which you could also think of as a 
flowchart representing a workflow.  HTCondor has a way of representing 
these type of workflows in a system called "DAGMan"; creating a DAGMan 
file (usually just called a "DAG") is a way to describe a workflow of multiple 
steps, where some of the steps are HTCondor jobs.  

The format of the DAG file is this -- each line begins with a keyword, then 
a label for that step of the workflow (so different workflow steps can be 
grouped or run in a specific order) and sometimes an appropriate file name 
(a script or submit file).  Here are three common line types: 

* This line in a DAG describes a job or group of jobs to be submitted by the 
referenced submit file. 
```
JOB label submit.file
```

* This line indicates a script to be run before a `JOB` step; this script 
will be run on the submit server.  
```
SCRIPT PRE label script.sh arguments
```
There's a similar `SCRIPT POST` option that runs *after* a job completes. 

* Finally, if you have multiple job steps, they can be run in a particular order
 by using a statement like: 
```
PARENT label1 CHILD label2
```

### Our example

If we want to turn our python geolocation code into a DAG, what are the steps we need 
to run?  Which keywords from above do we need to use?  Think about this and then 
move on.  

Still in the `scaling-up` directory, create a new file called `location.dag`
that looks like this: 

``` file
JOB locate scalingup.submit
SCRIPT POST locate collect_results.sh
```

We need to actually put our results command into a script for the workflow to use; 
you can do this by creating a file called `collect_results.sh` that looks like 
this: 

``` file
#!/bin/bash

cat job.*.output | sort | uniq > all_locations.txt
```

Make the file executable by running: 

``` console
$ chmod 770 collect_results.sh
```

We have our two workflow components (the job submission and the script to 
collect results) and a DAG file tying them together.  We can now submit them 
as a single workflow by running: 

``` console
$ condor_submit_dag location.dag
```

The `dagman.out` file will indicate the status of the workflow.  Once it completes, 
it should produce the file `all_locations.txt`

### Learn More

There are a lot of other useful features in the DAGMan syntax, like `RETRY`, and 
setting specific variables with `VARS`.  To learn more: 

* [this lesson](https://swc-osg-workshop.github.io/OSG-UserTraining-Internet2-2018/novice/DHTC/04-dagman.html) walks through more specifics of a more complex DAG.
* [this presentation](https://agenda.hep.wisc.edu/event/1201/session/18/contribution/33/material/slides/1.pdf) gives an outline of DAGMan's most useful features.