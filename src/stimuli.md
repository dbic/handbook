# Stimuli

## Video/Audio

TODO: WiP to establish collection and automagic alignment/inclusion in BIDS datasets
of the video and audio presented to the participants for all experiments.
Envisioned purposes include, but are not limited to:

- QA and troubleshooting:

  - did MR collection start on time?

  - were there any abnormalities with video/audio stimuli delivery for that
    session?

- automated extraction of visual annotations to be used as:

  - forward modeling inputs
  - standard GLM explanatory variables

Video/audio tracks are to be stored under `stimuli/` of BIDS datasets, under
filenames matching the corresponding `.nii.gz` data.

The video capturing device at hand is a
Magewell USB Capture DVI Plus (DVI, VGA, HDMI, Composite and Component).
It is accessible under Linux via V4L and friends, but for fine-grained control over USB
we need to use proprietary dynamic libraries.
Those libraries and C++ examples
are available within the [SDK for Linux](http://www.magewell.com/files/sdk/Magewell_Capture_SDK_Linux_3.3.1.813.tar.gz).

## Behavioral data

TODO: WiP ... to automagically provide detailed `_events.tsv` files, temporally
aligned to the MRI acquisition, for BIDS datasets.
