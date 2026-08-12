# Signature Extraction and Manual-Signing Indicator Analysis

![](https://github.com/debabratapruseth/Hybrid-Signature-Forensics/blob/main/Signaturre%20Detection%20%26%20Manual%20Sign%20Verification%20.png)

This Google Colab notebook detects signatures in document images, extracts them, and measures static-image characteristics that may be consistent with manual signing or digital reproduction.

The project does **not** verify the identity of a signer and does not declare a signature genuine or forged. Its output is intended to support human review.

## Problem statement

Given an image or a selected PDF page, the notebook should:

1. Locate signature-like regions using a signature-trained YOLO model.
2. Allow a reviewer to accept the correct detection candidates.
3. Segment accepted signatures using the Segment Anything Model (SAM).
4. Clean the extracted signatures conservatively using OpenCV.
5. Measure ink variation, stroke-width variation, texture, curvature, and continuity.
6. Report cautious manual-signing and reproduction indicators with quality warnings.
7. Store all inputs, models, intermediate artifacts, and reports persistently in Google Drive.

## Input Output (Sample)

1) Input Image of a scanned Form with Signature.

![](https://github.com/debabratapruseth/Signature-Extraction-and-Manual-Signing-Indicator-Analysis/blob/main/Reference%20Material/Signed_Input.png)

3) Signature identified and marked with bounding box and confidence score.

![](https://github.com/debabratapruseth/Signature-Extraction-and-Manual-Signing-Indicator-Analysis/blob/main/Reference%20Material/Signature%20Identified%20with%20Confidence%20Score.png)

5) Signature segmented and extracted. In this example the signature with highest confidence score is extracted. The segmentation also removes any marks etc. overriding the signature. This helps to extract a clean signature for forensic.

![](https://github.com/debabratapruseth/Signature-Extraction-and-Manual-Signing-Indicator-Analysis/blob/main/Reference%20Material/Signarure%20Extraction%20and%20Segmentation.png)

7) Signature forensic covering manual signature patterns. Note as we are analyzing a image and not actual paper with sign, the forensics are proxies for actual review.

![](https://github.com/debabratapruseth/Signature-Extraction-and-Manual-Signing-Indicator-Analysis/blob/main/Reference%20Material/Manual%20Sign%20Forensics.png)

## Architecture

```text
Image or PDF page
        |
        v
Document decoding and quality checks
        |
        v
Tech4Humans YOLO signature detection
        |
        v
Human candidate-selection gate
        |
        v
SAM box-prompted segmentation
        |
        v
Conservative OpenCV cleaning
        |
        v
Static-image feature extraction
        |
        v
Quality-gated indicator interpretation
        |
        v
JSON, images, Markdown report, and checksum manifest
```

## Models

### Signature detection

The notebook uses the gated [Tech4Humans YOLOv8 Signature Detector](https://huggingface.co/tech4humans/yolov8s-signature-detector).

- Repository: `tech4humans/yolov8s-signature-detector`
- Checkpoint: `yolov8s.pt`
- Expected class: `signature`
- Purpose: locate signature-like regions only

Access must be approved on Hugging Face. The notebook requests a read token through a hidden prompt when the model is first downloaded. The checkpoint is then stored in Google Drive for reuse.

### Segmentation

The notebook uses [SAM ViT-Base](https://huggingface.co/facebook/sam-vit-base):

- Model: `facebook/sam-vit-base`
- Prompt: reviewer-accepted YOLO bounding boxes
- Purpose: separate the signature candidate from surrounding document content

SAM is a general segmentation model and is not trained to determine signature authenticity.

## Datasets

No signature-verification dataset is used in the current pipeline.

BHSig260 and CEDAR were considered but intentionally excluded because they primarily contain genuine and forged handwritten signatures for writer verification or forgery research. This project does not compare a questioned signature with reference signatures and does not perform writer verification.

A future learned classifier for wet-ink originals versus printed, scanned, stamped, or digitally pasted signatures would require a separate, representative dataset with those acquisition and reproduction classes explicitly labeled.

## Processing stages

The notebook is organized as sequential, annotated Colab cells:

1. Mount Google Drive and create persistent folders.
2. Install and verify dependencies.
3. Define configuration and create a unique run directory.
4. Upload one questioned document.
5. Decode the image or PDF page and assess acquisition quality.
6. Download, validate, and load the Tech4Humans YOLO model.
7. Detect and save signature candidates.
8. Review and explicitly accept the correct candidates.
9. Load SAM from a persistent Google Drive cache.
10. Segment accepted candidates and assess mask quality.
11. Clean signatures with OpenCV and roll back destructive cleaning.
12. Extract static-image measurements.
13. Produce quality-gated manual-signing indicators.
14. Generate the final report and artifact checksum manifest.

## Measured indicators

The analysis includes:

- Ink darkness and local intensity variation
- Estimated stroke-width variation
- Width–darkness correlation
- Intensity entropy and high-frequency texture proxies
- Contour turning-angle and curvature proxies
- Skeleton endpoints and junctions
- Connected-component and fragmentation measurements
- Uniformity-based digital-reproduction warnings

These are measurements from a raster image. They are not direct measurements of physical pen pressure, speed, direction, or stroke order.

## Possible assessments

The notebook restricts its output to:

- `manual_signing_indicators_present`
- `possible_reproduction_indicators`
- `inconclusive`

Quality gates can force an `inconclusive` result even when the manual-indicator score is high. For example, an incomplete SAM mask or destructive cleaning can invalidate downstream measurements.

Numeric scores are uncalibrated engineering indicators, **not probabilities**.

## Assumptions

- The user has authorization to process the uploaded documents.
- The selected page contains at least one visible signature.
- Dark ink appears on a comparatively light background.
- The Tech4Humans checkpoint contains a class named `signature`.
- The reviewer manually confirms the correct YOLO candidates.
- Fine texture measurements require a sufficiently sharp, high-resolution image.
- Google Drive is available and has enough space for model checkpoints and outputs.
- Model and package licenses permit the intended use; users must verify this before redistribution or commercial deployment.

## Running the notebook

1. Open [`Signature_Analysis_Colab.ipynb`](Signature_Analysis_Colab.ipynb) in Google Colab.
2. Select a GPU runtime when available.
3. Run every cell in order.
4. Authorize Google Drive access.
5. Upload exactly one supported document when prompted.
6. Provide an approved Hugging Face read token when the gated YOLO model is first downloaded.
7. Inspect the numbered YOLO crops and edit `ACCEPTED_DETECTION_NUMBERS` before continuing.
8. Review the final report and the intermediate segmentation and cleaning artifacts.

Supported inputs include PDF, PNG, JPEG, TIFF, and BMP. PDF page numbering begins at zero.


## Limitations and responsible use

A static scan cannot establish whether a pen physically touched the examined sheet. Scanner enhancement, image compression, printing, photocopying, paper texture, resolution, and segmentation errors can all affect the measured features.

The notebook must not be used by itself to determine:

- Signer identity
- Signature genuineness
- Forgery
- Intent or fraud
- Legal validity

For consequential decisions, inspect the original document and consider magnification, oblique lighting, indentation examination, high-resolution lossless scanning, PDF object inspection, duplicate-image screening across documents, and review by a qualified forensic document examiner.

## Status

This repository is a research and educational prototype. The manual-signing thresholds have not been calibrated on a labeled wet-ink-versus-reproduction dataset.
