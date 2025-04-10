# NeRF Training: Style-Aware 3D Reconstruction of the Minaret

This document describes the training pipeline for NeRF-based 3D reconstruction of the Great Mosque of Aleppo's minaret using both original and style-transferred image datasets.

---

## Overview

To achieve high-fidelity geometry reconstruction with rich surface textures, we trained a NeRF model (specifically `nerfacto`) on:

1. **The original primitive minaret dataset**
2. **A style-transferred version** using Arabic decorative motifs applied via Stable Diffusion

We utilized COLMAP for camera pose estimation and sparse point cloud generation to provide geometric priors and reduce alignment errors during training.

---

## 1. Dataset Preparation

We selected 10 architectural elements from the mosque, focusing on the minaret. For each element, over 100 multi-view images were captured from different angles:

- Horizontal view
- +20° Upward angle
- -20° Downward angle

The minaret dataset (original and style-transferred versions) is formatted into `images/`, with accompanying `transforms.json` or COLMAP-derived pose data.

**Input Example:**

/minaret_dataset/
├── images/
│   ├── img001.jpg
│   ├── img002.jpg
│   └── …
└── colmap/ (or transforms.json)

---

## 2. COLMAP Processing

To obtain accurate camera poses and initial geometry:

1. Run COLMAP feature extraction and matching:
    ```bash
    colmap feature_extractor --database_path database.db --image_path images/
    colmap exhaustive_matcher --database_path database.db
    ```

2. Run sparse reconstruction:
    ```bash
    mkdir sparse/
    colmap mapper --database_path database.db --image_path images/ --output_path sparse/
    ```

3. Export to NeRF-compatible format (use `colmap2nerf.py` or tools like `nerfstudio`)

4. Visualize sparse point cloud (optional)

---

## 3. Model Configuration

We adopted `nerfacto`, an improved NeRF model suitable for static scenes. Key advantages:

- Faster convergence
- Higher visual fidelity
- Integration-friendly with COLMAP geometry priors

### Configuration Parameters

| Parameter        | Value           |
|------------------|-----------------|
| `near`           | 0.1             |
| `far`            | 10.0            |
| `sample count`   | 64 rays/sample  |
| `max steps`      | 30,000 iterations |
| `device`         | NVIDIA RTX 4090 |

These values ensure the model focuses on the correct scene scale and retains high-frequency geometric features.

---

## 4. Training Steps

1. Launch training:
    ```bash
    nerfstudio train --data /path/to/minaret_dataset --pipeline nerfacto
    ```

2. Monitor training with viewer or tensorboard

3. Optional: Export intermediate mesh or point cloud for validation

---

## 5. Output and Export

After 30k iterations, the model produces high-fidelity reconstructions:

- Format: `.glb` / `.gltf`
- Export tool (e.g., `nerfstudio export`, `convert_nerf_to_mesh.py`)

Example output:

/outputs/
├── mesh.glb
└── texture_map.png

---

## 6. Notes and Tips

- Always verify that COLMAP poses are correct. Misaligned camera data severely impacts NeRF training.
- Style-transferred datasets can introduce noise—use appropriate prompts like:
  - Positive: `"intricate stone texture", "realistic arabic carving"`
  - Negative: `"no distortion", "preserve structure"`
- Use `init_image_strength=1.0` and `sampling_steps=50` for best style fusion.

---

## References

- COLMAP: https://colmap.github.io/
- Nerfacto / NeRF Studio: https://docs.nerf.studio/
- Stable Diffusion Integration: ControlNet + Reference Preprocessor