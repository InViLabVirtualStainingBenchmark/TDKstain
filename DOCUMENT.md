# Documentation

<!--
This file lives in the root of every forked repo.
Fill it in as you go. Do not reconstruct it after the fact.
Keep entries factual and brief. The audience is a future person
reproducing your setup on a different machine or the HPC cluster.
-->

---

## Model Info

<!--
Copy this information from the upstream repo's README and paper.
"Paired or unpaired" refers to whether the model assumes paired training data.
If the model is domain-specific to virtual staining, note the exact staining task (e.g. H&E to HER2 IHC).
-->

- **Model name:** TDKStain
- **Upstream repo URL:** https://github.com/balball/TDKstain
- **Fork URL:** https://github.com/InViLabVirtualStainingBenchmark/TDKstain
- **Upstream last commit date:** Jul 4, 2024
- **Paper / citation:**
- **Paired or unpaired assumption:** Paired
- **Intended staining task (if domain-specific):** H&E to IHC HER2

---

## Environment Claimed by Authors

<!--
Record exactly what the authors say in their README or requirements file.
Do not adjust or interpret -- copy their stated versions.
"Requirements file present" should note the filename if it exists.
If no version is specified for Python or PyTorch, write "not specified".
-->

- **Python version:** 3.9.7
- **PyTorch version:** 2.3.1
- **CUDA version:** not specified
- **Installation method:** conda & pip 
- **Requirements file present:** requirements.txt 
- **Pretrained weights available:** yes (2 models)
- **Pretrained weights notes:** Hosted on GD (At risk, may rot. Manually downloaded 2025-04)
<!-- Where are they hosted? Are they behind a login? Is the link likely to rot (GDrive, Dropbox, personal server)? -->

---

## Environment Actually Used

<!--
Record the environment you actually created and tested in.
If you deviated from what the authors specified, briefly note why (e.g. "authors' version not compatible with CUDA 12.1").
Conda env name should follow the convention: the model's short name.
-->

- **Python version:** 3.9.7
- **PyTorch version:** 2.3.1
- **CUDA version:** 12.1
- **Conda environment name:** TDKStain
- **Date tested:** 02/04/2026
- **Hardware:** RTX 4090, WSL2 on Windows 11

---

## Installation

<!--
Follow the authors' README exactly before making any changes.
Record the commands you ran in order.
If an error occurred, paste the key line of the error (not the full traceback) and then record the fix.
If installation succeeded without issues, write "No issues."
-->

### Commands Run

```bash
conda create -n TDKstain python==3.9.7
conda activate TDKstain
pip install -r requirements.txt
```

### Issues and Fixes

<!--
Format: problem encountered -> fix applied.
If no issues, write "None."
-->

| Issue | Fix Applied |
| --- | --- |
|  |  |

### GPU Confirmation

<!--
Paste the output of the check below so there is proof the GPU was visible.
Command: python -c "import torch; print(torch.cuda.is_available(), torch.cuda.get_device_name(0))"
-->

```
True NVIDIA GeForce RTX 4090
```

---

## Dataset Preparation

<!--
Record how the dataset was prepared for this specific model.
"Format expected" means what folder layout or file structure the model's data loader assumes
(e.g. side-by-side paired images, separate A/B folders, CSV manifest, etc.).
"Conversion applied" means any script or command you ran to reformat the standard BCI/MIST-HER2
download into the format this model needs.
If no conversion was needed, write "None -- dataset used as downloaded."
-->

- **Dataset used:** BCI
- **Format expected by model:** separate test_HE and test_IHC folders, matched filenames
- **Conversion applied:**
    
    ```bash
    python -u ./preprocess/get_dab_mask.py
    CUDA_VISIBLE_DEVICES=0 python -u ./preprocess/get_nuclei_map.py
    ```
    
- **Final folder layout used:**
    
    ```
    # ~/datasets/
    #       TDKstain-BCI/
    #               test_HE/   (20)
    #               test_IHC/  (20)
    #               train_HE/  (50)
    #               train_IHC/ (50)
    ```
    
- **Number of images used for smoke test:** 20 test images for **inference** (in /test_.. dir) & 50 image pairs for training smoke test (in /train_.. dir).

---

## Pretrained Weights

<!--
Only fill this section if pretrained weights exist.
Record the exact download source. Flag any link that is not on a stable host
(Zenodo and HuggingFace are stable; Google Drive, Dropbox, and personal servers are at risk).
Record where you placed the weights relative to the repo root.
-->

- **Download source URL:** https://drive.google.com/drive/folders/1AlmE-zLpAxaRA4JvLgecm-1v9aEnkaPY
- **Host stability:** at-risk (GDrive)
- **Weights placed at (relative path):** ./checkpoints/model1 & ./checkpoints/model2
- **Size on disk:** 463 MB

---

## Inference Smoke Test

<!--
Run inference before training if pretrained weights are available -- it is faster
and confirms the code path works independently of the training loop.
Use 10-20 images from the BCI or MIST-HER2 test split.
"Visual check" is a qualitative sanity check only -- not a metric.
Valid outcomes: "images look like expected domain", "blank/grey output", "wrong resolution", "file not written".
-->

- **Script / command run:**
    
    ```bash
    python -u test.py --results_dir ./results --eval --num_test 20 --dataroot ~/internship-models/TDKstain/datasets/TDKstain-BCI --checkpoints_dir ./checkpoints --model tdk --netG resnet_9blocks --ngf 64 --n_downsampling 3 --dataset_mode aligned --batch_size 1 --load_size 1024 --preprocess none --display_winsize 512 --serial_batches --no_flip --name model1 --epoch latest --gpu_ids 0
    ```
    
- **Output folder:**  results/model1/test_latest/images/
- **Number of output images produced:** 60
- **Output image dimensions:**
- **Visual check result:** Good
- **Time to run (approx):** 10 sec
- **Errors or warnings during inference:** None
<!-- "None" if clean. Otherwise paste the key error line. -->

---

## Training Smoke Test

<!--Open 3-5 output images and comp
Run training for 5 epochs minimum. The goal is a clean exit, not a useful model.
Use the smallest viable batch size and the model's default resolution unless that causes an OOM error.
Always set checkpoint saving to every epoch (e.g. --save_epoch_freq 1 for pix2pix-style repos)
so there is proof a checkpoint was written.
Monitor GPU memory with: watch -n 1 nvidia-smi (run in a second terminal).
-->

- **Script / command run:**
    
    ```bash
    python -u train.py --save_epoch_freq 1 --n_epochs 5 --n_epochs_decay 0 --dataroot ~/internship-models/TDKstain/datasets/TDKstain-BCI --checkpoints_dir ./checkpoints --model tdk --netD basic --netG resnet_9blocks --ndf 64 --ngf 64 --n_downsampling 3 --num_D 3 --norm instance --dataset_mode aligned --batch_size 1 --use_tensorboard --coef_L1 10.0 --coef_mask 10.0 --coef_E 10.0 --coef_nuclei 10.0 --nef 128 --n_estimator_blocks 4 --load_size 1024 --preprocess none --display_winsize 512 --name smoke_test_bci --gpu_ids 0
    ```
    
- **Dataset used:** BCI (50 training pairs)
- **Epochs run:** 5
- **Batch size:** 1
- **Input resolution:** 1024x1024
- **Time per epoch (approx):** 
- **Peak GPU memory (approx, from nvidia-smi):**
- **Checkpoint saved:** yes
- **Checkpoint path:** checkpoints/smoke_test_bci
- **Crash or error during training:** None
<!-- "None" if clean. Otherwise paste the key error line and the fix applied. -->

- **Post-training inference check**: 
    ```bash
    python -u test.py --results_dir ./results --eval --num_test 20 --dataroot ~/internship-models/TDKstain/datasets/TDKstain-BCI --checkpoints_dir ./checkpoints --model tdk --netG resnet_9blocks --ngf 64 --n_downsampling 3 --dataset_mode aligned --batch_size 1 --load_size 1024 --preprocess none --display_winsize 512 --serial_batches --no_flip --name smoke_test_bci --epoch latest --gpu_ids 0
    ```

---

## Changes Made to Original Code

<!--
Record every change made to the original repo, no matter how small.
Do not make changes that alter model architecture or training logic.
Only changes needed for the code to run in the benchmark environment are allowed.
Add rows as needed.
-->

| File | Change Description | Reason |
| --- | --- | --- |
| train.py (command args) | --save_epoch_freq changed from 10 to 1 | default would not save within a 5-epoch smoke test |
| train.py (command args) | --n_epochs 5, --n_epochs_decay 0 | smoke test only |
| preprocess/get_dab_mask.py | substituted [DATASET DIR] with actual path | hardcoded placeholder |
| preprocess/get_nuclei_map.py | substituted [DATASET DIR] with actual path | hardcoded placeholder |

<!--
Common examples of acceptable changes:

- Pinning a dependency version in requirements.txt (e.g. torch==2.1.0) because no version was specified
- Replacing a hardcoded absolute path with a command-line argument
- Removing an import that is not used and is not installable in the benchmark environment
- Adapting the data loader to accept BCI/MIST-HER2 folder structure
-->

---

## Frozen Environment

<!--
After the smoke test passes, export and commit the environment file.
Command: conda env export > environment_<model-name>.yml
This file is what gets adapted for the HPC migration later.
Note any packages that are unusual, very large, or likely to cause conflicts on the cluster.
-->

- **Environment file:** `environment_<model-name>.yml`
- **Committed to fork:** yes / no
- **Notes on unusual or heavy dependencies:**
<!-- e.g. "requires openslide-python which needs a system-level apt install" -->

---

## HPC Readiness Notes

<!--
Fill this in after the local smoke test passes.
Flag anything that will need attention before running on the VSC cluster.
Common issues: GUI/display dependencies (matplotlib backends), hardcoded CUDA package versions,
dependencies that require apt/system installs, very large model downloads.
Leave blank until local test is complete.
-->

- **Display/GUI dependencies to remove or neutralize:**
- **System-level dependencies (non-pip/conda):**
- **Estimated GPU memory requirement:**
- **Estimated storage requirement (weights + data):**
- **Other notes for cluster adaptation:**

---

## Summary

<!--
Write 2-4 sentences summarizing what worked, what did not, and what the next step is.
Be specific. Include the overall pass/fail verdict.
This is the first thing someone reads when picking this model back up.
-->

**Overall result:** PASS

<!-- Example pass:
"[Model] smoke test completed on [date]. Inference with pretrained weights passed on 10 BCI test images.
Training ran for 5 epochs without crash. One change was made to the data loader to accept separate
source/target folders. Frozen environment committed. Ready for full benchmark run."

Example fail:
"[Model] smoke test failed at the environment step. The required PyTorch version (1.4) is not
compatible with CUDA 12.1. Blocked until a workaround is identified. Do not schedule for HPC."
-->