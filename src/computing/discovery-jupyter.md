# Using Jupyter Notebook on Discovery

Discovery schedules jobs with [Slurm](https://slurm.schedmd.com/), so jobs are
submitted with `sbatch`, listed with `squeue` and cancelled with `scancel`.
(Discovery used PBS/Moab in the past; `mksub`, `myjobs` and `qdel` are no longer used.)

You need to be on the Dartmouth network -- on campus, or on the VPN from
off campus -- to reach Discovery or the Open OnDemand portal at all.

## Open OnDemand

The simplest way to get a notebook is Dartmouth's
[Open OnDemand](https://rc.dartmouth.edu/hpc/ood/getting-started/) portal at
<https://ood.dartmouth.edu>: pick *Jupyter Notebook* from the Interactive Apps
menu, fill in the submission form and click *Launch*.  Open OnDemand submits the
Slurm job and proxies the notebook for you, so none of the manual steps below
are needed -- no password file, and no ssh tunnel.

Two fields on that form are easy to get wrong: the partition is always `ood`,
and the account is whichever group account you belong to, or `free` if you do
not have one.  See RC's [Jupyter on Open OnDemand](https://rc.dartmouth.edu/hpc/ood/jupyter/)
page for the details.

The rest of this page describes running the server yourself, which is still
useful if you need a partition or a resource configuration that the Open
OnDemand form does not offer.

## 1. Create a Jupyter password on your Discovery account

Log onto Discovery: `ssh <username>@discovery.dartmouth.edu`

First make `jupyter` available.
Discovery does not put it on your `PATH` by default, so load the environment you intend to use
-- either an [environment module](https://rc.dartmouth.edu/hpc/intro-to-hpc/environment-modules/)
(`module avail python` lists the versions installed) or a
[conda environment](https://rc.dartmouth.edu/hpc/intro-to-hpc/conda-tutorial/) of your own.
Use the same environment in step 2, otherwise the job will start a different Jupyter than the one
you configure here, and it will not find the password you are about to set.

Create a configuration directory and password for Jupyter Notebook:

```bash
[d31548v@discovery ~]$ jupyter notebook --generate-config
Writing default config to: /dartfs-hpc/rc/home/k/d31548v/.jupyter/jupyter_notebook_config.py
[d31548v@discovery ~]$ jupyter notebook password
Enter password: 
Verify password:
[NotebookPasswordApp] Wrote hashed password to /dartfs-hpc/rc/home/k/d31548v/.jupyter/jupyter_notebook_config.json
[d31548v@discovery ~]$
```

## 2. Submit a job that starts the Jupyter Notebook server on the cluster

Use a text editor to create the new file `jupyter_notebook.sh`.
Cut and paste the following text into the file and save it.

```bash
#!/bin/bash -l

# Name of the job
#SBATCH --job-name=jupyter-notebook
# Default partition for general use; see
# https://rc.dartmouth.edu/hpc/slurm-partition/ for the alternatives
#SBATCH --partition=standard
# Walltime (job duration)
#SBATCH --time=10:00:00
# One core on one node
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
# Memory for the job
#SBATCH --mem=8G
# Where the tunnelling instructions below will be written; %j is the job ID
#SBATCH --output=jupyter-notebook-%j.out

# If you belong to a group account, request it as well:
# #SBATCH --account=<your-group-account>

# get tunneling info
export XDG_RUNTIME_DIR=""
node=$(hostname -s)
user=$(whoami)
cluster="discovery"

# This next command chooses a random port number between 8000 and 8999
port=$(( 8000 + RANDOM % 1000 ))

# print tunneling instructions to the job output file
echo -e "
# Command to create ssh tunnel:
ssh -N -f -L ${port}:${node}:${port} ${user}@${cluster}.dartmouth.edu

# Use a Browser on your local machine to go to:
localhost:${port}
"
# Make jupyter available, using the same environment as in step 1. Run
# `module avail python` to see which modules are installed, and replace the
# line below with the one you want -- or activate your own conda environment
# instead.
module load python
# This runs in the foreground until the job's walltime expires or the job
# is cancelled with scancel.
jupyter-notebook --no-browser --port=${port} --ip=${node}
```

Now submit the job to the cluster:

```bash
[d31548v@discovery ~]$ sbatch jupyter_notebook.sh
Submitted batch job 4056
```

Note the job ID that `sbatch` reports -- `4056` above.
You can list your own jobs with `squeue --me`:

```bash
[d31548v@discovery ~]$ squeue --me
JOBID PARTITION     NAME     USER ST   TIME  NODES NODELIST(REASON)
 4056  standard jupyter- d31548v  R   0:01      1 s01
```

`ST` is the job state: `PD` while it is pending and `R` once it is running.
See [Submitting a Batch Job](https://rc.dartmouth.edu/hpc/intro-to-hpc/submitting-a-batch-job/)
to learn more about submitting jobs to Discovery.

Once the job starts, the `--output` file appears in your working directory.
Use `ls` to see when it shows up:

```bash
[d31548v@discovery ~]$ ls jupyter-notebook-*.out
jupyter-notebook-4056.out
```

If it does not appear right away, wait a minute or two and try again -- the job
has to be scheduled onto a node first.  The file collects everything the job
writes to standard output and standard error, which here means the tunnelling
instructions printed by the script followed by the notebook server's own log.

Once the file appears, view it:

```bash
[d31548v@discovery ~]$ cat jupyter-notebook-4056.out

# Command to create ssh tunnel:
ssh -N -f -L 8254:s01:8254 d31548v@discovery.dartmouth.edu

# Use a Browser on your local machine to go to:
localhost:8254
```

This file contains the instructions to connect to your Jupyter Notebook from
your local desktop or laptop.

## 3. Initiate the [ssh tunnel](https://www.ssh.com/ssh/tunneling/example)

Open a terminal on your local machine and paste in the command to create the ssh tunnel:

```bash
[andy@MyLaptop ~]$ ssh -N -f -L 8254:s01:8254 d31548v@discovery.dartmouth.edu
[andy@MyLaptop ~]$
```

## 4. Open a web browser and start working

Type `localhost:8254` into your browser.
You should now be prompted for the password you created at the beginning of these instructions.
Congratulations!

## Final note about ports

The submission script provided here chooses a random port number between 8000 and 8999.
This is somewhat arbitrary as any port number above 1024 should be a legal choice for creating an ssh tunnel, as long as that port is not otherwise in use.
The relevant line in the script is:

```bash
port=$(( 8000 + RANDOM % 1000 ))
```

In order to request a specific port (for example 8888), simply replace this line with

```bash
port=8888
```

It is possible that two users may request the same port on a given compute node, which should produce an error when the notebook server starts.
It is also possible that you have an old ssh tunnel running on your local machine that is using the same port number as the one you are currently requesting, which should cause an error on your local computer.
You can avoid this by cancelling old or stale jobs in order to free up ports.

To cancel a job on the Discovery cluster use `scancel`.
For example, to cancel the job with ID 4056 that runs the server created in this tutorial, type:

```bash
[d31548v@discovery ~]$ scancel 4056
```

On your local machine, first find the process ID (PID) of the process running the ssh tunnel, for example:

```bash
[andy@MyLaptop ~]$ ps -e | grep 8254
1630 ??         0:00.02 ssh -N -f -L 8254:s01:8254 d31548v@discovery.dartmouth.edu
```

I can now see that the PID for my ssh tunnel is `1630`.
I can literally kill this job with the `kill` command:

```bash
[andy@MyLaptop ~]$ kill 1630
```
