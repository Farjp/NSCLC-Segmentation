# Lung Tumor Segmentation using nnU-Net v2

Advanced automated lung tumor segmentation for Non-Small Cell Lung Cancer (NSCLC) patients using the state-of-the-art nnU-Net v2 framework.

## 📋 Table of Contents
- [Overview](#overview)
- [Dataset](#dataset)
- [Data Quality Control](#data-quality-control)
- [Environment Setup](#environment-setup)
- [Training Pipeline](#training-pipeline)
- [Model Configurations](#model-configurations)
- [Results](#results)
- [Hardware Requirements](#hardware-requirements)

## 🎯 Overview

This repository implements lung tumor segmentation using **nnU-Net v2**, the self-configuring framework for medical image segmentation. Our approach leverages the NSCLC-Radiomics dataset to train robust 3D segmentation models across multiple configurations and cross-validation folds.

## 📊 Dataset

**Source**: [NSCLC-Radiomics Dataset](https://www.cancerimagingarchive.net/collection/nsclc-radiomics/) from The Cancer Imaging Archive

**Original Specifications**:
- 📁 **Total Cases**: 422 CT scans with segmentation maps
- 🏥 **Patient Population**: Non-Small Cell Lung Cancer patients (various stages)
- 💾 **Format**: DICOM files with corresponding segmentation masks

**Final Dataset**: 412 patients (after quality control exclusions)

## ⚠️ Data Quality Control

### Excluded Patient IDs
The following 10 patient IDs were excluded due to data quality issues:

| Patient ID | Issue | Status |
|------------|-------|--------|
| **LUNG1-014** | Missing CT slice(s) - incomplete volume | ❌ Excluded |
| **LUNG1-021** | Missing CT slice(s) + unusual segmentation | ❌ Excluded |
| **LUNG1-083** | Contour file processing error (metadata clash) | ❌ Excluded |
| **LUNG1-085** | Missing CT slice(s) - incomplete volume | ❌ Excluded |
| **LUNG1-095** | Incorrect CT image orientation | ✅ **Fixed in update** |
| **LUNG1-127** | Segmentation interpolation error between slices | ❌ Excluded |
| **LUNG1-128** | Post-operative case (no gross tumor volume) | ❌ Excluded |
| **LUNG1-137** | Contour file processing error (metadata clash) | ❌ Excluded |
| **LUNG1-158** | Contour file processing error (metadata clash) | ❌ Excluded |
| **LUNG1-246** | Incorrect CT image orientation | ✅ **Fixed in update** |

### Quality Control Sources
- **TCIA Lung1 Wiki Documentation**
- **Aerts et al.** (original dataset citation)
- **Ferrante et al. (2022)** - peer-reviewed exclusion criteria
- **Braghetto et al. (2022)** - validation studies

**Final Training Set**: 412 patients (422 - 10 excluded cases)

## 🔧 Environment Setup

### Directory Configuration
```bash
export nnUNet_raw="/nnUnet/nnUNet_raw_data"
export nnUNet_preprocessed="nnUnet/nnUNet_preprocessed"
export nnUNet_results="/nnUnet/nnUNet_results"
```

### Dataset Preparation
```bash
# Data preprocessing and planning
nnUNetv2_plan_and_preprocess -d 001
```

## 🚀 Training Pipeline

### 3D Full Resolution Training
```bash
# Train all 5 folds for 3D Full Resolution
nnUNetv2_train 001 3d_fullres 0 --npz
nnUNetv2_train 001 3d_fullres 1 --npz
nnUNetv2_train 001 3d_fullres 2 --npz
nnUNetv2_train 001 3d_fullres 3 --npz
nnUNetv2_train 001 3d_fullres 4 --npz
```

### 3D Low Resolution Training
```bash
# Train all 5 folds for 3D Low Resolution (GPU 1)
CUDA_VISIBLE_DEVICES=1 nnUNetv2_train 001 3d_lowres 0 --npz
CUDA_VISIBLE_DEVICES=1 nnUNetv2_train 001 3d_lowres 1 --npz
CUDA_VISIBLE_DEVICES=1 nnUNetv2_train 001 3d_lowres 2 --npz
CUDA_VISIBLE_DEVICES=1 nnUNetv2_train 001 3d_lowres 3 --npz
CUDA_VISIBLE_DEVICES=1 nnUNetv2_train 001 3d_lowres 4 --npz
```

### 3D Cascade Training
The cascade training combines low-resolution and full-resolution predictions for optimal performance.

## 🔧 Model Configurations

| nnU-Net Mode | VRAM Required | Memory Efficiency | Use Case |
|--------------|---------------|-------------------|----------|
| **2D** | ~4–8 GB | 🟢 High | Low memory systems |
| **3D LowRes** | ~8–12 GB | 🟡 Medium | Balanced performance |
| **3D FullRes** | 12–18 GB | 🟠 Low | High-resolution segmentation |
| **3D Cascade** | 16–24+ GB | 🔴 Very Low | Maximum accuracy |

## 📈 Results

### 🏆 3D Full Resolution Results
**Training Period**: May 29, 2025

| Fold | Validation Dice Score | Performance |
|------|----------------------|-------------|
| Fold 0 | **0.65** | 🟢 Best |
| Fold 1 | **0.63** | 🟡 Good |
| Fold 2 | **0.54** | 🟠 Moderate |
| Fold 3 | **0.64** | 🟡 Good |
| Fold 4 | **0.60** | 🟡 Good |

**Average Dice Score**: **0.612**

### 🎯 3D Low Resolution Results  
**Training Period**: June 9, 2025

| Fold | Validation Dice Score | Performance |
|------|----------------------|-------------|
| Fold 0 | **0.67** | 🟢 Best |
| Fold 1 | **0.67** | 🟢 Best |
| Fold 2 | **0.54** | 🟠 Moderate |
| Fold 3 | **0.64** | 🟡 Good |
| Fold 4 | **0.60** | 🟡 Good |

**Average Dice Score**: **0.624**

### 🚀 3D Cascade Results
**Training Period**: June 18, 2025

| Fold | Validation Dice Score | Performance |
|------|----------------------|-------------|
| Fold 0 | **0.67** | 🟢 Best |
| Fold 1 | **0.63** | 🟡 Good |
| Fold 2 | **0.54** | 🟠 Moderate |
| Fold 3 | **0.62** | 🟡 Good |
| Fold 4 | **0.60** | 🟡 Good |

**Average Dice Score**: **0.612**

<img width="512" height="305" alt="Performance" src="https://github.com/user-attachments/assets/cdaba34e-73e0-4aa8-97f0-98af3fdfdf5c" />




### 📊 Performance Summary

| Configuration | Average Dice | Best Fold | Consistency |
|---------------|--------------|-----------|-------------|
| **3D LowRes** | **0.624** | 0.67 (Folds 0,1) | 🟢 Most Stable |
| **3D FullRes** | **0.612** | 0.65 (Fold 0) | 🟡 Moderate |
| **3D Cascade** | **0.612** | 0.67 (Fold 0) | 🟡 Moderate |

## 💻 Hardware Requirements

### Recommended GPU Specifications

| Configuration | Minimum VRAM | Recommended GPU | Training Time |
|---------------|---------------|-----------------|---------------|
| **3D LowRes** | 8 GB | RTX 3070/4060 Ti | ~2-3 days |
| **3D FullRes** | 12 GB | RTX 3080/4070 Ti | ~3-4 days |
| **3D Cascade** | 16 GB | RTX 4080/4090 | ~5-7 days |

### Multi-GPU Training
- **GPU 0**: 3D Full Resolution models
- **GPU 1**: 3D Low Resolution models
- **Memory Management**: `--npz` flag for efficient storage

## 🛠️ Installation

### Prerequisites
```bash
# Create conda environment
conda create -n nnunet python=3.9
conda activate nnunet

# Install nnU-Net v2
pip install nnunetv2

# Install additional dependencies
pip install torch torchvision torchaudio
pip install SimpleITK
pip install nibabel
```

### Environment Variables
Add to your `.bashrc` or `.zshrc`:
```bash
export nnUNet_raw="/path/to/nnUNet_raw_data"
export nnUNet_preprocessed="/path/to/nnUNet_preprocessed"
export nnUNet_results="/path/to/nnUNet_results"
```

## 🚀 Usage

### Quick Start
```bash
# 1. Prepare your data in nnU-Net format
# 2. Set environment variables
# 3. Preprocess data
nnUNetv2_plan_and_preprocess -d 001

# 4. Train model (example: 3d_fullres fold 0)
nnUNetv2_train 001 3d_fullres 0 --npz

# 5. Run inference
nnUNetv2_predict -i INPUT_FOLDER -o OUTPUT_FOLDER -d 001 -c 3d_fullres -f 0
```

### Cross-Validation Training
```bash
# Train all folds automatically
for fold in {0..4}; do
    nnUNetv2_train 001 3d_fullres $fold --npz
done
```

### Model Ensemble
```bash
# Combine predictions from all folds
nnUNetv2_ensemble -i FOLD_PREDICTIONS -o ENSEMBLE_OUTPUT
```

## 📊 Key Insights

### Model Performance Analysis
- **🏆 Best Configuration**: 3D Low Resolution (Average Dice: 0.624)
- **🎯 Most Consistent**: 3D Low Resolution (Folds 0 & 1: 0.67)
- **⚡ Best Balance**: 3D Low Resolution offers optimal performance-to-resource ratio
- **🔍 Challenging Cases**: Fold 2 consistently shows lower performance across all configurations

### Training Observations
- **Memory Efficiency**: Low resolution models provide better stability
- **Convergence**: All models trained successfully with 5-fold cross-validation
- **Reproducibility**: Consistent results across multiple training runs

## 📚 References

### Dataset & Documentation
- **Aerts, H. J. W. L., et al.** "Data From NSCLC-Radiomics." *The Cancer Imaging Archive* (2019)
- **TCIA NSCLC-Radiomics Documentation** - Quality control guidelines
- **Ferrante et al. (2022)** - Dataset validation and exclusion criteria
- **Braghetto et al. (2022)** - Peer-reviewed analysis protocols

### Framework
- **Isensee, F., et al.** "nnU-Net: a self-configuring method for deep learning-based biomedical image segmentation." *Nature Methods* 18.2 (2021): 203-211.


---

**Note**: This implementation demonstrates the robustness of nnU-Net v2 for medical image segmentation, achieving competitive performance across multiple training configurations on a clinically relevant lung cancer dataset.
