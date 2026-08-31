# Slurm Job Scheduler and 'dbic' Slurm Account

# Tips

These are possibly useful little things you can do to make it easier to work with SLURM, the job scheduler used on Discovery/Ndoli (and other places beyond).

## Do/Don’t

- **DON’T** update a container file you’re currently using. They are mounted as read only volumes during execution, and even a minor change can lead to a low level error at some point.
- **DO** clean up any temporary files in shared scratch space, either manually or automatically. Scratch space is cleaned periodically, but slowly enough that it can completely fill (which will cause jobs to fail for any users who try to use it). See sample code for automatic cleanup below.

## Cleanup on Exit

Especially if you need to use scratch space for temporary files, it’s a good idea to do a cleanup on exit. The simple way to do this when submitting a bash job to SLURM is to define a function there for cleanup.

While you will want to routinely clear scratch space, you might want to make that conditional on a non-error-free execution. Here is a simple way to define a “debugging mode” flag that when turned on will make a job keep scratch contents, but only on errors:

```bash
# if true preserves scratch directory on error (must then clear manually)
DBUG=false

# set your $SCRATCH_DIR somehow... e.g., here using common DartFS scratch space
SCRATCH_DIR=/dartfs-hpc/scratch/$(whoami)/$(uuidgen)

# This will be set to run on all exit conditions, but only preserves scratch 
# if DBUG is set.
cleanup() {
    case $? in
        0)
            rm -rf "$SCRATCH_DIR"
            echo "Cleanup completed successfully"
            ;;
        *)
				    if ${DBUG}; then
							echo "Script exited with error code $?. Temporary files preserved."
							echo "SCRATCH_DIR=${SCRATCH_DIR}"
				    else
							rm -rf "$SCRATCH_DIR"
							echo "Cleanup completed successfully"
				    fi
            ;;
    esac
}

# more specific values than EXIT can be used, e.g. SIGINT (what you get from a ctrl-C
# interrupt), but this generic code covers all and is thus useful for cleanup
trap cleanup EXIT
```

## Checking Run Info: squeue and sacct

You can use `squeue` to get information on currently queued/running jobs. 

You can also get some useful info after the fact about a completed job with `sacct`. It may be helpful to define an “alias” (actually a Bash function, since it takes an argument) for accessing this command. If in your .bashrc you add

```bash
sac(){
  sacct -j $1 -o "State,ExitCode,JobName,ReqMem,MaxRSS,MaxVMSize,Start,End"
}
```

then this will create something like an alias (`sac`) that calls `sacct` with an inquiry about the job ID that follows. This mix of output values might be useful, but can be tuned with other values based on your needs and tolerance for wide outputs.

Perhaps combine with

```bash
# squeue with wider columns and a pointer to 'sac'
alias sq='squeue --format="%12i %32j %3t %8M %3D %20R" -u $USER; echo "run sac on individual jobs for more info"'

# scancel_all for killing all your Slurm jobs en masse
# updated - now handles concurrency ('%50', etc.):
scancel_all() {
    jobids=$(squeue -u "$USER" -h -o "%i")

    # Strip concurrency suffix if present
    jobids=$(echo "$jobids" | sed 's/%.*//')

    # Remove duplicates if any (since multiple array elements fall under one job id)
    jobids=$(echo "$jobids" | sort -u)

    if [ -z "$jobids" ]; then
        echo "No jobs found running for user $USER"
        return 0
    fi

    count=$(echo "$jobids" | wc -l)
    echo "Killing $count jobs..."
    echo "$jobids" | xargs -n 1 scancel
}
```

Example:

```bash
$ sq
JOBID        NAME                             ST  TIME     NOD NODELIST(REASON)    
5226931_181  194443_splinehrf                 R   40:17    1   s02                 
5226931_180  194140_splinehrf                 R   42:27    1   s43                 
5226931_178  192540_splinehrf                 R   44:07    1   s43                 
5226931_148  175742_splinehrf                 R   46:28    1   s02                 
5226931_149  176542_splinehrf                 R   46:28    1   s02                 
5226931_150  177241_splinehrf                 R   46:28    1   s04   

$ scancel_all
Killing 6 jobs...

$ sac 5226931_181
     State ExitCode    JobName     ReqMem     MaxRSS  MaxVMSize               Start                 End 
---------- -------- ---------- ---------- ---------- ---------- ------------------- ------------------- 
CANCELLED+      0:0 SPM_spline        32G                       2025-06-19T14:11:48 2025-06-19T14:14:30 
 CANCELLED     0:15      batch              1607000K   8253288K 2025-06-19T14:11:48 2025-06-19T14:14:31 
 COMPLETED      0:0     extern                 1484K    179428K 2025-06-19T14:11:48 2025-06-19T14:14:30 
```

A nice feature of `sacct` is that you can use it to see how much memory was actually used in order to fine tune your memory requests for future runs. However, an important caveat is that it may not include everything, depending on the structure of your job: in particular, memory used by shared libraries, child processes, or kernel‑level buffers can be missed.

Note that there is a tradeoff with changing the default `sqeueue` format. If you use `--format`, SLURM disables its default pretty column spacing and instead outputs the fields exactly as defined. Thus, it is good to specify column widths for *all* columns, as above, not just ones you want to be wider than the default, or you may get a “ragged” look in the output.

### ExitCode Note

One note about error codes here: if SLURM fails in certain ways, it may not get to the point of outputting a log file. In that case, `sacct` may be your only clue to what’s going on. An exit code 53 seems to be a generic indicator of “failed and could not create log”. That happened in this example,

```
$ sac 3647719
     State ExitCode    JobName     ReqMem     MaxRSS  MaxVMSize               Start   End
 ---------- -------- ---------- ---------- ---------- ---------- ------------------- --------------
     FAILED     0:53  brainager         4G                       2025-02-14T10:51:25 2025-02-14T10:52:37
  CANCELLED     0:53      batch                                  2025-02-14T10:51:25 2025-02-14T10:52:37
  COMPLETED      0:0     extern                 1480K    179196K 2025-02-14T10:51:25 2025-02-14T10:52:37 
```

where it was because a cluster node was accidentally set to accept jobs despite being not set up yet — jobs failed mysteriously after a little over a minute. In other cases, it could be permissions, filesystem problems, incorrect paths, and so on.

## Suspend and Resume

Sometimes you might submit a lot of jobs, but then realize you want to slow down how fast they are running — e.g. if they are using too much temporary storage. You can individually or “en masse” tell Slurm to hold and release queued jobs.

To set all pending jobs to priority=0, and thus paused (status `JobHeldUser`):

```jsx
squeue -u $USER -t PENDING -h -o "%A" | xargs scontrol hold
```

To resume them all:

```jsx
squeue -u $USER -t PENDING -h -o "%A" | xargs scontrol release
```

To resume just one, or a few, just use the job ID, e.g. `scontrol release 8866548` for a single one, or a series of IDs is also accepted:

```jsx
scontrol release 8866549 8866550 8866551
```

There are also programs `slurm-hold-jobs` and `slurm-release-jobs` in the `/dartfs/rc/lab/D/DBIC/DBIC/code/bin` directory that provide a front end for these commands. By default, each holds and releases all pending jobs, as above, but both can also take a job ID, or multiple IDs, as arguments.

# Background

- Trackable resources (TRES): https://slurm.schedmd.com/tres.html

# Local Configuration

Normally the configuration is in `/etc/slurm/slurm.conf`; while this directory exists on Discovery nodes, Slurm is running with `SlurmctldParameters = enable_configless`. This means the controller has no static `slurm.conf` file on disk; the config and data are likely managed dynamically, probably via the Slurm database (`slurmdbd`). 

Some (partial?) configuration information can be obtained from `sacctmgr` (`sacctmgr show conf`).

Fairshare scheduling is not active: the priority weights for fairshare are zero, `PriorityWeightFairShare = 0`, so balancing by historic usage is off. Actually, ranking by anything seems to be off, since although `Priority Type = priority/multifactor` (which weighs a number of factors in principle), in practice all weights are zero!

```jsx
$ scontrol show config | grep -i priority
PriorityParameters      = (null)
PrioritySiteFactorParameters = (null)
PrioritySiteFactorPlugin = (null)
PriorityDecayHalfLife   = 6-00:00:00
PriorityCalcPeriod      = 00:05:00
PriorityFavorSmall      = No
PriorityFlags           =
PriorityMaxAge          = 7-00:00:00
PriorityUsageResetPeriod = DAILY
PriorityType            = priority/multifactor
PriorityWeightAge       = 0
PriorityWeightAssoc     = 0
PriorityWeightFairShare = 0
PriorityWeightJobSize   = 0
PriorityWeightPartition = 0
PriorityWeightQOS       = 0
PriorityWeightTRES      = (null)
```

Looking at the results from `sacctmgr show assoc account=dbic` shows restrictions for the `dbic` account and for users associated with it.

- The overall **`dbic`** account limit is large (`cpu=1280` — at or close to the full cluster capacity?), with a very long max walltime (`1680-00:00:00`, about 4.5 years)
- Some users had no restrictions set for a while (since fixed to impose the intended `dbic` ones described on the DBIC website), and some can have a non-standard CPU cap (usually 240) and walltime limit (typically ~163 days)
- At least one user has `share=2` instead of the usual `share=1`; this has no effect without Fairshare, but if that were active would indicate a higher priority weight

## Configuration Implications

See  https://www.dartmouth.edu/dbic/research_infrastructure/discovery.html for DBIC policies. As it mentions, 

> “…The SLURM scheduler enforces hard limits on resource allocation so that once the DBIC usage meets the limit, no more DBIC jobs can run until resources are freed.”

In other words,

- There are resource limits applied primarily at the DBIC account level, not just the per-user ones above
- Since Fairshare is disabled, a few users using the `dbic` account can saturate the allocation and block other users entirely for an indefinite period
- Individual users are not differentiated by Slurm for scheduling or limit enforcement within the dbic account

Fortunately, there have not been reported problems with access to the cluster for the most part. If there were, it’s possible Fairshare or other prioritization options could be enabled; also, per-user limits could be set more tightly. Research Computing is, though, satisfied that the Fairshare-less setup seems to work fine.
