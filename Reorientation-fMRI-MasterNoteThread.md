# Reorientation fMRI Analysis - Master Note Thread

**Project Directory:** `/Users/linfenghan/Desktop/fMRI_Data_Analysis`
**Last Updated:** 2026-01-15
**Primary Contact:** [Your Name]
**Collaborators:** [List collaborators]

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Directory Structure](#directory-structure)
3. [Processing Pipeline](#processing-pipeline)
4. [Progress Tracker](#progress-tracker)
5. [Key Files and Scripts](#key-files-and-scripts)
6. [Notes and Issues](#notes-and-issues)
7. [Analysis Log](#analysis-log)

---

## Project Overview

### Research Question
[Add your research question here]

### Study Design
**Tasks:**
- `localizerMemory` - Memory localizer task (1 run per subject)
- `localizerPerception` - Perception localizer task (2 runs per subject)
- `reorientXlong1` - Reorientation task, long version 1 (2 runs per subject)
- `reorientXlong2` - Reorientation task, long version 2 (2 runs per subject)
- `reorientXshort` - Reorientation task, short version (3 runs per subject)

**Total runs per subject:** 10 functional runs

**Acquisition:**
- Multi-echo BOLD fMRI (5 echoes per acquisition)
- Phase and magnitude images acquired
- Single-band reference (sbref) images included

**Current Subjects:** 2 subjects × 2 sessions each (4 total sessions)

### Scan Parameters

**Functional (BOLD):**
- **Scanner:** Siemens Prisma 3T (SC3T)
- **Institution:** University of Pennsylvania
- **TR:** 2.3 seconds
- **Multi-echo TEs:** 5 echoes
- **Voxel Size:** 2.5 × 2.5 × 2.5 mm
- **Multiband Factor:** 2
- **Phase Encoding:** Anterior-Posterior
- **Parallel Imaging:** GRAPPA (factor 3)

**Anatomical (T1w):**
- **Sequence:** MPRAGE
- **Matrix:** 256 × 256

---

## Directory Structure

```
fMRI_Data_Analysis/
├── dset/                          # Clean BIDS dataset (master backup)
│   ├── sub-*/ses-*/               # Subject/session data
│   ├── dataset_description.json
│   └── task-*_bold.json
│
├── preproc_pipeline1/             # Pipeline WITH NORDIC processing
│   ├── sub-*/ses-*/               # Copy of BIDS data
│   ├── nordic_outputs/            # NORDIC denoised outputs
│   ├── fprep/                     # fMRIPrep outputs
│   ├── freesurfer/                # FreeSurfer outputs
│   ├── tedana/                    # TEDANA outputs
│   └── freesurfer_license.txt
│
├── preproc_pipeline2/             # Pipeline WITHOUT NORDIC (control)
│   ├── sub-*/ses-*/               # Copy of BIDS data
│   ├── fprep/                     # fMRIPrep outputs
│   ├── freesurfer/                # FreeSurfer outputs
│   ├── tedana/                    # TEDANA outputs
│   └── freesurfer_license.txt
│
├── fMRI_data_raw/                 # Raw DICOM files
├── mebold-curation-test/          # Processing scripts + Python venv
├── notes/                         # Project documentation
└── freesurfer_license.txt
```

---

## Processing Pipeline

### Processing Comparison Strategy

This project implements **two parallel preprocessing pipelines** to compare the effects of NORDIC denoising:

| Pipeline | Steps | Purpose |
|----------|-------|---------|
| **Pipeline 1** | NORDIC → fMRIPrep → TEDANA | WITH NORDIC denoising |
| **Pipeline 2** | fMRIPrep → TEDANA | WITHOUT NORDIC (control) |

Both pipelines start with identical copies of the raw BIDS data for fair comparison.

---

### Stage 0: DICOM to BIDS Conversion
**Location:** `mebold-curation-test/`
**Environment:** Apple Silicon terminal + Python virtual environment

```bash
cd /Users/linfenghan/Desktop/fMRI_Data_Analysis/mebold-curation-test
source .venv/bin/activate
python3 01_unzip_dicoms.py
sh 02_run_heudiconv.sh
python3 03_fix_bids.py
```

**Status:** ✅ Complete

---

### Pipeline 1: WITH NORDIC Processing

**Working Directory:** `preproc_pipeline1/`

#### Step 1.1: NORDIC Denoising
**Environment:** MATLAB with [MB toolbox](https://github.com/CMRR-C2P/MB) and [NORDIC_Raw](https://github.com/SteenMoeller/NORDIC_Raw)

```matlab
% Reference: sample_files_from_professor/matlab/scripts/script_20240930.m
applyNordic
```
**Status:** ⏳ Not started

#### Step 1.2: fMRIPrep
**Environment:** Rosetta terminal

```bash
cd /Users/linfenghan/Desktop/fMRI_Data_Analysis/preproc_pipeline1/
fmriprep-docker . fprep participant \
  --nthreads 2 --omp-nthreads 4 --mem-mb 16000 \
  --bold2anat-init header --me-output-echos \
  --fs-license-file freesurfer_license.txt \
  --fs-subjects-dir freesurfer
```
**Status:** ⏳ Not started (awaiting NORDIC)

#### Step 1.3: TEDANA
**Status:** ⏳ Not started (awaiting fMRIPrep)

---

### Pipeline 2: WITHOUT NORDIC (Control)

**Working Directory:** `preproc_pipeline2/`

#### Step 2.1: fMRIPrep
**Environment:** Rosetta terminal

```bash
cd /Users/linfenghan/Desktop/fMRI_Data_Analysis/mebold-curation-test
source .venv/bin/activate

fmriprep-docker \
  /Users/linfenghan/Desktop/fMRI_Data_Analysis/preproc_pipeline2 \
  /Users/linfenghan/Desktop/fMRI_Data_Analysis/preproc_pipeline2/fprep \
  participant \
  --nthreads 4 --omp-nthreads 8 --mem-mb 40000 \
  --bold2anat-init header --me-output-echos \
  --fs-license-file /Users/linfenghan/Desktop/fMRI_Data_Analysis/freesurfer_license.txt \
  --fs-subjects-dir /Users/linfenghan/Desktop/fMRI_Data_Analysis/preproc_pipeline2/freesurfer
```
**Status:** ✅ Completed (2026-01-17, ~10 hours)

#### Step 2.2: TEDANA
**Status:** ⏳ Not started

---

## Progress Tracker

### Completed
- [x] DICOM to BIDS conversion (all subjects/sessions)
- [x] BIDS validation and fixing
- [x] Pipeline directory setup
- [x] Data copied to both pipeline directories

### Pipeline 1: WITH NORDIC
- [ ] NORDIC denoising
- [ ] fMRIPrep preprocessing
- [ ] TEDANA processing

### Pipeline 2: WITHOUT NORDIC
- [x] Preprocessing scripts (05, 06, 07)
- [x] fMRIPrep preprocessing ✅ Completed (~10 hours)
- [ ] TEDANA processing

### Current Status
**Pipeline 1 NORDIC testing in progress** (2026-01-17)

---

## Key Files and Scripts

### Processing Scripts (`mebold-curation-test/`)
| Script | Purpose |
|--------|---------|
| `01_unzip_dicoms.py` | Extract DICOM archives |
| `02_run_heudiconv.sh` | Convert DICOMs to BIDS |
| `03_fix_bids.py` | Fix BIDS compliance (renames to part-mag/part-phase) |

### MATLAB Scripts
| Script | Purpose |
|--------|---------|
| `applyNordic` | NORDIC denoising |
| `tedanaPreProcess` | TEDANA processing |

---

## Notes and Issues

### Technical Notes
- **Terminal switching:** Apple Silicon for Python scripts, Rosetta for fMRIPrep
- **Python environment:** `source .venv/bin/activate`
- **Memory:** fMRIPrep requires ~16GB RAM

### MATLAB Requirements for NORDIC
- **Image Processing Toolbox** - Required by NORDIC (install via MATLAB Add-Ons)
- **Signal Processing Toolbox** - Required for `tukeywin` function (install via MATLAB Add-Ons)
- **XQuartz** - Required by AFNI tools (download from https://www.xquartz.org/, restart after install)
- **Toolboxes location:** `/toolboxes/` folder contains:
  - `NORDIC_Raw-main/` - NORDIC denoising functions
  - `MB-master/` - CMRR MB toolbox
  - `afni/` - AFNI command-line tools (called via system())

### Processing Times (for reference)
| Process | Data | Time | Notes |
|---------|------|------|-------|
| `01_unzip_dicoms.py` | 2 subjects × 2 sessions | ~few mins | Unzipping DICOM archives |
| `02_run_heudiconv.sh` | 2 subjects × 2 sessions | ~15-30 mins | DICOM → NIfTI conversion |
| `03_fix_bids.py` | 2 subjects × 2 sessions | ~few mins | BIDS fixes |
| `05_splitOffNoiseVols.py` | 2 subjects × 2 sessions | ~few mins | Split noise volumes |
| `06_assign_intendedfor.py` | 2 subjects × 2 sessions | ~few mins | Assign fieldmap metadata |
| `07_bids_cleanup.py` | 2 subjects × 2 sessions | ~few mins | Move unused files |
| fMRIPrep | 2 subjects × 2 sessions | **~10 hours** | 40GB RAM, 4 threads |
| NORDIC (test) | 1 run × 5 echoes | ~5 mins | Per run timing |
| NORDIC (full) | 2 subjects × 2 sessions | **~7 hours** | See breakdown below |

### NORDIC Timing Breakdown

**Processing rate:** ~0.056 min per volume

| Task | Volumes | Runs/session | Time/run | Time/session |
|------|---------|--------------|----------|--------------|
| localizerMemory | 194 | 1 | 10.8 min | 10.8 min |
| localizerPerception | 147 | 2 | 8.2 min | 16.4 min |
| reorientXlong1 | 242 | 2 | 13.6 min | 27.2 min |
| reorientXlong2 | 242 | 2 | 13.6 min | 27.2 min |
| reorientXshort | 122 | 3 | 6.8 min | 20.4 min |
| **Total per session** | | **10 runs** | | **~102 min (~1.7 hrs)** |

**Total for 4 sessions:** 4 × 102 min = **~408 min (~6.8 hours)**

*Note: Can run 2 subjects in parallel (separate MATLAB instances) to reduce to ~3.4 hours*

### Known Issues
- dcm2niix must be in PATH for heudiconv (symlink created in `bin/afni/`)

### B0FieldIdentifier Must Be Unique Per Subject AND Session

**Problem:** The `06_assign_intendedfor.py` script assigned the same `B0FieldIdentifier` ("phdiff_fmap") to ALL fieldmaps across all subjects and sessions. This identifier is like a "name tag" that links fieldmaps to BOLD scans. When multiple fieldmaps share the same name, fMRIPrep crashes with: `KeyError: "'phdiff_fmap' is already in mapping"`

**Important:** The B0FieldIdentifier must be unique across the **entire dataset**, not just within a session. When processing multiple subjects, each subject's fieldmaps need their own unique identifier.

**Which files were affected:**
| Folder | File Type | Field |
|--------|-----------|-------|
| `fmap/` | `*_epi.json` | `B0FieldIdentifier` |
| `func/` | `*.json` (all BOLD JSONs) | `B0FieldSource` |

**Fix:** Make identifiers unique per subject AND session:
- sub-sub002/ses-ses001 → `"phdiff_fmap_sub-sub002_ses-ses001"`
- sub-sub002/ses-ses002 → `"phdiff_fmap_sub-sub002_ses-ses002"`
- sub-sub010/ses-ses001 → `"phdiff_fmap_sub-sub010_ses-ses001"`
- sub-sub010/ses-ses002 → `"phdiff_fmap_sub-sub010_ses-ses002"`

**Command to fix:**
```bash
cd /path/to/pipeline_folder
for sub in sub-sub002 sub-sub010; do
  for ses in ses-ses001 ses-ses002; do
    for json in $sub/$ses/fmap/*_epi.json $sub/$ses/func/*.json; do
      sed -i '' "s/\"phdiff_fmap\"/\"phdiff_fmap_${sub}_${ses}\"/g" "$json"
    done
  done
done
```

---

## Analysis Log

### 2026-01-16 - Pipeline 2 fMRIPrep Started
- Ran preprocessing scripts for Pipeline 2:
  - `05_splitOffNoiseVols.py` - Split noise volumes from BOLD scans
  - `06_assign_intendedfor.py` - Assigned fieldmap metadata
  - `07_bids_cleanup.py` - Moved unused files to derivatives
- Fixed B0FieldIdentifier collision error (see notes above)
- Updated `06_assign_intendedfor.py` to use subject+session in identifiers (both master and pipeline2 copies)
- **Started fMRIPrep** for all 4 sessions combined (sub-sub002 + sub-sub010, ses-ses001 + ses-ses002)
- Command used:
  ```bash
  fmriprep-docker \
    /Users/linfenghan/Desktop/fMRI_Data_Analysis/preproc_pipeline2 \
    /Users/linfenghan/Desktop/fMRI_Data_Analysis/preproc_pipeline2/fprep \
    participant \
    --nthreads 4 --omp-nthreads 8 --mem-mb 40000 \
    --bold2anat-init header --me-output-echos \
    --fs-license-file /Users/linfenghan/Desktop/fMRI_Data_Analysis/freesurfer_license.txt \
    --fs-subjects-dir /Users/linfenghan/Desktop/fMRI_Data_Analysis/preproc_pipeline2/freesurfer
  ```
- **Status:** ✅ Completed (~10 hours total)

### 2026-01-15 - Fresh BIDS Conversion & Pipeline Setup
- Performed complete fresh BIDS conversion from raw DICOMs for all subjects/sessions
- Cleaned up redundant/interrupted scan files in raw data
- Modified `02_run_heudiconv.sh` to include all subjects and sessions
- Ran full pipeline: `01_unzip_dicoms.py` → `02_run_heudiconv.sh` → `03_fix_bids.py`
- Verified BIDS output: 400 func files, 10 anat files, 84 fmap files per session
- Created fresh pipeline directories and copied clean BIDS data to both
- **Result:** Both pipelines ready with identical starting data

### 2026-01-13 - Initial Documentation
- Created master note document
- Established two-pipeline comparison approach (NORDIC vs non-NORDIC)
- Initial directory structure planning

---

## Questions for Collaborators

1. **NORDIC Processing:** Ready to begin - any specific parameters needed?
2. **Analysis Approach:** Which space for final analysis (MNI vs native)?
3. **ROI Analysis:** Specific regions of interest (brainstem mentioned)?
4. **Task Contrasts:** What contrasts should be modeled?

---

## Contact Information
- **Principal Investigator:** [Name] - [Email]
- **Lab Manager:** [Name] - [Email]
- **Data Analyst:** [Name] - [Email]
