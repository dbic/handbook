# Stimuli

## Video/Audio

TODO: WiP to establish collection and automagic alignment/inclusion in BIDS datasets
of video and audio as presented to the participants for all experiments.
Envisioned purposes are not limited to

- QA and trouble-shooting:
  - did MR collection started in time?
  - were there any abnormalities with video/audio stimuli delivery for that 
    session?
- Automated extraction of visual annotations to be used as
  - Forward modeling inputs
  - Standard GLM explanatory variables

video/audio tracks are to be stored under `stimuli/` of BIDS datasets, under
filenames matching corresponding `.nii.gz` data.

## Behavioral data

TODO: WiP ... to automagically provide detailed, and temporally aligned to MRI 
acquisition, `_events.tsv` files for BIDS datasets.