# Super-Resolution Preprocessing (CodeFormer)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Morteza-Asadi-Shalmaiy/superres-fr/blob/main/superres_fr.ipynb)

Restores and upscales low-resolution or distant faces before they're passed to face recognition, using [CodeFormer](https://github.com/sczhou/CodeFormer). Chosen over GFPGAN and Real-ESRGAN alone for better identity preservation on degraded faces — important since the goal is to help recognition match a face correctly, not just make an image look sharper.

This is a **preprocessing step** in the larger pipeline: it runs when a detected face is too low-quality for face recognition to confidently match, restoring it before a second recognition attempt.

## Demo

| Before (low-res input) | After (CodeFormer restored) |
|---|---|
| ![Before](https://raw.githubusercontent.com/Morteza-Asadi-Shalmaiy/Superres-fr/refs/heads/main/assets/superres-test-image-04.jpg) | ![After](https://raw.githubusercontent.com/Morteza-Asadi-Shalmaiy/Superres-fr/refs/heads/main/assets/result.png) |

## How It Works

1. Face detected (by the face recognition module) but too low-resolution/distant to match confidently
2. CodeFormer restores and upsamples the face region (fidelity vs. quality controlled by the `-w` weight parameter)
3. Restored face is re-attempted against the recognition step

## Repo Structure

```
.
├── superres_fr.ipynb   # Full Colab notebook: setup, weights, inference, before/after comparison
├── assets/              # Demo before/after images
├── LICENSE
└── .gitignore
```

## Setup (Google Colab)

Run the notebook top to bottom. It handles:

1. **Environment setup** — installs `basicsr`/`facexlib`, with a compatibility shim for a broken torchvision import in `basicsr` (must run *before* `basicsr` is imported)
2. **Cloning CodeFormer** and installing its requirements
3. **Downloading model weights** — CodeFormer, face detection, face parsing, and Real-ESRGAN (background upsampling), with an automatic fallback to a mirror if the primary Real-ESRGAN download comes back corrupted
4. **A `torch.load` compatibility patch** — newer PyTorch defaults to `weights_only=True`, which breaks basicsr's checkpoint loader; this patches it safely (checks if already patched before touching the file)
5. **Inference** on a test image, with a before/after visual comparison

> **Note on CodeFormer itself:** unlike the anti-spoofing module, CodeFormer's weights (~400MB+ combined) are *not* vendored into this repo — that's too large for a normal Git repo without Git LFS. The notebook clones CodeFormer and downloads weights fresh each run instead, from the actively-maintained official repo.

## Environment Issues Solved

- `basicsr` importing a torchvision internal (`functional_tensor`) that no longer exists in current torchvision — fixed with a module shim
- `torch.load` defaulting to `weights_only=True` in newer PyTorch, breaking basicsr's Real-ESRGAN loader — patched in-place
- Unreliable Real-ESRGAN weight download from the official release URL — automatic corrupted-file detection and fallback to a mirror

## Tech Stack

`CodeFormer` · `basicsr` · `facexlib` · `Real-ESRGAN` · `PyTorch` · `OpenCV` · `Google Colab (GPU runtime)`
