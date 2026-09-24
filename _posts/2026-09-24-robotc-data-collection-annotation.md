---
title: RGB depth and thermal data collection and annotation tutorial
date: 2026-09-24
categories: [tutorial]
tags: [robotc]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---

## RGB Depth and Thermal Dataset Setup

### Overview of the dataset

This tutorial shows how to collect RGB, depth, and thermal images, then use
SAM3 to annotate the people and activities in the RGB images.

Each sample includes three images: an RGB image, a depth image aligned to RGB,
and a thermal image. The images share the same sample number.

Before starting,

1. Connect the RealSense and SenXor cameras.
2. Use the workspace at `/home/bob/lightsplat_ros1`.
3. Choose a new experiment name and decide which activities to record.

> The commands and options below were checked against the local source files
> on 2026-09-24. Collection and annotation are separate steps.
{: .prompt-info }

### Collect the data

#### 1. Prepare the cameras and environment

Connect the RealSense and SenXor cameras. Keep their relative mounting fixed
if you intend to reuse a thermal-to-RGB calibration.

Open a terminal, go to the workspace, and activate the existing ROS environment:

```bash
cd /home/bob/lightsplat_ros1
conda activate ros_env
source devel/setup.bash
rospack find slam
rospack find realsense2_camera
python -c 'import rospy, cv_bridge, cv2, numpy, senxor'
```

These checks must succeed before collection. This guide assumes ROS and the
camera dependencies are already installed; `requirements.txt` alone does
not install ROS. If this machine uses a different ROS environment, activate
that environment instead. These commands do not verify live camera access.

The launch file starts the RealSense driver, SenXor publisher, collector,
and (by default) RViz. Avoid starting a second copy of the camera drivers.

#### 2. Change the experiment name

In the same terminal, set the experiment name and save path:

```bash
export EXP_NAME=thermal_rgbd_new_office1
export DATASET="/home/bob/lightsplat_ros1/data/robotc_dataset/$EXP_NAME"
if [ -e "$DATASET" ]; then
  echo "Already exists: $DATASET — choose a new EXP_NAME for a new experiment."
else
  echo "New experiment path: $DATASET"
fi
```

> Change `thermal_rgbd_new_office1` according to your new experiment. For a test
> session, change `data/robotc_dataset` to `data/robotc_dataset_test`. Keep each
> independent experiment in its own folder.
{: .prompt-info }

The actual launch file is
`/home/bob/lightsplat_ros1/src/slam/launch/collect_thermal_rgbd.launch`.
Its current default `output_dir` ends in `thermal_rgbd_dough1`. Override it
with `output_dir:="$DATASET"` as below. Alternatively, edit that argument's
default in the launch file for the new experiment; there is no `exp_name`
launch argument.

> Make sure you change the experiment name before starting a new recording.
> An existing folder resumes collection: the recorder appends to `manifest.csv`
> after its highest sample index. It rewrites `dataset_info.json` but keeps an
> existing `camera_info.json`.
{: .prompt-warning }

#### 3. Start collecting images

In the same terminal, start the cameras and recorder:

```bash
roslaunch slam collect_thermal_rgbd.launch output_dir:="$DATASET"
```

You can change the collection settings with the following launch arguments:

| Argument | Default | Meaning |
|---|---|---|
| `output_dir` | `$(env HOME)/lightsplat_ros1/data/robotc_dataset/thermal_rgbd_dough1` | Experiment directory |
| `width`, `height` | `640`, `480` | Requested RealSense color/depth resolution |
| `fps` | `30` | Requested RealSense stream rate, not guaranteed saved sample rate |
| `save_rate` | `0.0` | No additional rate limit; positive values cap saving using thermal timestamps |
| `sync_slop` | `0.12` | Approximate synchronization tolerance in seconds |
| `queue_size` | `30` | Synchronizer queue size |
| `serial_no` | empty | Optional RealSense serial selection |
| `device_index` | `0` | SenXor index in detected serial devices |
| `rviz` | `true` | Start RViz |
| `rviz_software_rendering` | `true` | Use the launch file's software-rendering environment for RViz |

For example, to cap saving at approximately two sets per second and disable
RViz, use this **instead of** the command above:

```bash
roslaunch slam collect_thermal_rgbd.launch \
  output_dir:="$DATASET" save_rate:=2.0 rviz:=false
```

The collector uses approximate timestamp matching, not hardware
synchronization. SenXor timestamps are assigned with `rospy.Time.now()` when
the publisher creates the ROS message. RealSense depth is aligned to color;
the thermal stream is not geometrically registered by this launch file.

#### 4. Record the activities and stop collection

Look for `Saving synchronized thermal/RGB/depth data to ...` and periodic
`Saved ... synchronized samples` messages. In another terminal, activate the
same ROS environment and source `devel/setup.bash`, then check the default
topics if samples are not being saved:

```bash
rostopic hz /senxor/thermal/image_raw
rostopic hz /camera/color/image_raw
rostopic hz /camera/aligned_depth_to_color/image_raw
```

Run each command separately; stop it with Ctrl+C before running the next.
Adjust topic names if you changed the launch arguments. A SenXor connection
failure reports checks for USB connection, serial permissions, and
ModemManager in the publisher's error message.

Perform the activities planned for this session. As a collection practice,
record which activities actually occurred and the approximate sample/time
intervals in separate session notes; the collector does not record activity
labels. Include the scene conditions you need to evaluate, such as distance,
occlusion, reflections, and non-person warm objects.

Press **Ctrl+C in the roslaunch terminal** to stop recording. Expected layout:

```text
<experiment>/
├── rgb/00000000.png
├── depth/00000000.png
├── thermal/00000000.png
├── manifest.csv
├── dataset_info.json
└── camera_info.json
```

Filenames use the same eight-digit sample ID across modalities. RGB is saved
through OpenCV as 8-bit color; depth and thermal are uint16 PNGs. The launch
sets depth scale to `0.001` meters per unit. Thermal stores native ADC counts,
not degrees Celsius. `camera_info.json` is written after a color CameraInfo
message arrives and contains the RGB camera calibration, not thermal extrinsics.

#### 5. Check the saved images

After collection finishes, enter the following commands to check the saved files:

> Keep using the same terminal where you set `DATASET`. If you open a new
> terminal, set `EXP_NAME` and `DATASET` again before running the commands.
{: .prompt-info }

```bash
python - <<'PY'
import csv
import os
from pathlib import Path

p = Path(os.environ['DATASET'])
with (p / 'manifest.csv').open(newline='') as stream:
    rows = list(csv.DictReader(stream))
assert rows, 'No samples recorded'
ids = [row['sample'] for row in rows]
assert len(ids) == len(set(ids)), 'Duplicate manifest sample IDs'
for kind in ('rgb', 'depth', 'thermal'):
    expected = {row[f'{kind}_file'] for row in rows}
    actual = {str(f.relative_to(p)) for f in (p / kind).glob('*.png')}
    assert expected == actual, f'{kind}: missing or extra PNG files'
    print(f'{kind}: {len(actual)} images')
for name in ('dataset_info.json', 'camera_info.json'):
    assert (p / name).is_file(), f'Missing {name}'
print(f'Complete manifest sets: {len(rows)}')
PY
```

This checks file correspondence, not image quality. Inspect representative
frames before annotating. Interrupted or failed writes can leave files not
listed in the manifest; investigate a failed check before proceeding.

### Annotate the RGB images with SAM3

#### 1. Add the activity labels

Edit `DEFAULT_ACTIVITY_PROMPTS` near the top of
`/home/bob/lightsplat_ros1/scripts/annotate_rgb_sam3.py`, or provide the complete
desired list through `--prompts`. The current defaults are:

```python
DEFAULT_ACTIVITY_PROMPTS = [
    "a person sleeping",
    "a person reclining",
    "a seated person",
    "a person standing relaxed",
    "a person walking",
    "a seated person reading",
    "a person writing",
    "a person typing on a keyboard",
    "a seated person filing documents",
    "a standing person filing documents",
    "a person lifting or packing objects",
]
```

Add more activity labels according to your experiment. For example, if you
record the following activities, add these strings before the closing `]`:

```python
    "a person drinking from a cup",
    "a person eating",
    "a person using a phone",
    "a person carrying a box",
    "a person bending down",
```

These additions are proposed prompt examples, not claims that the existing
datasets contain these activities or that SAM3 will recognize them reliably.
Use one agreed list and order across sessions: category IDs are the prompt
positions plus one. Appending preserves earlier IDs; reordering changes them.

> If you use `--prompts`, include the complete list of labels you want. This
> option replaces the default list. For example, `--prompts "a person walking"
> "a person using a phone"` creates only those two categories. Leave out
> `--prompts` to use the list saved in the script.
{: .prompt-info }

SAM3 produces text-prompted mask predictions from individual RGB images.
The category is the matched prompt, not a verified activity annotation.
Similar prompts can match the same person. There is no sequence-level
activity recognition or manual annotation editor in this script.

#### 2. Check SAM3 and annotate a few images

Use the existing environment with PyTorch, Ultralytics SAM3, NumPy, and OpenCV:

```bash
cd /home/bob/lightsplat_ros1
conda activate ros_env
python - <<'PY'
from pathlib import Path
import torch
from ultralytics.models.sam import SAM3SemanticPredictor
assert Path('/home/bob/lightsplat_ros1/models/sam3.pt').is_file()
print('SAM3 import/checkpoint checks passed; CUDA available:', torch.cuda.is_available())
PY
```

The local `models/sam3.pt` exists at the time this guide was written. If using
another checkpoint location, pass it with `--model`. See the existing
[SAM3 setup notes](scripts/DEPTH_THERMAL_README.md) for environment setup.

> Set `--dataset` to one experiment folder containing `rgb/`. The current
> script default contains the path typo `lightplat_ros1` and points at a
> collection parent. `--recursive` only searches inside the selected RGB
> directory; it does not process all experiment folders automatically.
{: .prompt-warning }

After setting the activity labels, annotate the first ten images:

```bash
python scripts/annotate_rgb_sam3.py \
  --dataset "$DATASET" \
  --model /home/bob/lightsplat_ros1/models/sam3.pt \
  --output-dir "$DATASET/sam3_preview" \
  --limit 10 \
  --conf 0.3 --iou 0.7 --imgsz 1008 \
  --max-instances 1 --prompt-batch-size 1
```

Inspect `sam3_preview/overlays/` for mask quality, and inspect
`sam3_preview/annotations.json` for category names, predicted categories, and
scores. The overlay image does not print activity label text. `--limit 10`
selects the first ten sorted images, not a random sample.

The default `--max-instances 1` keeps the highest-confidence instance across
all prompts for each frame. For multiple people, raise this limit or use `0`
to retain all predictions. The script does not deduplicate masks across
separate prompt batches, so retaining all can keep overlapping activity
predictions for the same person.

Device selection defaults to CUDA `0` when available, otherwise CPU. Use
`--device cpu` to request CPU explicitly. If memory is insufficient, try a
smaller `--imgsz`, such as `640`, and keep `--prompt-batch-size 1`; inspect
the resulting mask quality again.

#### 3. Annotate all images in the experiment

After checking the preview and completing the label list, run the annotation
script again without `--limit`:

```bash
python scripts/annotate_rgb_sam3.py \
  --dataset "$DATASET" \
  --model /home/bob/lightsplat_ros1/models/sam3.pt \
  --conf 0.3 --iou 0.7 --imgsz 1008 \
  --max-instances 1 --prompt-batch-size 1
```

Use your reviewed settings if they differ from this example. The default
output is `$DATASET/sam3_annotations`:

| Output | Contents |
|---|---|
| `annotations.json` | COCO-style image/category records, masks as uncompressed RLE, boxes, scores, and mask paths |
| `masks/<sample>/` | Binary PNG for each saved instance, values 0 and 255 |
| `instance_maps/<sample>.png` | uint16 local instance IDs; 0 is background |
| `semantic_maps/<sample>.png` | uint16 category IDs; 0 is background |
| `overlays/<sample>.jpg` | RGB images with colored masks and outlines |

Images with no detections still receive zero-valued maps and an overlay.
For overlapping predictions, later, higher-confidence instances overwrite
earlier pixels in the maps. Individual mask files retain their full masks.

> Use a new `--output-dir` to compare different label sets. If you intend to
> replace existing annotations, add `--overwrite`. This does not remove old
> files that are no longer used. Wait for the run to finish: `annotations.json`
> is written at the end, and an interrupted run can leave partial outputs.
{: .prompt-warning }

#### 4. Check the annotation results

```bash
python - <<'PY'
import json
import os
from collections import Counter
from pathlib import Path

p = Path(os.environ['DATASET'])
out = p / 'sam3_annotations'
a = json.loads((out / 'annotations.json').read_text())
expected = {f'rgb/{f.name}' for f in (p / 'rgb').glob('*.png')}
actual = {image['file_name'] for image in a['images']}
assert actual == expected, 'Annotation image coverage differs from RGB files'
for image in a['images']:
    key = Path(image['file_name']).stem
    for folder in ('semantic_maps', 'instance_maps'):
        assert (out / folder / f'{key}.png').is_file()
for annotation in a['annotations']:
    assert (out / annotation['mask_file']).is_file()
counts = Counter(x['category_id'] for x in a['annotations'])
for category in a['categories']:
    print(category['id'], category['name'], counts[category['id']])
detected = {x['image_id'] for x in a['annotations']}
print('Images:', len(a['images']), 'Images without predictions:', len(a['images']) - len(detected))
PY
```

Review examples from every intended activity and frames without predictions.
Compare predictions with the RGB frames and session notes; a confidence score
does not establish that an activity is correct. Correct or exclude erroneous
annotations using your chosen review workflow; this batch script has no
interactive correction mode. Save the reviewed prompt list and command with
the session notes.

### View the depth and thermal images

After annotation finishes, convert the depth and thermal images into color
visualizations and add the SAM3 masks to the depth images:

```bash
python scripts/visualize_depth_thermal.py "$DATASET" --depth-mask-overlay
```

Outputs remain inside the experiment:

```text
visualizations/
├── depth/                # Grayscale; nearer brighter, invalid pixels dark
├── thermal/              # Inferno heatmap
└── depth_mask_overlay/   # Blue SAM3 foreground overlay on grayscale depth
```

The depth overlay combines all positive semantic categories into a single
blue foreground mask; it does not distinguish activities by color. Masks
must match the aligned depth dimensions. The exporter reads masks from
`sam3_annotations/semantic_maps`, not a custom annotation output directory.
Existing visualizations are skipped; add `--overwrite` to refresh them after
changing annotations. Thermal colors use per-frame percentiles by default
and do not represent calibrated temperatures. Preserve the original uint16
images for numerical processing. RGB masks cannot be transferred directly
to thermal coordinates without a separate registration step.

## Reference

- Collection launch: [collect_thermal_rgbd.launch](src/slam/launch/collect_thermal_rgbd.launch)
- Recorder: [sensor_data_collector.py](src/slam/scripts/sensor_data_collector.py)
- Thermal publisher: [senxor_thermal_publisher.py](src/slam/scripts/senxor_thermal_publisher.py)
- Annotation: [annotate_rgb_sam3.py](scripts/annotate_rgb_sam3.py)
- Visualization: [visualize_depth_thermal.py](scripts/visualize_depth_thermal.py)
