---
layout: lesson
root: ../..
title: Scaling Up
---
<!-- <div class="objectives" markdown="1">

#### Objectives
*   Learn how to write an HTCondor submit script.
*   Learn how to submit multiple jobs at once with HTCondor.
*   Visualize how jobs distribute on OSG.

-->

### Overview

In this section, we will learn how to quickly submit multiple jobs simultaneously using HTCondor and we will visualize where these jobs run so we can get an idea of where and jobs are distributed on the Open Science Grid.

### Hostname fetching code

The following Python script finds the ClassAd of the machine it's running on and finds a network identity that can be used to perform lookups:

``` file
#!/bin/env python

import re
import os
import socket

machine_ad_file_name = os.getenv('_CONDOR_MACHINE_AD')
try:
    machine_ad_file = open(machine_ad_file_name, 'r')
    machine_ad = machine_ad_file.read()
    machine_ad_file.close()
except TypeError:
    print socket.getfqdn()
    exit(1)

try:
    print re.search(r'GLIDEIN_Gatekeeper = "(.*):\d*/jobmanager-\w*"', machine_ad, re.MULTILINE).group(1)
except AttributeError:
    try:
        print re.search(r'GLIDEIN_Gatekeeper = "(\S+) \S+:9619"', machine_ad, re.MULTILINE).group(1)
    except AttributeError:
        exit(1)
```

### Gathering network information from the OSG

Now to create a submit file and that will run in the OSG!

1.  Use the command `cd ..` to move out of the `tutorial-quickstart` folder
2.  Make a new directory for this exercise, `scaling-up` and change into it with `cd scaling-up`
3.  Copy the tutorial submit script and rename it to `scaling-up.submit` with the command `cp ../tutorial-quickstart/tutorial02.submit ./`
4.  Retrieve the geolocation script files:

``` console
wget http://proxy.chtc.wisc.edu/SQUID/osgschool18/location-wrapper.sh \
                 http://proxy.chtc.wisc.edu/SQUID/osgschool18/wn-geoip.tar.gz
```

### Geolocating machines in the OSG

Now that you have your results, it's time to summarize them. Rather than inspecting each output file individually, you can use the cat command to print the results from all of your output files at once. If all of your output files have the format location-#.out (e.g., location-10.out), your command will look something like this:

user@learn $ cat location-*.out

The * is a wildcard so the above cat command runs on all files that start with location- and end in .out. Additionally, you can use cat in combination with the sort and uniq commands to print only the unique results:

user@learn $ cat location-*.out | sort | uniq

### Mapping your results

To visualize the locations of the machines that your jobs ran on, you will be using http://www.mapcustomizer.com/. Copy and paste the collated results into the text box that pops up when clicking on the 'Bulk Entry' button on the right-hand side. Where did your jobs run?
