# Everything you need to know about Discovery HPC

Discovery is a HPC cluster which DBIC users could utilize to run heavy computation.
[Discovery Overview](https://rc.dartmouth.edu/index.php/discovery-overview/)
and pages within provide official generic information about Discovery - what it
provides and how to use it.  This section provides additional, typically DBIC 
users specific information and hints.

## The "modules"

TODO: brief intro into [modules](http://modules.sourceforge.net/) - the system
used a collection of available environments.

For the purpose of using DataLad, please use `python/3.7-Anaconda-datalad` 
module which you can enable via `module load python/3.7-Anaconda-datalad`: 

```bash
[d31548v@discovery7 ~]$ which datalad
/usr/bin/which: no datalad in (/dartfs-hpc/admin/opt/el7/intel/...
[d31548v@discovery7 ~]$ module load python/3.7-Anaconda-datalad 
[d31548v@discovery7 ~]$ which datalad
/optnfs/common/miniconda3-datalad/bin/datalad
```
 

## The Great and Evil ACL

If you drag&drop a file from your Windows OS machine into the mounted POSIX 
environment, its permissions now will be ACLed and you will not be able to do 
anything with that file until you use `nfs4_setfacl -e filename` to interactive 
remove "ace"s from the file permissions. (TODO: JH example)

TODO
https://rc.dartmouth.edu/wp-content/uploads/2019/04/Intro_to_Cluster.pdf

## POSIXy filesystem(s) for git-annex/DataLad inspired

TODO

## Password less authentication for SSH

1. modify ~/.ssh/config to have a section like

        Host discovery7.hpcc.dartmouth.edu discovery7
	  User <netid>
          GSSAPIAuthentication yes
          GSSAPIDelegateCredentials yes

2. install kerberos client utilities.

On Debian: `sudo apt-get install krb5-user`

On MacOS with [Homebrew](https://brew.sh/): `brew install krb5`

3. initialize your kerberos token

        kinit <NETID>@KIEWIT.DARTMOUTH.EDU
        
where `<NETID>` is your Dartmouth NetID (like `d11191d`).  Use `klist` command
to see if there is an active token.  As you will see from `klist` output, that 
token has expiration date, which is 10 hours from the moment you `kinit`ed it. 
You could use `kinit -R` (or just ssh again) to refresh the ticket.  It will be
refreshed for up to 30 days. 


TODO: make it even more sophisticated (auto-updated)

Now you should be able to just `ssh discovery7` (or `ndoli` -- TODO JH)

## Using Jupyter Notebook on Discovery

The following instructions allow you to run a Jupyter Notebook on Discovery from 
your local web browser using an ssh tunnel.

* Create a Jupyter password on your discovery account

Log onto discovery: `ssh <username>@discovery7.dartmouth.edu`

Create a configuration directory and password for Jupyter Notebook:

```bash
[d31548v@discovery7 ~]$ jupyter notebook --generate-config
Writing default config to: /dartfs-hpc/rc/home/k/d31548v/.jupyter/jupyter_notebook_config.py
[d31548v@discovery7 ~]$ jupyter notebook password
Enter password: 
Verify password:
[NotebookPasswordApp] Wrote hashed password to /dartfs-hpc/rc/home/k/d31548v/.jupyter/jupyter_notebook_config.json
[d31548v@discovery7 ~]$
```

* Submit a job to Discovery cluster that starts the Jupyter Notebook server on the cluster

Use a text editor to create the new file `jupyter_notebook.pbs`. 
Cut and paste the following text into the file and save.

```console
#!/bin/bash -l
#PBS -q default
#PBS -N Jupyter_notebook
#PBS -l walltime=10:00:00
#PBS -l nodes=1:ppn=1
#PBS -l feature=bigmem # to request 8 Gb per core


# get tunneling info
XDG_RUNTIME_DIR=""
node=$(hostname -s)
user=$(whoami)
cluster="discovery7"

# This next command chooses a random port number between 8000 and 8999
port=`echo $(( 8000 + RANDOM % 1000 ))`


# print tunneling instructions jupyter-log
echo -e "
# Command to create ssh tunnel:
ssh -N -f -L ${port}:${node}:${port} ${user}@${cluster}.dartmouth.edu

# Use a Browser on your local machine to go to:
localhost:${port}  
"
module load python
jupyter-notebook --no-browser --port=${port} --ip=${node}
# keep it up and running
sleep 3600
```

Now submit the job to the cluster:

```bash
[d31548v@discovery7 ~]$ mksub jupyter_notebook.pbs
1310001.pearl.hpcc.dartmouth.edu
[d31548v@discovery7 ~]$
```

Note the number of the submitted job `1310001`. This is the unique Job ID for the 
process running your Jupyter Notebook. You can list all of your jobs using 
the `myjobs` command, which will list all of your currently submitted jobs, 
including this one.  For example:

```bash
[d31548v@discovery7 ~]$ myjobs

active jobs------------------------
JOBID              USERNAME      STATE PROCS   REMAINING            STARTTIME

1310001             d31548v    Running     1     9:59:35  Thu Dec  5 16:35:11

1 active job             1 of 2456 processors in use by local jobs (0.04%)
			  1 of 124 nodes active      (0.81%)

eligible jobs----------------------
JOBID              USERNAME      STATE PROCS     WCLIMIT            QUEUETIME


0 eligible jobs   

blocked jobs-----------------------
JOBID              USERNAME      STATE PROCS     WCLIMIT            QUEUETIME


0 blocked jobs   

Total job:  1
```

Once the job has been submitted, it will take a minute or two before two new files
will appear in your working directory. Use `ls` to see when they show up:

```bash
[d31548v@discovery7 ~]$ ls Jupyter*
Jupyter_notebook.e1310001  Jupyter_notebook.o1310001
```

If these files don't appear right away, wait a minute or two and try again. The filenames 
are formatted to have `<job_name>.[o,e]<job_ID>` where `<job_name>` is the name you
specified in the submission script with the line: `#PBS -N Jupyter_notebook`, and the
`<job_ID>` is the assigned unique job ID number. The `e` file contains any errors 
reported by your job, and the `o` files contain any text your job prints to standard 
output. See [Scheduling Jobs](https://rc.dartmouth.edu/index.php/using-discovery/scheduling-jobs/)
to learn more about submitting jobs to Discovery.

Once the files appear, view the output file:

```bash
[d31548v@discovery7 ~]$ cat Jupyter_notebook.o1310001
------------------------- Prologue ----------------------------
	 Started: Thu Dec  5 16:35:20 EST 2019
	  Job ID: 1310001.pearl.hpcc.dartmouth.edu
	 User ID: d31548v
	Group ID: rc-users
	Job Name: Jupyter_notebook
 Resources Req d: walltime=10:00:00,nodes=1:ppn=1,neednodes=1:ppn=1
      Queue Name: default
    Account Name: 
------------------------- Prologue ----------------------------


# Command to create ssh tunnel:
ssh -N -f -L 8254:n12:8254 d31548v@discovery7.dartmouth.edu

# Use a Browser on your local machine to go to:
localhost:8254  

```

This file contains instructions to connect to your Jupyter notebook from your
local desktop or laptop. 

* Initiate the [ssh tunnel](https://www.ssh.com/ssh/tunneling/example)

Open a terminal on your local machine and paste in the command to create the ssh tunnel:

```bash
[andy@MyLaptop ~]$ ssh -N -f -L 8254:n12:8254 d31548v@discovery7.dartmouth.edu
[andy@MyLaptop ~]$
```



* Open a web browser and start working

	Type `localhost:8254` into your browser. You should now be prompted for the password you created
	at the beginning of these instructions. Congratulations!

FINAL NOTE ABOUT PORTS:

The submission script provided here chooses a random port number between 8000 and 8999. This is
somewhat arbitrary as any port number above 1024 should be a legal choice for creating an ssh tunnel, as 
long as that port is not otherwise in use. The relevant line in the script is:

```bash
port=`echo $(( 8000 + RANDOM % 1000 ))`
``` 

In order to specify a specific port (for example 8888), simply replace this line with

```bash
port=8888
```

It is possible that two users may request the same port on a given compute node, which should 
produce an error when you try to submit the job to run the Jupyter Notebook. It is also possible
that you have an old ssh tunnel running on your local machine that is using the same port number as
the one you are currently requesting, which should cause an error on your local computer. You can avoid
this by killing old or stale jobs in order to free up ports.

To kill jobs on the Discovery cluster use `qdel`. For example to kill the job running the server on Discovery 
created in this tutorial with job ID 1310001, type:

```bash
[d31548v@discovery7 ~]$ qdel 1310001
```

On your local machine, first find the process id (PID) for the process running the ssh tunnel, for example:

```bash
[andy@MyLaptop ~]$ ps -e | grep 8254
1630 ??         0:00.02 ssh -N -f -L 8254:n12:8254 d31548v@discovery7.dartmouth.edu
```

I can now see that the PID for my ssh tunnel is `1630`. I can literally kill this job with the `kill` command:

```bash
[andy@MyLaptop ~]$ kill 1630
```
