# Rendering 360 degree view with few input views by PixelSplat


This is the code for **Rendering 360 degree view with few input views by PixelSplat model**

## Output
https://github.com/RyanPham19092002/Rendering-360-degree-view-with-few-input-views-by-PixelSplat/blob/main/assets/output.mp4

https://github.com/dcharatan/pixelsplat/assets/13124225/de90101e-1bb5-42e4-8c5b-35922cae8f64

## Camera-ready Updates

This version of the codebase has been updated slightly to reflect the CVPR camera-ready version of the paper (and the latest version of the paper on arXiv). Here are the changes:

* The models have all been retrained with small bug fixes and a slight architectural improvement (per-image self-attention + convolution has been replaced with only per-image self-attention in the epipolar transformer). This has improved the results slightly across the board:

| Run Name      | PSNR  | SSIM  | LPIPS |
| :------------ | ----: | ----: | ----: |
| `re10k` (old) | 25.89 | 0.858 | 0.142 |
| `re10k` (new) | 26.09 | 0.863 | 0.136 |
| `acid` (old)  | 28.14 | 0.839 | 0.150 |
| `acid` (new)  | 28.27 | 0.843 | 0.146 |

* A configuration for 3-view pixelSplat has been added. In general, it's now possible to run pixelSplat with an arbitrary number of views, although you'll need a lot of GPU memory to do so.
* The original version of the code base can be found at commit `787a8896deb232ec652426bf157f9679d4046c3c`.

## Installation

To get started, create a virtual environment using Python 3.10+:

```bash
python3.10 -m venv venv
source venv/bin/activate
# Install these first! Also, make sure you have python3.11-dev installed if using Ubuntu.
pip install wheel torch torchvision torchaudio
pip install -r requirements.txt
```

If your system does not use CUDA 12.1 by default, see the troubleshooting tips below.

<details>
<summary>Troubleshooting</summary>
<br>

The Gaussian splatting CUDA code (`diff-gaussian-rasterization`) must be compiled using the same version of CUDA that PyTorch was compiled with. As of December 2023, the version of PyTorch you get when doing `pip install torch` was built using CUDA 12.1. If your system does not use CUDA 12.1 by default, you can try the following:

- Install a version of PyTorch that was built using your CUDA version. For example, to get PyTorch with CUDA 11.8, use the following command (more details [here](https://pytorch.org/get-started/locally/)):

```bash
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

- Install CUDA Toolkit 12.1 on your system. One approach (_try this at your own risk!_) is to install a second CUDA Toolkit version using the `runfile (local)` option [here](https://developer.nvidia.com/cuda-12-1-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=runfile_local). When you run the installer, disable the options that install GPU drivers and update the default CUDA symlinks. If you do this, you can point your system to CUDA 12.1 during installation as follows:

```bash
LD_LIBRARY_PATH=/usr/local/cuda-12.1/lib64 pip install -r requirements.txt
# If everything else was installed but you're missing diff-gaussian-rasterization, do:
LD_LIBRARY_PATH=/usr/local/cuda-12.1/lib64 pip install git+https://github.com/dcharatan/diff-gaussian-rasterization-modified
```

</details>

## Acquiring Datasets

pixelSplat was trained using versions of the RealEstate10k and ACID datasets that were split into ~100 MB chunks for use on server cluster file systems. Small subsets of the Real Estate 10k and ACID datasets in this format can be found [here](https://drive.google.com/drive/folders/1joiezNCyQK2BvWMnfwHJpm2V77c7iYGe?usp=sharing). To use them, simply unzip them into a newly created `datasets` folder in the project root directory.

If you would like to convert downloaded versions of the Real Estate 10k and ACID datasets to our format, you can use the [scripts here](https://github.com/dcharatan/real_estate_10k_tools). Reach out to us if you want the full versions of our processed datasets, which are about 500 GB and 160 GB for Real Estate 10k and ACID respectively.

## Acquiring Pre-trained Checkpoints

You can find pre-trained checkpoints [here](https://drive.google.com/drive/folders/1ZYInQyBHav979dH7arITG8Z-wTSR_Bkm?usp=sharing). You can find the checkpoints for the original codebase (without the improvements from the camera-ready version of the paper) [here](https://drive.google.com/drive/folders/18nGNWIn8RN0aEWLR6MC2mshAkx2uN6fL?usp=sharing).

## Running the Code

### Training

The main entry point is `src/main.py`. Call it via:

```bash
python3 -m src.main +experiment=re10k
```

This configuration requires a single GPU with 80 GB of VRAM (A100 or H100). To reduce memory usage, you can change the batch size as follows:

```bash
python3 -m src.main +experiment=re10k data_loader.train.batch_size=1
```

Our code supports multi-GPU training. The above batch size is the per-GPU batch size.

### Evaluation

To render frames from an existing checkpoint, run the following:

```bash
# Real Estate 10k
python3 -m src.main +experiment=re10k mode=test dataset/view_sampler=evaluation dataset.view_sampler.index_path=assets/evaluation_index_re10k.json checkpointing.load=checkpoints/re10k.ckpt

# ACID
python3 -m src.main +experiment=acid mode=test dataset/view_sampler=evaluation dataset.view_sampler.index_path=assets/evaluation_index_acid.json checkpointing.load=checkpoints/acid.ckpt
```

Note that you can also use the evaluation indices that end with `_video` (in `/assets`) to render the videos shown on the website.

### Ablations

You can run the ablations from the paper by using the corresponding experiment configurations. For example, to ablate the epipolar encoder:

```bash
python3 -m src.main +experiment=re10k_ablation_no_epipolar_transformer
```

Our collection of pre-trained [checkpoints](https://drive.google.com/drive/folders/1ZYInQyBHav979dH7arITG8Z-wTSR_Bkm?usp=sharing) includes checkpoints for the ablations.

### VS Code Launch Configuration

We provide VS Code launch configurations for easy debugging.

## Camera Conventions

Our extrinsics are OpenCV-style camera-to-world matrices. This means that +Z is the camera look vector, +X is the camera right vector, and -Y is the camera up vector. Our intrinsics are normalized, meaning that the first row is divided by image width, and the second row is divided by image height.

