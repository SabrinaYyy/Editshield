# EditShield Repo Guidance

This file is the standing context for Claude Code and other coding agents working in
this repo. Read it before editing files.

## Project Goal
Build a local portrait-protection tool for non-technical users. A user drops in a
portrait and receives a visually unchanged protected image to post instead of the
original. By default, images stay on the user's own machine.

## Threat Model
We commit to two threats:

1. Malicious editing / nudify via img2img or inpainting, often instruction-guided diffusion.
2. Face swap / identity transfer.

Do not overclaim. We do not claim protection against unknown future tools,
re-photographed images, re-upload/compression pipelines, closed models, or
non-diffusion systems unless explicitly tested.

## Protector Direction
Use one protector with two strength modes:

- Default / PhotoGuard-style: VAE encoder attack toward a target latent, prioritizing invisibility.
- Strong / EditShield-style: maximize latent inconsistency from the original, with cheap EoT transforms for robustness.

Both modes should eventually include an ArcFace identity-shift term for face-swap
defense. A fast/strong slider maps to mode, l_inf budget, and step count.

When implementing protection code, preserve these priorities:

- The protected image should remain visually close to the input portrait.
- Default mode should favor imperceptibility and CPU/MPS-friendly runtime.
- Strong mode may be more visible but should document the tradeoff.
- Any face-swap claim needs identity metrics, not only CLIP/image-editing metrics.

## Repo Shape
Important files:

- `50_sample_of_EditShield_Colab (1).ipynb`: original Colab reproduction notebook.
- `Functions.py`: shared VAE/CLIP helper code from the reproduction.
- `EOT_Center.py`, `EOT_Gaussion.py`, `EOT_Resize.py`: original EOT attack variants.
- `Cap_EOT_C.py`, `Cap_EOT_G.py`, `Cap_EOT_R.py`, `Caption_Metrics.py`: caption and metric scripts.
- `environment.yaml`, `download_checkpoints.sh`: setup/checkpoint helpers.

Large generated artifacts, model checkpoints, Colab outputs, zip files, and image
datasets should not be committed to this repo unless explicitly requested.

## Notebook Rule
Never modify the original notebook:

`50_sample_of_EditShield_Colab (1).ipynb`

For experiments, create a clearly named copy first, such as:

`50_sample_of_EditShield_LFW_Faces.ipynb`

Make all experimental changes in the copy. If asked to update an experiment, edit
the copied experiment notebook only unless the user explicitly says to edit the
original.

## Current Colab Experiment
The existing notebook reproduces EditShield on 50 MagicBrush samples. For
facial-image testing, use LFW images from:

`/content/drive/MyDrive/fawkes/data/lfw`

Create the same expected data layout:

```text
train_data/
  <sample_id>/
    <sample_id>_0.jpg
    prompt.json
```

Use reproducible random sampling with seed `33`.

Recommended face-test config names:

```python
USE_LFW = True
LFW_ROOT = "/content/drive/MyDrive/fawkes/data/lfw"
N_FACE_SAMPLES = 50
RANDOM_SEED = 33
```

For LFW setup cells:

- Recursively find `.jpg`, `.jpeg`, and `.png` files under `LFW_ROOT`.
- Skip unreadable/corrupt files.
- Convert images to RGB.
- Write exactly `N_FACE_SAMPLES` valid samples when enough files exist.
- Use benign face-edit prompts, for example "make the person smile", "change the
  hair color", "make the person look older", "add dramatic makeup", and "turn the
  portrait into a studio headshot".
- Keep LFW outputs separate from MagicBrush outputs, for example under
  `OUT_DIR/faces_lfw_50`.
- Add resume behavior where possible: skip work when expected output files and
  result rows already exist.

Note: if the MagicBrush preparation cell is touched in a copied notebook, check
the emptiness guard. The current reproduction intent is "download only when
`DATA_DIR` is empty"; do not accidentally overwrite an existing dataset.

## Evaluation Notes
Random LFW portraits are acceptable for testing:

- visual quality
- whether instruction-guided edits are disrupted
- CLIP image similarity before/after protection

Random single LFW portraits are not sufficient for face-swap claims. Face-swap
evaluation needs identity-aware pairs or multiple images per identity, plus
ArcFace or face-recognition metrics.

Use metrics carefully:

- Editing/nudify robustness: compare edited original vs edited protected output,
  visual inspection, CLIP image similarity, and direction similarity if captions
  are available.
- Visual quality: compare original vs protected image with CLIP similarity and,
  when added, LPIPS/SSIM/PSNR.
- Face-swap robustness: use ArcFace/face-recognition embedding similarity across
  source, protected source, target, and swapped outputs. Do not rely on CLIP alone.

## Output Rule
Save outputs persistently to Google Drive, not only Colab cache. Include:

- original images
- protected images
- edited originals
- edited protected images
- CSV metrics
- zip archive

Prefer explicit output paths and printed summaries at the end of notebooks so the
user can find results after a Colab runtime disconnect.

## Coding Guidance
Keep changes small and traceable:

- Preserve existing reproduction behavior unless the user asks to change it.
- Prefer adding new cells/functions over rewriting large notebook sections.
- Keep paths configurable at the top of the notebook or script.
- Use deterministic seeds for sampling and generation when comparing methods.
- Do not add claims to docs or UI unless the repo has tests or experiments
  supporting them.
