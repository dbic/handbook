# ReproIn Convention

ReproIn project is a part of the [ReproNim Center](http://ReproNim.org)
suite of tools and frameworks, and was established when DBIC obtained a new
Siemens 3T Scanner.   ReproIn consists of two parts: a specification on how to 
organize and name exam cards in the scanner, and a tool to convert acquired
DICOMs to BIDS layout.


## Specification

The header of the [ReproIn heuristic] file shipped within [HeuDiConv] describes details of the
specification on how to organize and name study sequences at MR console.  Currently it is

### Sequence naming on the scanner console

Sequence names on the scanner must follow this specification to avoid manual
conversion/handling (split in multiple lines only for visualization, there 
should be no spaces or new lines in the name):

    [PREFIX:][WIP ]<seqtype[-label]>[_ses-<SESID>]
       [_task-<TASKID>][_acq-<ACQLABEL>][_run-<RUNID>][_dir-<DIR>][<more BIDS>]
       [__<custom>]

where

 - `[PREFIX:]` - leading capital letters followed by ":" are stripped/ignored
 - `[WIP ]` - prefix is stripped/ignored (added by Philips for patch sequences)
 - `<...>` - value to be entered
 - `[...]` - optional -- might be nearly mandatory for some modalities (e.g.,
         `_run-<RUNID>` for functional) and very optional for others. 
         See [BIDS IV. Entity Table](https://bids-specification.readthedocs.io/en/stable/99-appendices/04-entity-table.html)
 - `*ID` - alpha-numerical identifier (e.g., `01`, `02`, `pre`, `post`, `pre01`)
       for a run, task, session. Note that makes more sense to use numerical 
       values for RUNID (e.g., `_run-01`, `_run-02`) for obvious sorting and possibly
       descriptive ones for e.g., `SESID` (`_ses-movie`, `_ses-localizer`)


- `<seqtype[-label]>`
   is a known BIDS sequence type which is usually a name of the folder under
   subject's directory. And (optional) label is specific per sequence type
   (e.g. typical `bold` for `func`, or `T1w` for `anat`, `fid` for `mrs`), which could often
   (but not always) be deduced from DICOM. Known to BIDS modalities are:

   - `anat` - anatomical data.  Might also be collected multiple times across
            runs (e.g. if subject is taken out of magnet etc), so could
            (optionally) have `_run` definition attached. For "standard anat"
            labels, please consult to [BIDS specification "Anatomy imaging data"]
            but most common are `T1w`, `T2w`, `angio`,
   - `func` - functional (AKA task, including resting state) data.
            Typically contains multiple runs, and might have multiple different
            tasks different per each run
            (e.g., `_task-memory_run-01`, `_task-oddball_run-02`),
   - `fmap` - field maps. Could be spin-echo sequences with `_dir-`
            (e.g., `fmap_dir-AP`, `fmap_dir-PA`),
   - `dwi`  - diffusion weighted imaging (also can as well have runs),
   - `mrs`  - magnetic resonanse spectroscopy (WiP [BEP022](https://docs.google.com/document/d/1pWCb02YNv5W-UZZja24fZrdXLm4X7knXMiZI7E2z7mY))

- `_ses-<SESID>` (optional)
    a session.  Having a single sequence within a study would make that study
    follow "multi-session" layout. A common practice to have a _ses specifier
    within the scout sequence name. You can either specify explicit session
    identifier (SESID) or just say to maintain, create (starts with 1).
    You can also use `_ses-{date}` in case of scanning phantoms or non-human
    subjects and wanting sessions to be coded by the acquisition date.
    (see e.g., [///dbic/QA] dataset acquired with such session identified).

- `_task-<TASKID>` (optional)
    a short name for a task performed during that run.  If not provided and it
    is a func sequence, `_task-UNKNOWN` will be automatically added to comply with
    BIDS. Consult http://www.cognitiveatlas.org/tasks on known tasks.

- `_acq-<ACQLABEL>` (optional)
    a short custom label to distinguish a different set of parameters used for
    acquiring the same modality (e.g., `_acq-highres`, `_acq-lowres`, etc)

- `_run-<RUNID>` (optional)
    a (typically functional) run. The same idea as with SESID.

- `_dir-[AP,PA,LR,RL,VD,DV]` (optional)
    to be used for fmap images, whenever a pair of the SE images is collected
    to be used to estimate the fieldmap.

- `<more BIDS>` (optional)
    any other fields (e.g., `_acq-`) from the [BIDS specification] pertinent
    to that `seqtype`.

- `__<custom>` (optional)
  after two underscores any arbitrary comment which will not matter to how
  layout in BIDS. But that one theoretically should not be necessary,
  and (ab)use of it would just signal lack of thought while preparing sequence
  name to start with since everything could have been expressed in BIDS fields.

### Last moment checks/FAQ:

- Functional runs should have `_task-<TASKID>` field defined
- It is advisable to avoid using sequential `_run-<index>` through out different
  functional tasks -- use separate sequences of run indices within each task. E.g.
  `_task-1_run-01`, `_task-1_run-02`, `_task-2_run-01`, `_task-2_run-02` instead of  
  `_task-1_run-01`, `_task-1_run-02`, `_task-2_run-03`, `_task-2_run-04`.  
- Do not use `+`, `_` or `-` within SESID, TASKID, ACQLABEL, RUNID,  so we
  could detect "canceled" runs.
- If run was canceled -- just copy canceled run (with the same index) and re-run
  it. Files with overlapping name will be considered duplicate/canceled session
  and only the last one would remain.  The others would acquire
  `__dup0<number>`  suffix.

Although we still support `-` and `+` used within SESID and TASKID, their use is
not recommended, thus not listed here

[BIDS specification]: https://bids-specification.readthedocs.io
[BIDS specification "Anatomy imaging data"]: https://bids-specification.readthedocs.io/en/stable/04-modality-specific-files/01-magnetic-resonance-imaging-data.html#anatomy-imaging-data
[///dbic/QA]: http://datasets.datalad.org/?dir=/dbic/QA

## The [HeuDiConv] tool

[HeuDiConv] is 
  a flexible DICOM converter for organizing brain imaging data into structured
  directory layouts.
  ReproIn [heuristic] was developed and now is shipped within HeuDiConv,
  so it could be used independently of the ReproIn setup on any HeuDiConv
  installation (specify `-f reproin` to heudiconv call).

TODO: describe DBIC specific settings etc. if to be done independently.
Probably with use of docker and/or singularity

[HeuDiConv]: https://github.com/nipy/heudiconv
[DataLad]: http://datalad.org
[ReproIn heuristic]: https://github.com/nipy/heudiconv/blob/master/heudiconv/heuristics/reproin.py
[specification]: https://github.com/nipy/heudiconv/blob/master/heudiconv/heuristics/reproin.py
[heudiconv-monitor]: https://github.com/nipy/heudiconv/blob/master/heudiconv/cli/monitor.py
[DBIC]: http://dbic.dartmouth.edu
[///dbic/QA]: http://datasets.datalad.org/?dir=/dbic/QA


## Meta studies

In some cases it might be desired to collect sequences (e.g., localizer runs) from
different studies.  `heudiconv` with `reproin` heuristic could be used there as well,
just point to a new dataset (e.g. `localizers`) and specify empty `--locator` to avoid
re-establishing the original hierarchy.  You could provide dicom tarballs from BIDS
datasets as input, so something like

  heudiconv -f reproin --locator '' --bids --files \
    /inbox/BIDS/PI/INV/[0-9]*/sourcedata/sub-*/func/sub-*_task-{ffa,mimetic,ppa,...}*.tgz \
    -o /inbox/BIDS/PI/INV/localizers
  
TODO: convert to containerized example
TODO: check that it actually works ;)


## Samples

### MRS

WiP to define sensible names for MRS sequences. BIDS MRS BEP022 is not yet
finalized, so we are only self-compliant here

- `svs_GABA_160_rival` -> `mrs_acq-gaba_task-rival`  -- produces 3 measurements
  - `acq-gaba_task-rival`
  - `acq-gaba_task-rival_edit-on`   PulseSequenceTiming + PulseSequencePulses + PulseSequenceName into .json
  - `acq-gaba_task-rival_proc-diff`
`svs_se_water_rival` -> `mrs_spec-unsup`
`svs_se_dummy`       -> `mrs_acq-quick16_spec-sup`
