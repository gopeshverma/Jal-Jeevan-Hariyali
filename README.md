# 💧 Jal Jeevan Water Resource Detection

### Water Resource & Irrigation Infrastructure Detection using YOLOv8

An AI-powered **object detection system** built with **YOLOv8s** for automatically identifying rural water resources and irrigation infrastructure from geo-tagged field images.

The system detects **handpumps, wells, lakes, and irrigation canals** and provides bounding boxes, class labels, and confidence scores for detected objects. A **Flask REST API** is included to serve the trained model and perform image inference using base64-encoded images.

This project was developed as part of the **Jal Jeevan Hariyali initiative** to support automated mapping and identification of rural water-resource assets.

---

## 📋 Table of Contents

* [Overview](#overview)
* [Key Features](#key-features)
* [Object Classes](#object-classes)
* [Dataset](#dataset)
* [Model and Methodology](#model-and-methodology)
* [Training Pipeline](#training-pipeline)
* [Evaluation](#evaluation)
* [Sample Predictions](#sample-predictions)
* [Project Structure](#project-structure)
* [Installation](#installation)
* [Running the API](#running-the-api)
* [API Usage](#api-usage)
* [Tech Stack](#tech-stack)
* [Future Improvements](#future-improvements)
* [Acknowledgements](#acknowledgements)

---

## 🔎 Overview

Field surveys conducted under the **Jal Jeevan Hariyali scheme** generate a large number of geo-tagged photographs containing rural water infrastructure.

Manually identifying and locating assets such as **handpumps, wells, lakes, and irrigation canals** in these images can be time-consuming and error-prone.

This project automates the process using a fine-tuned **YOLOv8s object detection model**.

### Input

A field image containing one or more water-resource assets.

### Output

The model returns:

* Detected object class
* Confidence score
* Bounding-box coordinates
* Annotated image with detected objects

### Overall Workflow

```text
Geo-tagged Field Image
          │
          ▼
    Image Preprocessing
          │
          ▼
      YOLOv8s Model
          │
          ▼
    Object Detection
          │
          ├── Class Label
          ├── Confidence Score
          └── Bounding Box
          │
          ▼
      Detection Result
          │
          ▼
       Flask REST API
```

---

## ✨ Key Features

* 🤖 YOLOv8s-based object detection
* 💧 Detection of rural water resources
* 🌾 Detection of irrigation infrastructure
* 📍 Designed for geo-tagged field imagery
* 🎯 Bounding-box based object localization
* 📊 Confidence scores for predictions
* 🧠 Fine-tuned on a custom dataset
* 🔄 Pascal VOC to YOLO annotation conversion
* 📈 Comprehensive model evaluation
* 🌐 Flask REST API for model inference
* 🖼️ Base64 image input/output support
* 📊 Confusion matrix and precision-recall analysis
* 📁 Sample prediction visualizations

---

## 🎯 Object Classes

The trained model detects five classes:

| Class        | Description                        |
| ------------ | ---------------------------------- |
| **handpump** | Hand-operated water pumps          |
| **well**     | Open or covered wells              |
| **lake**     | Natural or man-made water bodies   |
| **pine**     | Irrigation canals containing water |
| **pine_dry** | Irrigation canals without water    |

### Why Two Canal Classes?

Irrigation canals were divided into two separate classes:

* `pine` — water-filled canal
* `pine_dry` — dry canal

A single canal class was not able to reliably distinguish between water-filled and dry canals. Separating the two classes improved the model's ability to identify these visually different conditions.

---

## 📊 Dataset

The dataset consists of **geo-tagged field images** collected under the Jal Jeevan Hariyali scheme.

The original annotations were provided in **Pascal VOC XML format** and were converted into YOLO-compatible annotation files using the dataset preparation pipeline.

### Dataset Download

Due to the size of the dataset, the raw and processed datasets are hosted externally:

**[Download Dataset from Google Drive](https://drive.google.com/drive/folders/17Bkbxz6ZCxlqWoc15eE6a6gFGHr06DuV?usp=sharing)**

The dataset repository contains:

```text
dataset/
├── images/
├── labels/
└── train/test splits

raw_dataset/
├── images/
└── Pascal VOC XML annotations
```

---

# 🧠 Model and Methodology

## Base Model

The project uses **YOLOv8s**, the small variant of the YOLOv8 object detection architecture.

The model was initialized using **COCO-pretrained weights** and fine-tuned on the custom Jal Jeevan Hariyali dataset.

## Training Configuration

| Parameter     | Value     |
| ------------- | --------- |
| Model         | YOLOv8s   |
| Epochs        | 100       |
| Batch Size    | 4         |
| Image Size    | 640 × 640 |
| Device        | CPU       |
| Optimizer     | SGD       |
| Learning Rate | 1e-3      |
| Momentum      | 0.937     |
| Weight Decay  | 5e-4      |
| Warmup        | 3 epochs  |

---

## 🔄 Data Augmentation

Several augmentation techniques were used during training to improve model generalization:

* Mosaic: `1.0`
* Mixup: `0.1`
* Rotation: `±10°`
* Vertical Flip: `0.3`
* Horizontal Flip: `0.5`
* HSV Jittering

These augmentations help the model become more robust to variations in field images, object orientation, lighting, and environmental conditions.

---

# ⚙️ Training Pipeline

The project uses a three-stage sequential pipeline.

### Step 1 — Dataset Preparation

```bash
python scripts/step1_prepare_dataset.py
```

This script:

* Reads Pascal VOC XML annotations
* Extracts bounding-box coordinates
* Converts annotations to YOLO format
* Normalizes bounding-box coordinates
* Organizes training and testing data
* Generates `data.yaml`

### Step 2 — Model Training

```bash
python scripts/step2_train.py
```

This script fine-tunes YOLOv8s using the prepared dataset and configured training parameters.

### Step 3 — Model Evaluation

```bash
python scripts/step3_evaluate.py
```

The evaluation pipeline generates:

* Test-set metrics
* Per-class AP
* Confusion matrix
* Precision-recall curves
* F1 curves
* Sample prediction images
* Training curves

---

# 📈 Evaluation

The final **YOLOv8s** model was evaluated on the held-out test set.

## Overall Performance

| Metric        |      Score |
| ------------- | ---------: |
| **mAP@50**    | **0.8548** |
| **mAP@50-95** | **0.4284** |
| **Precision** | **0.8576** |
| **Recall**    | **0.8737** |

These results indicate that the trained model is capable of detecting the targeted water-resource and irrigation assets with good precision and recall.

---

## Per-Class Performance

| Class    |  AP@50 | AP@50-95 |
| -------- | -----: | -------: |
| handpump | 0.8045 |   0.3583 |
| lake     | 0.9950 |   0.6546 |
| pine     | 0.9950 |   0.5377 |
| pine_dry | 0.6021 |   0.2031 |
| well     | 0.8775 |   0.3885 |

### Performance Analysis

The `lake` and `pine` classes achieve very high AP@50 scores.

The `pine_dry` class has the lowest performance, with an AP@50 of approximately **0.60**. This may be attributed to the visual similarity between dry irrigation canals and surrounding terrain.

---

## 📊 Confusion Matrix

![Confusion Matrix](runs/jjh_yolov8s/test_eval/metrics/confusion_matrix_normalized.png)

---

## 📈 Precision-Recall Curve

![Precision Recall Curve](runs/jjh_yolov8s/test_eval/metrics/BoxPR_curve.png)

---

## 📉 Training Curves

![Training Curves](runs/jjh_yolov8s/test_eval/training_curves.png)

---

## 📊 Per-Class AP

![Per-Class AP](runs/jjh_yolov8s/test_eval/test_per_class_ap.png)

---

# 🖼️ Sample Predictions

Example detection results from the test dataset:

| Handpump                                                              | Well                                                          | Lake                                                          |
| --------------------------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------- |
| ![Handpump](runs/jjh_yolov8s/test_eval/all_predictions/Handpump1.jpg) | ![Well](runs/jjh_yolov8s/test_eval/all_predictions/Well5.jpg) | ![Lake](runs/jjh_yolov8s/test_eval/all_predictions/Lake1.jpg) |

Additional prediction results can be found in:

```text
runs/jjh_yolov8s/test_eval/all_predictions/
```

---

# 📁 Project Structure

```text
jal-jeevan-water-resource-detection/
│
├── api/
│   ├── app.py
│   └── encode_image.py
│
├── scripts/
│   ├── step1_prepare_dataset.py
│   ├── step2_train.py
│   └── step3_evaluate.py
│
├── runs/
│   └── jjh_yolov8s/
│       ├── weights/
│       │   └── best.pt
│       │
│       └── test_eval/
│           ├── all_predictions/
│           ├── metrics/
│           ├── test_per_class_ap.csv
│           ├── test_per_class_ap.png
│           └── training_curves.png
│
├── data.yaml
├── requirements.txt
├── .gitignore
└── README.md
```

---

# 🚀 Installation

## Prerequisites

Make sure the following are installed:

* Python **3.10 or higher**
* Git
* pip
* Virtual environment support

## Clone the Repository

```bash
git clone https://github.com/Yuvraj428/jal-jeevan-water-resource-detection.git
```

Navigate into the project:

```bash
cd jal-jeevan-water-resource-detection
```

## Create Virtual Environment

### Windows

```bash
python -m venv venv
venv\Scripts\activate
```

### Linux / macOS

```bash
python -m venv venv
source venv/bin/activate
```

## Install Dependencies

```bash
pip install -r requirements.txt
```

---

# 🌐 Running the API

The trained YOLOv8s model is served through a **Flask REST API**.

Navigate to the API directory:

```bash
cd api
```

Start the Flask server:

```bash
python app.py
```

The API will be available at:

```text
http://localhost:5000
```

---

# 🔌 API Endpoints

## `GET /`

Health-check endpoint used to verify that the API server is running.

---

## `POST /predict`

Accepts a base64-encoded image and returns object-detection results.

### Request

```json
{
  "image": "<base64-encoded-image-string>",
  "image_id": "test_001"
}
```

### Response

```json
{
  "image_id": "test_001",
  "predictions": [
    {
      "class": "handpump",
      "confidence": 0.9012
    }
  ],
  "annotated_image": "<base64-encoded-annotated-image>"
}
```

The response contains:

* Image ID
* Detected class
* Confidence score
* Annotated image

---

# 🧪 Testing the API

A utility script is provided to convert an image into a base64 string.

```bash
cd api
python encode_image.py path/to/your/image.jpg
```

The generated base64 string can then be supplied to the `/predict` endpoint using tools such as **Postman**, **curl**, or another API client.

---

# 🛠️ Tech Stack

| Technology             | Purpose                      |
| ---------------------- | ---------------------------- |
| **Python 3.10+**       | Core development             |
| **Ultralytics YOLOv8** | Object detection             |
| **PyTorch**            | Deep learning framework      |
| **Flask**              | REST API                     |
| **OpenCV**             | Image processing             |
| **NumPy**              | Numerical computation        |
| **Pandas**             | Data processing              |
| **Matplotlib**         | Evaluation and visualization |

---

# 🔮 Future Improvements

Potential improvements include:

* GPU-based training and inference
* Larger and more diverse training dataset
* Improved detection of dry irrigation canals
* Real-time camera/image-stream detection
* GPS-based asset mapping
* Interactive GIS visualization
* Cloud-based model deployment
* REST API authentication
* Automated asset database generation
* Model optimization for edge devices
* Mobile application integration

---

# 🙏 Acknowledgements

This project was developed in the context of the **Jal Jeevan Hariyali initiative**, with the goal of exploring automated identification and mapping of rural water resources and irrigation infrastructure using computer vision and deep learning.

---

## 👨‍💻 Author

**Yuvraj Aarsh**

---

## 📄 License

This project is intended for **educational, research, and demonstration purposes**. Please refer to the repository for any additional licensing or dataset usage restrictions.
