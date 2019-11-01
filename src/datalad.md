# DataLad

The DataLad Handbook at [http://handbook.datalad.org](http://handbook.datalad.org)
is the ultimate introduction into DataLad. 

## An example of sample analysis using DataLad & containers
 
This is a WiP example on how to organize your study as DataLad dataset and
use [datalad-container](http://docs.datalad.org/projects/container/en/latest/index.html)
extension to facilitate efficient and reproducible research:
 
```bash
# Create a dataset where we will keep all bits and pieces of a study
# With -c text2git we instruct to have data files to go to annex, while
# text files (code, docs, scripts) to git
datalad create -c text2git 1021_actions
cd 1021_actions
# Install a (sub)dataset with (Singularity) containers
datalad install -d . http://github.com/repronim/containers
# Install ReproIn'ed dataset from rolando
datalad install -d . -s rolando.cns.dartmouth.edu:/inbox/BIDS/Haxby/Sam/1021_actions bids
# Getting only a sample data (1 subject) for demonstration here
datalad get bids/sub-sid000005
```
