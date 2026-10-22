# MICCAI 2023 SurgToolLoc: Team ANL-Surg

Code, Docker submission container, and data pointers from **Team ANL-Surg** (Argonne National Laboratory) for the [MICCAI 2023 Surgical Tool Localization (SurgToolLoc) challenge](https://surgtoolloc23.grand-challenge.org/), part of the MICCAI Endoscopic Vision Challenge.

**Result:** Team ANL-Surg placed **third** in the 2023 SurgToolLoc challenge.

**Team:** Neil Getty and Ruoxi Zhao, with Team ANL-Surg at Argonne National Laboratory.

## Task

Given frames of robotic-surgery endoscopic video, detect and localize the surgical tools in each frame with bounding boxes and confidence scores. The challenge uses tool-presence labels at the video level, so training a detector requires building frame-level box annotations.

## Approach

- **Annotation with foundation models.** We used the Segment Anything Model (SAM) with text and box prompts to localize tools and generate bounding-box annotations, then curated them manually.
- **Detection.** We trained a YOLOv8 detector (Ultralytics) on the resulting COCO-format annotations.
- **Inference.** The submission container runs the trained YOLOv8 weights frame by frame (input size 512 x 640, IoU 0.4, confidence 0.25, up to 3 detections per frame) and writes the Grand Challenge JSON format.

## Repository layout

| Path | Contents |
|---|---|
| `docker/` | Grand Challenge algorithm container: `Dockerfile`, `process.py` (inference), `build.sh`, `export.sh`, `test.sh`, `requirements.txt`. See `docker/README.md` for the output format. |
| `surgtoolloc2022-category-2` | Git submodule pointing to the challenge's Category 2 submission template. |

## Data and weights

- Manual annotations used for training: [download (Argonne Box)](https://anl.box.com/s/yes55hjivp43x7iweu3nijsdmkehc9k4)
- Trained weights: [download (Argonne Box)](https://anl.box.com/s/den7qrdzvj709ea0d4pxu3evmtnd6ze9). Place `best.pt` in `docker/weights/` before building; `process.py` loads it from `/opt/algorithm/weights/best.pt` inside the container.

## Build and export the container

```bash
cd docker
sudo ./build.sh     # builds the surgtoolloc_det image
sudo ./export.sh    # builds and saves surgtoolloc_det.tar.gz for upload
sudo ./test.sh      # optional local test run
```

## Citation

Challenge results paper, which includes this team's entry:

> A. Zia, M. Berniker, et al. *Intuitive Surgical SurgToolLoc and SurgVU Challenges Results: 2022–2025.* [arXiv:2305.07152](https://arxiv.org/abs/2305.07152)

```bibtex
@article{zia2023surgtoolloc,
  title   = {Intuitive Surgical SurgToolLoc and SurgVU Challenges Results},
  author  = {Zia, Aneeq and Berniker, Max and others},
  journal = {arXiv preprint arXiv:2305.07152},
  year    = {2023}
}
```

## Acknowledgments

This work was carried out at Argonne National Laboratory during the 2023 summer research program.
