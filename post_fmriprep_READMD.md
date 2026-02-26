# Task based fMRI analysis using FSL
running Feat from GUI
- required input data
- (eprime <path>)
**input image** found under fmriprep output folder and ends with `task-faces_space-MNI152NLin2009cAsym_desc-preproc_bold.nii.gz`
**confound regressors** found under fmriprep output folder and ends with `task-faces_desc-confounds_timeseries.tsv`
   (script for generating the confounds.txt)
   - path to script and description of 24 params based on discussion w Colin
   
Looping over participants to process everyone

