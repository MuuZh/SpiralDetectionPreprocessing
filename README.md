to flatten the brain structure

get FreeSurfer license from https://surfer.nmr.mgh.harvard.edu/fswiki/License

# Using docker images

`docker pull bids/freesurfer`

`docker pull tigrlab/fmriprep_ciftify`

then create a folder with BIDS structure
```
BIDS_project
    - sub-01
        - anat
            sub-01_T1w.nii.gz
```

create a description files under BIDS_project folder for freesurfer to run

`dataset_description.json`:
```
{
  "Name": "Example dataset",
  "BIDSVersion": "1.4.0"
}
```

`README`:
```
test
```



then use FreeSurfer to create the surfaces

example: if my BIDS_project folder is in `E:\resarch_data\fMRI\fMRI_DMT\freesurfer_test1\`

```
docker run -ti --rm `
	-v E:\resarch_data\fMRI\fMRI_DMT\freesurfer_test1\BIDS_project:/data:ro `
	-v E:\resarch_data\fMRI\fMRI_DMT\freesurfer_test1\FS_output:/out `
	-v E:\resarch_data\fMRI\FreeSurfer\license.txt:/opt/freesurfer/license.txt:ro `
	bids/freesurfer `
	/data /out participant `
	--participant_label 01 `
	--license_file /opt/freesurfer/license.txt
```

`BIDS_project` is where the input files are, and `FS_output` is where the output files will be saved. They are mounted to the docker container as `/data` and `/out` respectively.

`--participant_label 01` is the subject label, and `--license_file /opt/freesurfer/license.txt` is the path to the license file.

Then `FS_output` will have the output files.
```
FS_output
    - sub-01
        - label
        - mri
        - scripts
        - stats
        - surf
        - tmp
        - touch
        - trash
```


# Post-freesurfer: ciftify_recon_all

```
docker run -ti --entrypoint /bin/bash --rm `
    -v E:\resarch_data\fMRI\fMRI_DMT\freesurfer_test1\FS_output:/data:ro `
    -v E:\resarch_data\fMRI\fMRI_DMT\freesurfer_test1\ciftify_out:/out `
    -v E:\resarch_data\fMRI\FreeSurfer\license.txt:/opt/freesurfer/license.txt `
    tigrlab/fmriprep_ciftify:latest
```

This will open a bash shell in the docker container. Then run the following command to run `ciftify_recon_all`:

```
ciftify_recon_all --resample-to-T1w32k --ciftify-work-dir /out --fs-subjects-dir /data sub-01
```
Then the results will be saved in `ciftify_out` folder.
```
ciftify_out
    - sub-01
        - MNINonLinear
        - T1w
```

can also run the following to visualize the results
```
cifti_vis_recon_all subject --ciftify-work-dir /out sub-01
cifti_vis_recon_all index --ciftify-work-dir /out
```


# Finally: ciftify_subject_fmri
```
docker run -ti --entrypoint /bin/bash --rm `
    -v E:\resarch_data\fMRI\fMRI_DMT\freesurfer_test1\FS_output:/data:ro `
    -v E:\resarch_data\fMRI\fMRI_DMT\freesurfer_test1\ciftify_out:/out `
    -v E:\resarch_data\fMRI\FreeSurfer\license.txt:/opt/freesurfer/license.txt `
    -v E:\resarch_data\fMRI\fMRI_DMT\DMT_clean\DMT\post1:/funcnii `
    tigrlab/fmriprep_ciftify:latest
```

here, the `funcnii` folder contains the functional data in BIDS format. Then run the following command to run `ciftify_subject_fmri`:

```
ciftify_subject_fmri --already-in-MNI --ciftify-work-dir /out /funcnii/S01_DMT_post1_clean.nii.gz sub-01 DMT
```

`S01_DMT_post1_clean.nii.gz` is the functional data in MNI space. The last argument `DMT` is the task name.

Then the results will be saved in `ciftify_out` folder.
```
ciftify_out
    - sub-01
        - MNINonLinear
            - fsaverage_LR32k
            - Native
            - Results
                - DMT
                    - DMT_Atlas_s0.dtseries.nii
            - ROIs
            - xfms
        - T1w
```

And here we have the dtseries file.