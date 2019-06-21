=======
DTI data: preprocessing with FSL
=======
-----------------
Goal:
-----------------
The generic2_diffsl script does a generic diffusion tensor preprocessing using fsl. 

-----------------
Configuration:
-----------------

generic2_diffsl needs FSL>=5.09 to be installed on your system and properly set up (http://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslInstallation).

-----------------
Preprocessing steps:
-----------------
The pipeline consists in the following steps:

1. Linear registration with eddy current correction and bvec rotation using the first volume of the dti data as reference.
2. B0 mapping correction if possible using magnitude and phase images from the fieldmap acquisition. The correction direction is defined automatically (default is y).
3. Brain extraction.
4. Tensor computation with weighted least squares.

-----------------
Inputs:
-----------------
The general use is:

generic2_diffsl <dtisets> <pdir> <mag> <phase> <dti1> <dti2> <dti3> ... 

* dtisets: # of dti files.
* pdir, output directory for preprocessed data (has to be the full path).
* mag, the raw magnitude image from the fieldmap.
* phase, the raw phase image from the fieldmap.
* dti#, 4D DTI image with diffusion weighted images to be concatenated.

All images must be in ANALYZE, NIFTI or NIFTI.GZ.

-----------------
Outputs:
-----------------

The script creates one folder per subject with inside:

======================================= =======================================================================================
file            Description
======================================= =======================================================================================
subject_mag                             the raw magnitude image
subject_phase 	                        the raw phase image
subject_phase_w	                        the wrapped phase ima
subject_phase_uw			                  the unwrapped phase image
subject_phase_rads			                the rad/sec phase image
subject_dti_ecc.unwarp/		              folder for the distortion correction
subject_dti			                        the raw dti data
subject_dti.bval			                  the b values
subject_dti.bvec			                  the gradient directions
subject_dti_ecc			                    the eddy current corrected dti data
subject_dti_ecc.bvec		                the bvec table corrected for rotation
subject_dti_ecc_mask		                the brain mask after eddy current correction
subject_dti_ecc_brain		                the brain extracted and eddy current corrected dti data
subject_dti_ecc_brain_*                 the diffusion measures from the tensor estimation with wls
subject_dti_ecc_dc			                the eddy current and distortion corrected dti data
subject_dti_ecc_dc_mask		              the mask after eddy current and distortion correction
subject_dti_ecc_dc_brain		            the brain extracted and distortion corrected dti data
subject_dti_ecc_dc_brain_*		          the diffusion measures from the tensor estimation after distortion correction with wls
subject_dtit.slices				              mean value per axial slices and diffusion-weighted volumes
subject_dti_ecc.log				              transformation matrix during the affine registration
subject_dti_ecc.rotation			          total angle rotation in radians and flag for 2 and 5 degrees
subject_dti_ecc_dc.log			            cost values after distortion correction (smaller is better)
subject_dti_ecc_brain_tf.log			      summary of the wls tensor fitting after eddy current correction
subject_dti_ecc_dc_brain_tf.log		      summary of the wls tensor fitting after distortion correction
======================================= =======================================================================================

*: _V1 - 1st eigenvector; _V2 - 2nd eigenvector; _V3 - 3rd eigenvector; _L1 - 1st eigenvalue; _L2 - 2nd eigenvalue; _L3 - 3rd eigenvalue; _RD – radial diffusivity
_MD - mean diffusivity; _FA - fractional anisotropy; _MO - mode of the anisotropy (oblate ~ -1; isotropic ~ 0; prolate ~ 1); _S0 - raw T2 signal with no diffusion weighting

-----------------
Automatic QC:
-----------------
Several automatic QC steps can be performed on these output files:

1. Volumes and slices

QCing the number of volumes and slices.

* File: subject_dtit.slices
* Flag: if # of volumes does not correspond

2. Slice-dropout

QCing slice by slice for divergent values indicating a signal dropout.

* File: subject_dtit.slices
* Flag: if # > 1

3. Head motion

QCing volume by volume for head rotation based on the matrix used for the spatial registration.

* File: subject_dti_ecc.rotation
* Flag: if # > 1

4. B0 mapping correction

QCing the distortion correction by looking at the cost measure of the spatial registration before and after correction.

* Files: subject_dti_ecc_dc.log
* Flag: if the cost measure is bigger after correction than without correction 

5. Tensor computation

QCing the tensor computation by using a k-means clustering on the global FA, MD, L1, L2, L3 and MO values.

* Files: subject_dti_ecc_brain_tf.log
* Flag: if not in the main clusters.

-----------------
Contact:
-----------------
Herve Lemaitre (herve.lemaitre@u-psud.fr).
