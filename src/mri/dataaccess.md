# Neuroimaging Data Access

All data is collected and provided for access and conversion into BIDS on rolando.cns.dartmouth.edu.

You need to request access permissions from [Research Computing](mailto:research.computing@dartmouth.edu).


```mermaid
---
config:
      theme: redux
---
flowchart TD

subgraph subGraph1["Source Data from Scanner"]
       inbox-dicom(("/inbox/DICOM"))
       details1("Accessions stored by date<br>/YYYY/MM/DD/A#")
 end
subgraph subGraph2["Converted Data"]
       inbox-bids(("/inbox/BIDS"))
       details2("Stored as /PI/Lead/Study<br/><a href='https://github.com/bids-standard/bids-specification/pull/1861#issuecomment-2183701293'>BIDS Project</a><br/>under 'sourcedata/raw/'")
 end
   local-storage("local storage")
   inbox-dicom -- "<a href='https://dbic-handbook.readthedocs.io/en/latest/mri/dataaccess.html#dicoms'>scp/rsync</a>" --> local-storage
   inbox-bids -- "<a href='https://dbic-handbook.readthedocs.io/en/latest/mri/dataaccess.html#bids'>scp/rsync/DataLad</a>" --> local-storage
   inbox-dicom --> request("PI requests<br/>dataset conversion")
   request --> conversion{"<a href='https://github.com/repronim/reproin'>Reproin<br/>conversion</a>"}
   conversion --> inbox-bids


   style subGraph1 fill:#FFFFFF,stroke:#000000
   style subGraph2 fill:#FFFFFF,stroke:#000000
   style inbox-dicom fill:#C8E6C9,stroke-width:4px,stroke-dasharray: 0,stroke:#00C853
   style details1 stroke:#000000
   style inbox-bids fill:#C8E6C9,stroke-width:4px,stroke-dasharray: 0,stroke:#00C853
   style details2 stroke:#000000
   style request stroke-width:4px,stroke-dasharray: 0,fill:#FFE0B2,stroke:#00C853
   style local-storage fill:#C8E6C9,stroke-width:4px,stroke-dasharray: 0,stroke:#00C853
   style conversion stroke:#00C853,stroke-width:4px,stroke-dasharray: 0
```

N.B. Storing under `sourcedata/raw/` is a planned solution.
At the moment it would be just in the root of the dataset.

## DICOMs

All dicoms are organized into `YEAR/MONTH/DATE/ACCESSION` hierarchy under `/inbox/DICOM`.
You can `scp` or `rsync` them to your local storage.

## BIDS

At the moment, upon request from a lab member to [Yaroslav Halchenko](mailto:yoh@dartmouth.edu), data is converted from DICOMs into BIDS within the directories hierarchy under `/inbox/BIDS`, following convention described in the [ReproIn section](./reproin.md).
If any metadata (`subject_id` or `session_id`) need to be corrected, inform ahead of time.

These directories are also DataLad datasets, so you have two options on how to transfer them:

### DataLad

Before you proceed, please refer to the [DataLad section](../basics/datalad.md) of the handbook.
When you are all set, you can do simply something like

```commandline
datalad clone myrolandoid@rolando.cns.dartmouth.edu:/inbox/BIDS/dbic/dbic-animals dbic-animals
```

This will create a local clone of the dataset, and you can use `datalad get` to get the data you need.

### Old fashion way

`scp` or `rsync`. But you would need to take care about de-referencing symlinks.

    rsync --exclude=.git --copy-links -r \
        rolando.cns.dartmouth.edu:/inbox/BIDS/dbic/dbic-animals dbic-animals


You could add `--exclude=sourcedata` and/or `--exclude=derivatives` to exclude folders with original DICOMS and possible derivatives (fmriqc, etc).

