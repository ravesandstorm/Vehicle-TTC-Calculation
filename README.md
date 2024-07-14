## Vehicle-TTC-Calculation
Solution developed as a submission for Intel Unnati Industrial Training Program 2024.

This repository contains the code for detecting vehicle cut-in events using a combination of computer vision and speed calculations derived from positional data. The project leverages the YOLOv8 model for object detection and calculates Time-To-Collision (TTC) to issue warnings for potential cut-in events.

## Table of Contents

- [Introduction](#introduction)
- [Dataset](#dataset)
- [Installation](#installation)
- [Usage](#usage)
- [Technical Approach](#technical-approach)
- [Results](#results)
- [License](#license)

## Introduction

Vehicle cut-in detection is crucial for Advanced Driver Assistance Systems (ADAS) and autonomous driving. This project aims to detect cut-in events by analyzing images from a car's camera and calculating distances to other vehicles in a single forward pass for minimum inference time. The system issues warnings if a vehicle cuts in within a critical distance and time threshold.

## Dataset

The dataset used in this project includes images and positional data from the Indian Driving Dataset (IDD). The positional data comprises latitude, longitude, and altitude for each image frame. The exact dataset used can be found at https://www.kaggle.com/datasets/ravesandstorm/indian-driving-dataset-primary-obd

## Installation

To get started with this project, follow the steps below:

1. Clone the repository:
    ```bash
    git clone https://github.com/ravesandstorm/vehicle-TTC-Calculation.git
    cd vehicle-cut-in-detection
    ```

2. Install the required Python packages:
    ```bash
    pip install ultralytics
    ```

3. Download the YOLOv8 model weights:
    ```bash
    wget -P models/ https://path_to_yolov8_weights/yolov8s.pt
    ```

## Usage

### Preprocessing and Speed Calculation

1. Load the dataset and calculate smoothed speeds using the positional data:

### Object Detection and Cut-In Detection

2. Run the object detection and cut-in detection script.

### Configuration

Ensure that the paths to the dataset and model weights are correctly set in the script files.

## Technical Approach

### Loading Data and Calculating Speed

The positional data and timestamps are loaded from a CSV file. The distance between consecutive points is calculated using the Haversine formula, and the speed is smoothed using an Exponential Moving Average (EMA).

### Object Detection Using YOLOv8

The YOLOv8 model, pre-trained on the COCO dataset, is used for detecting vehicles and other objects in the images. The height of detected objects is used to estimate the distance from the camera.

### Distance Calculation from Bounding Boxes

The bounding box height and known focal length of the camera are used to calculate the distance to detected objects.

### Cut-In Detection

A cut-in event is detected if a vehicle is within a certain distance and in the second lane section. The Time-To-Collision (TTC) is calculated and compared to a threshold to issue warnings.

## Results

The system detects cut-in events and issues warnings if the TTC is below a critical threshold. The results are visualized, showing bounding boxes, distances, and TTC values.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
