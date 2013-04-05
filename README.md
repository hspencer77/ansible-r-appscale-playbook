Ansible Playbook to Deploy R, RHadoop rmr2/rhdfs on AppScale Nodes
===========================

Ansible Playbook to bootstrap AppScale nodes with the R programming language, RHadoop rmr2 and rhdfs modules. 

## Pre-reqs

In order to use the playbook, the following is needed:

* appscale-tools installed - https://github.com/AppScale/appscale-tools
* ansible installed - http://ansible.cc/docs/gettingstarted.html#getting-ansible
* The AWS/Eucalyptus variables exported as global variables in your shell
  * EC2_ACCESS_KEY
  * EC2_SECRET_KEY
  * EC2_PRIVATE_KEY
  * EC2_CERT

_For more information, check out the documentation from <a href="http://www.eucalyptus.com/docs/3.2/ug/get_creds.html#get_cred">Eucalyptus</a> and <a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/setting_up_ec2_command_linux.html#set_aws_credentials_linux">AWS</a> regarding user credentials.

## Usage

This playbook is dependent upon an AppScale cluster being deployed on AWS/Eucalyptus.

### Deployment of AppScale

To deploy a cluster on AppScale, do the following:

_Note: Make sure you have appscale-tools installed, and the global varibles defined_

* In a terminal windows, execute the following:

```
$ ./appscale-tools/bin/appscale init cloud
```

* The _AppScalefile_ will be created.  The example below shows the AppScale AMI (us-east-1) that should be used.  If the cluster is being deployed on Eucalyptus, please use the EMI of the AppScale image on that cloud.  The _group_ and _keyname_ are the security group and keypair on the cloud.  They <b>DO NOT</b> need to be pre-created:

```
---
group : 'appscale-rmr'
infrastructure : 'ec2'
instance_type : 'm1.large'
keyname : 'appscale-rmr'
machine : 'ami-4e472227'
max : 3
min : 3
table : 'hypertable'
```

* After finishing editing the _AppScalefile_, stand up the AppScale cluster:

```
$ ./appscale-tools/bin/appscale up
```

* After it has finished deploying, you can check the status of the cluster by executing the following command:

```
$ ./appscale-tools/bin/appscale status
```

### R-Appscale Ansible Playbook

Now that the cluster is up, download the playbook from git:

_Note: Make sure ansible is installed before running this playbook_

```
$ git clone https://github.com/hspencer77/ansible-r-appscale-playbook.git
```

After downloading the playbook, the playbook needs to understand what hosts are in the cluster.  To configure this information, do the following:

* Get a list of all the instances part of the cluster by running the following command:

```
$ ./appscale-tools/bin/appscale status | grep amazon | grep Status | awk '{print $5}' | cut -d ":" -f 1
ec2-50-17-96-162.compute-1.amazonaws.com
ec2-50-19-45-193.compute-1.amazonaws.com
ec2-67-202-23-157.compute-1.amazonaws.com
```

* Add the list of DNS names to the ansible-r-appscale-playbook/production file.  Using the DNS names above for example, the ansible-r-appscale-playbook/production would look like this:

```
[appscale-nodes]
ec2-50-17-96-162.compute-1.amazonaws.com
ec2-50-19-45-193.compute-1.amazonaws.com
ec2-67-202-23-157.compute-1.amazonaws.com
```

Now its time the run the playbook.  Before running it, the ssh private key is needed to access the instances.  The ssh private key is located under ~/.appscale directory.  In this example, the ssh private key is appscale-rmr.key.  Use _ansible-playbook_ to execute the playbook:

```
$ ansible-playbook -i r-appscale-deployment/production --private-key=~/.appscale/appscale-rmr.key -v r-appscale-deployment/site.yml
```

After the playbook has finished executing, just SSH into the head-node of your cluster using the private key under ~/.appscale, to check out the setup.  The head-node can be found by running the following command:

```
$ ./appscale-tools/bin/appscale status
```

SSH in as root:

```
$ ssh -i ~/.appscale/appscale-rmr.key root@ec2-50-17-96-162.compute-1.amazonaws.com
```

### Test out MapReduce using wordcount.R

Once you have SSH'ed into the head node, pull out the example R program from rmr2_2.0.2.tar.gz:

```
root@appscale-image0:~# tar zxf rmr2_2.0.2.tar.gz rmr2/tests/wordcount.R
```

Now start up the R shell, and execute the wordcount.R script.  The results should look similar to the following:

```
root@appscale-image0:~# R

R version 2.15.3 (2013-03-01) -- "Security Blanket"
Copyright (C) 2013 The R Foundation for Statistical Computing
ISBN 3-900051-07-0
Platform: x86_64-pc-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

  Natural language support but running in an English locale

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

[Previously saved workspace restored]

> source('rmr2/tests/wordcount.R')
Loading required package: Rcpp
Loading required package: RJSONIO
Loading required package: digest
Loading required package: functional
Loading required package: stringr
Loading required package: plyr
13/04/05 02:33:41 INFO security.UserGroupInformation: JAAS Configuration already set up for Hadoop, not re-installing.
13/04/05 02:33:43 INFO security.UserGroupInformation: JAAS Configuration already set up for Hadoop, not re-installing.
packageJobJar: [/tmp/RtmprcYtsu/rmr-local-env19811a7afd54, /tmp/RtmprcYtsu/rmr-global-env1981646cf288, /tmp/RtmprcYtsu/rmr-streaming-map198150b6ff60, /tmp/RtmprcYtsu/rmr-streaming-reduce198177b3496f, /tmp/RtmprcYtsu/rmr-streaming-combine19813f7ea210, /var/appscale/hadoop/hadoop-unjar5632722635192578728/] [] /tmp/streamjob8198423737782283790.jar tmpDir=null
13/04/05 02:33:44 WARN snappy.LoadSnappy: Snappy native library is available
13/04/05 02:33:44 INFO util.NativeCodeLoader: Loaded the native-hadoop library
13/04/05 02:33:44 INFO snappy.LoadSnappy: Snappy native library loaded
13/04/05 02:33:44 INFO mapred.FileInputFormat: Total input paths to process : 1
13/04/05 02:33:44 INFO streaming.StreamJob: getLocalDirs(): [/var/appscale/hadoop/mapred/local]
13/04/05 02:33:44 INFO streaming.StreamJob: Running job: job_201304042111_0015
13/04/05 02:33:44 INFO streaming.StreamJob: To kill this job, run:
13/04/05 02:33:44 INFO streaming.StreamJob: /root/appscale/AppDB/hadoop-0.20.2-cdh3u3/bin/hadoop job  -Dmapred.job.tracker=10.77.33.247:9001 -kill job_201304042111_0015
13/04/05 02:33:44 INFO streaming.StreamJob: Tracking URL: http://appscale-image0:50030/jobdetails.jsp?jobid=job_201304042111_0015
13/04/05 02:33:45 INFO streaming.StreamJob:  map 0%  reduce 0%
13/04/05 02:33:51 INFO streaming.StreamJob:  map 50%  reduce 0%
13/04/05 02:33:52 INFO streaming.StreamJob:  map 100%  reduce 0%
13/04/05 02:33:59 INFO streaming.StreamJob:  map 100%  reduce 33%
13/04/05 02:34:02 INFO streaming.StreamJob:  map 100%  reduce 100%
13/04/05 02:34:04 INFO streaming.StreamJob: Job complete: job_201304042111_0015
13/04/05 02:34:04 INFO streaming.StreamJob: Output: /tmp/RtmprcYtsu/file1981524ee1a3
13/04/05 02:34:05 INFO security.UserGroupInformation: JAAS Configuration already set up for Hadoop, not re-installing.
13/04/05 02:34:07 INFO security.UserGroupInformation: JAAS Configuration already set up for Hadoop, not re-installing.
13/04/05 02:34:08 INFO security.UserGroupInformation: JAAS Configuration already set up for Hadoop, not re-installing.
13/04/05 02:34:10 INFO security.UserGroupInformation: JAAS Configuration already set up for Hadoop, not re-installing.
Deleted hdfs://10.77.33.247:9000/tmp/wordcount-test
>quit("yes")
```

Thats it!  Now you can use R to leverage MapReduce on AppScale running on AWS/Eucalyptus.  For additional information, please reference the following URL:

- https://github.com/RevolutionAnalytics/RHadoop/wiki



