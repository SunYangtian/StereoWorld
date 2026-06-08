# Stereo World Model: Camera-Guided Stereo Video Generation

[![Paper](https://img.shields.io/badge/arXiv-PDF-b31b1b)](https://arxiv.org/abs/2603.17375)
[![Website](imgs/badge-website.svg)](https://sunyangtian.github.io/StereoWorld-web/)

![teaser](./imgs/teaser.jpg)

StereoWorld generates stereo video from one input image and a text prompt. The camera trajectory is controlled by action tokens, and the output video is saved as a side-by-side left/right stereo MP4.

## Demo Gallery

![demo results](./imgs/demo_results.jpg)

## Features

- Camera-guided stereo video generation from a single RGB image.
- WASD-style camera controls for translation, yaw, and pitch.
- Side-by-side stereo output with configurable stereo baseline.
- Single-GPU inference and optional multi-GPU sequence-parallel inference.

## Installation

Create a Python environment and install the dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

The code is tested with CUDA 12.6 and PyTorch 2.4+. For faster attention, install `flash-attn` if it is compatible with your CUDA/PyTorch environment. Multi-GPU sequence parallel inference additionally requires `xfuser` and its dependencies.

## Model Weights

Download the model from Hugging Face:

```bash
huggingface-cli download Yang-Tian/StereoWorldModel \
  --local-dir weights/StereoWorldModel
```

The downloaded pipeline directory should contain:

```text
weights/StereoWorldModel/
  transformer/
  vae/
  tokenizer/
  text_encoder/
  scheduler/
```

You can keep the weights anywhere and pass the local model directory with `--pipeline_dir`.
When using this checkpoint, add `--use_raymap` during inference.

## TODO

- [x] Open-source the 5-second, 832x480 binocular teacher model.
- [ ] Release the few-step student model.
- [ ] Release more flexible multi-view world model.

## Quick Start

Run the included demo set on one GPU:

```bash
bash run_single.sh \
  --pipeline_dir weights/StereoWorldModel \
  --use_raymap
```

Run on multiple GPUs:

```bash
bash run.sh \
  --pipeline_dir weights/StereoWorldModel \
  --num_gpus 4 \
  --use_raymap
```

By default, the scripts use `ExpData/demo_custom_eval.json`, which contains 33 prompt/action examples. The corresponding input images are included under `ExpData/demo_custom/`.

## Custom Inference

Batch inference from an eval JSON:

```bash
python3 inference.py \
  --pipeline_dir weights/StereoWorldModel \
  --eval_json /path/to/eval.json \
  --output_dir output \
  --H 480 --W 832 \
  --num_frames 81 \
  --baseline 0.2
```

Single-folder inference:

```bash
python3 inference.py \
  --pipeline_dir weights/StereoWorldModel \
  --input_dir /path/to/sample_folder \
  --action_seq w wl wj \
```

For `--input_dir`, the folder should contain:

```text
sample_folder/
  left.png
  caption.txt
```

For `--eval_json`, each entry should contain:

```json
{
  "image_path": "./demo_custom/example.png",
  "caption": "A descriptive text prompt.",
  "action_seq": ["w", "wl", "wj"],
  "scene_name": "example_scene"
}
```

Relative `image_path` values are resolved relative to the JSON file. `scene_name` is optional; when omitted, the image filename is used.

## Camera Actions

| Key | Motion |
| --- | --- |
| `w` | move forward |
| `s` | move backward |
| `a` | move left |
| `d` | move right |
| `j` | yaw left |
| `l` | yaw right |
| `i` | pitch up |
| `k` | pitch down |

Actions can be combined in one segment, for example `wl` means moving forward while yawing right.

## Outputs

Each job writes:

- `{scene_name}.mp4`: side-by-side stereo video, left view on the left and right view on the right.
- `{scene_name}.json`: metadata including caption, action sequence, baseline, intrinsics, and camera poses.

`num_frames` must satisfy `1 + 4k`, for example `81` or `121`. Height and width must be divisible by 8.

## Acknowledgements

We thank the authors of the following projects and models for their open-source contributions:

- [prope](https://github.com/liruilong940607/prope)
- [DreamX-World](https://github.com/AMAP-ML/DreamX-World)
- [StereoCrafter](https://github.com/TencentARC/StereoCrafter)
- [Wan-AI](https://huggingface.co/Wan-AI)

## Citation

If you find our work useful, please consider citing:

```bibtex
@article{sun2026stereo,
  title={Stereo World Model: Camera-Guided Stereo Video Generation},
  author={Sun Yang-Tian and Huang Zehuan and Niu Yifan and Ma Lin and Cao Yan-Pei and Ma Yuewen and Qi Xiaojuan},
  journal={arXiv preprint arXiv:2603.17375},
  year={2026}
}
```
