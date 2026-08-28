# ReproIn Convention

The ReproIn project is part of the [ReproNim Center](http://ReproNim.org) suite of tools and frameworks, and was established when DBIC obtained a new Siemens 3T scanner.
ReproIn consists of two parts: a specification of how to organize and name exam cards on the scanner, and a tool to convert acquired DICOMs to the BIDS layout.

## Specification

The header of the [ReproIn heuristic][] file shipped within [HeuDiConv][] describes the details of the specification on how to organize and name study sequences at the MR console.
Currently it is as follows.

### Sequence naming on the scanner console

Sequence names on the scanner must follow this specification to avoid manual conversion/handling (split into multiple lines only for readability; there should be no spaces or newlines in the name):

```text
[PREFIX:][WIP ]<seqtype[-label]>[_ses-<SESID>]
   [_task-<TASKID>][_acq-<ACQLABEL>][_run-<RUNID>][_dir-<DIR>][<more BIDS>]
   [__<custom>]
```

where

- `[PREFIX:]` -- leading capital letters followed by ":" are stripped/ignored

- `[WIP ]` -- the prefix is stripped/ignored (added by Philips for patch sequences)

- `<...>` -- value to be entered

- `[...]` -- optional; might be nearly mandatory for some modalities (e.g.
  `_run-<RUNID>` for functional) and very optional for others.
  See the [BIDS entity table](https://bids-specification.readthedocs.io/en/stable/99-appendices/04-entity-table.html)

- `*ID` -- an alphanumerical identifier (e.g. `01`, `02`, `pre`, `post`, `pre01`)
  for a run, task or session.
  Note that it makes more sense to use numerical values for `RUNID`
  (e.g. `_run-01`, `_run-02`) for obvious sorting, and possibly
  descriptive ones for e.g. `SESID` (`_ses-movie`, `_ses-localizer`)

- `<seqtype[-label]>` --
  a known BIDS sequence type, which is usually the name of the folder under
  the subject's directory.  The (optional) label is specific per sequence type
  (e.g. the typical `bold` for `func`, `T1w` for `anat`, or `fid` for `mrs`), and can often
  (but not always) be deduced from DICOM.  The modalities known to BIDS are:

  - `anat` -- anatomical data.  It might also be collected multiple times across
    runs (e.g. if the subject is taken out of the magnet), so it could
    (optionally) have a `_run` definition attached.  For "standard anat"
    labels, please consult [BIDS specification "Anatomy imaging data"][];
    the most common ones are `T1w`, `T2w` and `angio`

  - `func` -- functional (a.k.a. task, including resting state) data.
    It typically contains multiple runs, and might have a different task
    per run (e.g. `_task-memory_run-01`, `_task-oddball_run-02`)

  - `fmap` -- field maps.  Could be spin-echo sequences with `_dir-`
    (e.g. `fmap_dir-AP`, `fmap_dir-PA`)

  - `dwi` -- diffusion weighted imaging (which can have runs as well)

  - `mrs` -- magnetic resonance spectroscopy (WiP, [BEP022](https://docs.google.com/document/d/1pWCb02YNv5W-UZZja24fZrdXLm4X7knXMiZI7E2z7mY))

- `_ses-<SESID>` (optional) --
  a session.  Having it in a single sequence within a study makes that study
  follow the "multi-session" layout.  It is common practice to place the `_ses-`
  specifier within the scout sequence name.  You can either specify an explicit
  session identifier (`SESID`) or use `_ses-{date}` in case of scanning phantoms
  or non-human subjects, when you want sessions to be coded by the acquisition
  date (see e.g. the [///dbic/QA][] dataset, acquired with such session identifiers)

- `_task-<TASKID>` (optional) --
  a short name for the task performed during that run.  If it is not provided and
  it is a `func` sequence, `_task-UNKNOWN` will be automatically added to comply with
  BIDS.  Consult <http://www.cognitiveatlas.org/tasks> for known tasks

- `_acq-<ACQLABEL>` (optional) --
  a short custom label to distinguish a different set of parameters used for
  acquiring the same modality (e.g. `_acq-highres`, `_acq-lowres`, etc.)

- `_run-<RUNID>` (optional) --
  a (typically functional) run.  The same idea as with `SESID`

- `_dir-[AP,PA,LR,RL,VD,DV]` (optional) --
  to be used for `fmap` images, whenever a pair of SE images is collected
  to estimate the field map

- `<more BIDS>` (optional) --
  any other fields (e.g. `_acq-`) from the [BIDS specification][] pertinent
  to that `seqtype`

- `__<custom>` (optional) --
  after two underscores, any arbitrary comment which will not affect the
  layout in BIDS.  That one theoretically should not be necessary though,
  and (ab)use of it would just signal a lack of thought while preparing the
  sequence name to start with, since everything could have been expressed in
  BIDS fields

### Last moment checks/FAQ

- Functional runs should have the `_task-<TASKID>` field defined.

- It is advisable to avoid using sequential `_run-<index>` across different
  functional tasks -- use a separate sequence of run indices within each task,
  e.g. `_task-1_run-01`, `_task-1_run-02`, `_task-2_run-01`, `_task-2_run-02`
  instead of `_task-1_run-01`, `_task-1_run-02`, `_task-2_run-03`, `_task-2_run-04`.

- Do not use `+`, `_` or `-` within `SESID`, `TASKID`, `ACQLABEL` or `RUNID`, so that we
  can detect "canceled" runs.

- If a run was canceled, just copy the canceled run (with the same index) and re-run
  it.  Files with an overlapping name will be considered a duplicate/canceled
  acquisition and only the last one will remain.  The others will acquire a
  `__dup0<number>` suffix.

Although we still support `-` and `+` used within `SESID` and `TASKID`, their use is
not recommended, and thus not listed here.

## The [HeuDiConv][] tool

[HeuDiConv][] is a flexible DICOM converter for organizing brain imaging data into
structured directory layouts.
The ReproIn [heuristic][] was developed within, and is now shipped with, HeuDiConv,
so it can be used independently of the ReproIn setup on any HeuDiConv
installation (pass `-f reproin` to the `heudiconv` call).

TODO: describe DBIC-specific settings etc. if to be done independently,
probably with the use of Docker and/or Singularity.

## Meta studies

In some cases it might be desirable to collect sequences (e.g. localizer runs) from
different studies.  `heudiconv` with the `reproin` heuristic can be used there as well:
just point it to a new dataset (e.g. `localizers`) and specify an empty `--locator` to avoid
re-establishing the original hierarchy.  You can provide DICOM tarballs from BIDS
datasets as input, so something like

```shell
heudiconv -f reproin --locator '' --bids --files \
  /inbox/BIDS/PI/INV/[0-9]*/sourcedata/sub-*/func/sub-*_task-{ffa,mimetic,ppa,...}*.tgz \
  -o /inbox/BIDS/PI/INV/localizers
```

TODO: convert to a containerized example.
TODO: check that it actually works ;)

## Samples

### MRS

WiP to define sensible names for MRS sequences.  BIDS MRS BEP022 is not yet
finalized, so we are only self-compliant here.

- `svs_GABA_160_rival` -> `mrs_acq-gaba_task-rival` -- produces 3 measurements:

  - `acq-gaba_task-rival`
  - `acq-gaba_task-rival_edit-on` -- `PulseSequenceTiming` + `PulseSequencePulses` + `PulseSequenceName` into the `.json`
  - `acq-gaba_task-rival_proc-diff`

- `svs_se_water_rival` -> `mrs_spec-unsup`

- `svs_se_dummy` -> `mrs_acq-quick16_spec-sup`

[///dbic/qa]: http://datasets.datalad.org/?dir=/dbic/QA

[bids specification]: https://bids-specification.readthedocs.io

[bids specification "anatomy imaging data"]: https://bids-specification.readthedocs.io/en/stable/04-modality-specific-files/01-magnetic-resonance-imaging-data.html#anatomy-imaging-data

[heudiconv]: https://github.com/nipy/heudiconv

[heuristic]: https://github.com/nipy/heudiconv/blob/master/heudiconv/heuristics/reproin.py

[reproin heuristic]: https://github.com/nipy/heudiconv/blob/master/heudiconv/heuristics/reproin.py
