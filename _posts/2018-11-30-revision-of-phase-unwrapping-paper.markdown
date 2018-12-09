---
title: Revision of phase unwrapping paper
layout: post
date:   2018-12-07
categories: paper revision
---

- [x] Implementation of julia 1.0 code
    - [x] The basic form
    - [x] Parallel computation
    - [x] Loading 4 images: `magnitude.nii` `phase.nii` `partiallyUnwrappedPhase.nii` and `mask.nii`. These images are 3D nii files. 
    - [x] The reconstruction is performed for a single chosen slice. The `mask.nii` could have multiple ROIs in the chosen slice: Each ROI is assigned with a unique integer 
    * Julia file location:  
    	`/Users/nkchen/Dropbox/now/FT_phase_unwrapping/main_multi_patchsize.jl`  
    	`/Users/nkchen/Dropbox/now/FT_phase_unwrapping/main_single_patchsize.jl`  
- [x] Implementation of matlab code
    * File location: `/Users/nkchen/Dropbox/now/FT_phase_unwrapping/main_single_patchsize.m`
- [x] Better explain the new path-independent 2D numerical integration
- [x] Save output in nifti format
- [x] Process phase maps obtained from multi-TE GRE scans
	* File location: `~/Dropbox/now/Madden_QSM_data/juliacode1.0/main.jl` and the results (figures) have been saved.



