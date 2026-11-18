# Interactive Multi-Compartment 3D Brain Reconstruction

> Reconstructing layered 3D brain anatomy from volumetric MRI and
> uncalibrated 2D MRI slice stacks, with interactive cut-away
> visualization in the browser.

## Overview

Magnetic Resonance Imaging (MRI) captures detailed information about the
brain, but it is commonly reviewed as a sequence of separate 2D
cross-sectional images. Understanding how structures relate to one
another in three dimensions can be difficult, especially when examining
internal regions hidden beneath the cerebral cortex.

This research presents a dual-pathway framework for turning MRI data
into multi-compartment 3D brain models. It is designed to support
volumetric MRI inputs with spatial metadata (such as NIfTI, NRRD, or
DICOM) as well as uncalibrated 2D MRI slice stacks (such as PNG, JPEG,
or TIFF folders) that may lack reliable slice order, spacing, or
consistent brightness.

The reconstructed surfaces are intended for interactive WebGL
visualization. Users can inspect individual anatomical layers, adjust
visibility and transparency, and reveal internal structures using
synchronized axial, coronal, and sagittal cut-away planes.

**Repository note:** This GitHub repository documents the research and
explains the project. The implementation source code is intentionally
kept private and is not included here.

## The Problem

Conventional MRI viewers present images as separate 2D slices. This can
make it difficult to understand the spatial relationships between
cortical tissue, ventricles, and deep brain structures. The
reconstruction task also faces three practical challenges:

1.  **3D anatomical understanding:** Users must mentally combine many 2D
    cross-sections to understand the whole anatomy.
2.  **Uncalibrated input data:** Some image folders lack dependable
    slice ordering, voxel spacing, or consistent brightness information.
3.  **Interactive performance:** High-resolution meshes can exceed the
    memory and frame-rate budgets of standard web browsers.

## Project Objectives

-   Convert volumetric MRI data and uncalibrated 2D slice stacks into 3D
    anatomical representations.
-   Improve MRI slice-stack consistency before reconstruction.
-   Provide two alternative tissue-estimation pathways: classical image
    processing and pretrained deep probabilistic tissue priors.
-   Reconstruct smooth surfaces for multiple brain compartments.
-   Reduce mesh complexity while preserving important anatomical
    geometry.
-   Support interactive 3D inspection with synchronized axial, coronal,
    and sagittal MRI views.

## System Pipeline

``` text
MRI INPUT
  |-- Calibrated volume (NIfTI / NRRD / DICOM)
  `-- Uncalibrated 2D image stack (PNG / JPEG / TIFF)
             |
             v
      INPUT CONDITIONING
      - Spatial resampling
      - Intensity normalization
      - Slice-order recovery (ZNCC)
      - Brightness equalization
      - Slice-spacing estimation
             |
             v
      TISSUE ESTIMATION
        /           \
       /             \
  PATHWAY A         PATHWAY B
  Classical         Pretrained deep
  segmentation      probabilistic priors
       \             /
        \           /
             |
             v
      SURFACE RECONSTRUCTION
      - Marching Cubes
      - Fragment pruning
      - Taubin smoothing
      - QEM mesh simplification
             |
             v
      INTERACTIVE 3D VIEWER
      - Independent anatomical layers
      - Visibility and opacity controls
      - Orthogonal MPR cut-away planes
      - Synchronized 2D MRI slices
             |
             v
      MESH EXPORTS: GLB / STL
```

## 1. Input Conditioning

For calibrated volumes, the framework uses available spatial information
to resample the data onto a consistent isotropic voxel grid. Intensities
are standardized and noise reduction is applied.

For uncalibrated 2D slice stacks, the conditioning stage is designed to
address missing or unreliable metadata:

-   **Slice ordering:** Zero-Mean Normalized Cross-Correlation (ZNCC)
    estimates similarity between slices and helps recover a plausible
    anatomical sequence.
-   **Brightness correction:** Foreground intensity equalization reduces
    inter-slice brightness differences and banding.
-   **Spacing estimation:** Through-plane gradient information estimates
    missing slice thickness or spacing and helps reduce geometric
    distortion.

## 2. Dual-Pathway Tissue Estimation

The framework provides two alternative approaches for estimating
anatomical compartments.

### Pathway A --- Classical segmentation

This pathway uses interpretable image-processing operations, including
Otsu / Multi-Otsu thresholding, morphological operations,
connected-component analysis, and anatomical constraints or cleanup. It
provides a classical, comparatively lightweight route to tissue
estimation.

### Pathway B --- Deep probabilistic segmentation

This pathway uses pretrained deep-learning tissue priors to produce
voxel-wise probability maps. The paper discusses pretrained models such
as Deep Atropos as a source of these priors. Unlike hard binary masks,
probability maps retain confidence information and provide continuous
scalar fields for surface extraction.

The two pathways are alternatives within the framework and are compared
during evaluation.

## 3. Surface Reconstruction and Optimization

The estimated compartments are converted into polygonal surfaces through
the following geometry-processing pipeline:

-   **Marching Cubes:** Extracts triangular isosurfaces from volumetric
    scalar fields.
-   **Fragment pruning:** Removes unwanted disconnected surface
    components.
-   **Taubin smoothing:** Reduces voxel-grid stair-step artifacts while
    limiting the shrinkage associated with conventional smoothing.
-   **Quadric Error Metrics (QEM):** Simplifies meshes by reducing
    polygon count while aiming to preserve important surface shape.

Separate surfaces are maintained for each anatomical compartment.
High-detail meshes can be retained for archival use, while simplified
meshes are prepared for interactive browser rendering.

## 4. Interactive 3D Visualization

The final stage is designed around WebGL visualization in a standard web
browser. The viewer concept includes:

-   Independent visibility controls for each anatomical compartment.
-   Opacity/transparency adjustment to reveal structures beneath outer
    layers.
-   Predefined anatomical camera viewpoints.
-   Axial, coronal, and sagittal Multi-Planar Reconstruction (MPR)
    cut-away planes.
-   Corresponding 2D MRI intensity slices synchronized with the 3D
    scene.

This allows internal anatomy to be inspected without relying only on
separate 2D images.

## Anatomical Compartments

The framework focuses on six reconstructed compartments:

  -----------------------------------------------------------------------
  Compartment                         Description / visualization role
  ----------------------------------- -----------------------------------
  Cerebrospinal Fluid (CSF)           Fluid-filled spaces and ventricular
                                      regions

  Cortical Grey Matter                Outer cortical tissue and folded
                                      surface

  White Matter                        Major myelinated subcortical tissue

  Deep Grey Matter                    Broader subcortical nuclei,
                                      including the thalamus, caudate,
                                      putamen, and globus pallidus

  Cerebellum                          Posterior-inferior cerebellar
                                      structures

  Brain Stem                          Midbrain, pons, and medulla
  -----------------------------------------------------------------------

These are broad anatomical compartments. The framework does not claim to
separately identify every individual nucleus or ventricular
substructure.

## Experimental Evaluation and Reported Results

The paper describes three evaluation configurations:

1.  **Canonical MNI152 template:** A standardized T1-weighted brain
    template with 1.0 mm isotropic spacing.
2.  **Synthetic perturbation benchmark:** Controlled Rician/Gaussian
    noise, low-frequency intensity bias fields, and anisotropic
    through-plane spacing changes.
3.  **Uncalibrated 2D slice stacks:** Randomly ordered axial images with
    inter-slice intensity variations and removed spatial affine
    metadata.

### Reported performance

  ------------------------------------------------------------------------
  Configuration     Total mesh faces End-to-end latency Browser frame rate
  --------------- ------------------ ------------------ ------------------
  Classical                 \~68,000            \~1.8 s           \~60 FPS
  pathway, coarse                                       
  grid                                                  

  Classical                 ≤150,000            \~3.4 s         55--60 FPS
  pathway, fine                                         
  grid + QEM                                            

  Deep priors,             \~420,000            \~7.9 s         18--25 FPS
  full archival                                         
  mesh                                                  

  Deep priors +             ≤150,000            \~8.2 s         55--60 FPS
  QEM budget                                            
  ------------------------------------------------------------------------

Additional findings reported in the paper:

-   QEM reduced the total mesh from approximately **420,000 to 150,000
    faces**, a reduction of more than 62%.
-   Inter-pathway Dice similarity was approximately **0.86 for white
    matter** and greater than **0.94 for the combined brain envelope**.
-   The ZNCC-based slice-ordering experiment achieved **100% Kendall's
    tau rank correlation** on non-degenerate shuffled brain stacks in
    the reported benchmark.
-   The spacing-estimation experiment kept aspect-ratio distortion
    within approximately **±12%** of true physical proportions in the
    tested conditions.

**Interpretation note:** The Dice scores measure spatial agreement
between the two segmentation pathways; they are not measurements of
clinical diagnostic accuracy against expert manual annotations.
Performance figures apply to the experimental configurations described
in the paper and are not guarantees for every dataset or computer.

## Output Formats

The research describes two mesh export formats:

-   **`.glb`** --- Binary glTF 2.0, suitable for 3D viewing and
    web-oriented asset workflows.
-   **`.stl`** --- A common surface-mesh format for desktop 3D tools and
    downstream geometry workflows.

The browser-oriented representation uses QEM simplification to keep
combined geometry within a target polygon budget.

## Scope and Limitations

This project is intended for **education, research, and prototyping**.

-   It is **not** designed or validated for clinical diagnosis,
    treatment planning, or surgical navigation.
-   Resampling and interpolation cannot recover fine anatomical details
    that were never captured in the original scan.
-   Thick slices, patient motion, noise, and intensity inhomogeneity can
    affect reconstruction quality.
-   Estimated spacing for uncalibrated stacks is an approximation when
    reliable acquisition metadata is unavailable.
-   Broad compartment labels do not provide complete fine-grained
    parcellation of all deep nuclei or ventricular substructures.
-   Segmentation quality depends on input data and the priors or
    processing assumptions used.

## Future Work

The research identifies several possible extensions:

-   Extend the geometry and visualization pipeline to thoracic and
    abdominal CT/MRI using suitable multi-organ segmentation models.
-   Explore integration with sparse-view radiographic and cone-beam CT
    (CBCT) reconstruction methods.
-   Investigate collaborative WebXR visualization and haptic interaction
    for neuroanatomical education.

## Repository Contents

This repository is a **public-facing project overview**, not a
source-code release. The implementation is private, so this repository
does not provide:

-   Source code or executable scripts
-   Model weights or trained checkpoints
-   A public installation or deployment procedure
-   A downloadable application or hosted viewer

The README explains the research motivation, system design, processing
stages, evaluation, and limitations. Any figures or presentation
materials added to the repository should be identified clearly as
research illustrations or results.

## Research Paper

**Title:** *Interactive Multi-Compartment 3D Brain Reconstruction and
Cut-Away Visualization from Volumetric and Uncalibrated MRI Scans*

**Authors:** - Soumalya Mandal - Om Nimbalkar - Shreyas Bangar - Eshan Gaikwad

**Affiliation:** Department of Computer Science (AI & ML), D Y Patil
International University, Akurdi, Pune, India.

Refer to the accompanying research paper for the complete methodology,
mathematical formulation, evaluation setup, tables, and references.

## Acknowledgement

This work explores the combination of medical image processing,
classical and deep-learning-based segmentation, computational geometry,
and browser-based 3D visualization to make multi-compartment brain
anatomy easier to inspect and understand.

## License and Usage

No open-source software license is specified for this repository. The
research description is shared for documentation purposes; the
implementation remains private. Do not assume that the code, datasets,
model weights, or other project assets are available for reuse.For any further review, clarification, or permission-related inquiries, please contact the project owner, Soumalya Mandal.
