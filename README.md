## Vehicle-TTC-Calculation
Solution developed as a submission for Intel Unnati Industrial Training Program 2024.

This repository contains the code for detecting vehicle cut-in events using a combination of computer vision and speed calculations derived from positional data. The project utilizes the YOLOv8 model from the Ultralytics module for object detection and calculates Time-To-Collision (TTC < 650ms) to issue warnings for potential cut-in events in front of the moving vehicle.

## Table of Contents

- [Introduction](#introduction)
- [Dataset](#dataset)
- [Installation](#installation)
- [Usage](#usage)
- [Technical Approach](#technical-approach)
    - [Loading Data and Calculating Speed](#loading-data-and-calculating-speed)
    - [Object Detection Using YOLOv8](#object-detection-using-yolov8)
    - [Distance Calculation from Bounding Boxes](#distance-calculation-from-bounding-boxes)
    - [Cut-In Detection](#cut-in-detection)
- [Results](#results)
    -[Overall Evaluation Time](#overall-evaluation-time)
    - [Evaluation of Images](#evaluation-of-images)
- [Video Demonstration](#video-demonstration)
- [License](#license)

## Introduction

Vehicle cut-in detection is crucial for Advanced Driver Assistance Systems (ADAS) and autonomous driving. This project aims to detect cut-in events by analyzing images from a car's camera and calculating distances to other vehicles in a single forward pass for minimum inference time. The system issues warnings if any object cuts in within a critical distance and time threshold.

## Dataset

The dataset used in this project includes images and positional data from the Indian Driving Dataset (IDD). The positional data comprises latitude, longitude, and altitude for each image frame. The exact dataset used can be found at https://www.kaggle.com/datasets/ravesandstorm/indian-driving-dataset-primary-obd

## Installation

To get started with this project, follow the steps below:

1. Clone the repository:
    ```bash
    git clone https://github.com/ravesandstorm/vehicle-TTC-Calculation.git
    cd vehicle-cut-in-detection
    ```

2. Install the required Python package:
    ```bash
    pip install ultralytics
    ```

3. Download the YOLOv8 model weights:
    ```bash
    YOLO('yolov8s.pt')
    ```

## Usage

### Preprocessing and Speed Calculation

1. Load the dataset and calculate smoothed speeds using the positional data.

### Object Detection and Cut-In Detection

2. Run the object detection and cut-in detection script on a single image, and whole dataset. Additionally, you can change the markdown code at the bottom to code to do a limited run to save some images of input and output of the model. 

## Technical Approach

### Loading Data and Calculating Speed

The initial step involves loading the necessary data from a CSV file containing positional data and timestamps. (The path to be used can be modified by changing the “MAIN_PATH” variable.) This includes latitude, longitude, and altitude for each image frame. Using these absolute data points, the relative speeds between consecutive frames are calculated.
The function “calc” calculates the distance between two geographical points on earth using the Haversine formula, which considers the curvature of the Earth. The altitude difference is also calculated from the given coordinates.

The resulting speed is then smoothed using an Exponential Moving Average (with alpha=0.2) to maintain consistency and fix the issue of obtaining absolute speeds for every consecutive frame, which is then stored in the array “smoothed_speeds”. Storing the values is because of the pre-recorded nature of the data, which can easily be modified to fit real-time analysis.

### Object Detection Using YOLOv8

To detect vehicles and objects, the YOLOv8 model from the Ultralytics module is used. The small model (yolov8s) was preferred after testing all other models as it gave the results with the highest accuracy considering the minimal inference time on GPU. (~15ms)
For each image, the model predicts bounding boxes, class IDs, and confidence scores.

### Distance Calculation from Bounding Boxes

The height of each detected object class is used to estimate the distance from the camera using the point at (x/2, y/3) of each bounding box (to reduce the amount of false positives) and the estimated focal length of the camera, using the formula given in section 2.3 of this paper (https://arxiv.org/abs/2106.10319). The distance is taken into consideration only if the point falls within two lines denoting the forward direction of the camera car (as considering objects outside this area leads to reduced accuracy and a significant increase in the number of false positives), marked in two white lines, whose equations are considered as inequalities to judge which lane a point falls into.

### Cut-In Detection

The Time-To-Collision (TTC) is calculated using the predicted distance and the speed at current frame and compared to a threshold (0.65s), below which a warning is issued, along with a displayed image with red tone. The model is evaluated in two modes, one with a random image from the dataset, and the other on all the images in the dataset. The first mode annotates all detected images with confidence > 0.3, to check the model’s functions on a random image from the dataset, the second mode is suited for real world applications, as it does not show every image with its annotations, but only the ones where a cut-in is detected. This allows decreasing evaluation time (~30fps with P100 GPU). The print of evaluation of every image is not recommended in constant evaluation mode as it significantly increases time of evaluation (since I used matplotlib.pyplot, evaluation is paused till graph is plotted). A workaround of this could be saving the cut-in detection images as logs in an output folder, since the dataset doesn’t wait for the plotting. 

Code Link: https://github.com/ravesandstorm/Vehicle-TTC-Calculation/blob/main/Files/ttc-calc.ipynb

## Results

This model acts as an ADAS to warn the human driver of cut-in of vehicles, animals, or objects.

### Overall Evaluation Time

For each device, I calculated the time for evaluating 1000 frames, from which I was able to get the frame rate the model was capable of without falling behind, in order to give an accurate TTC calculation and warning.
For 100 frames, time = 42.92s on CPU. (NOT RECOMMENDED) 
CPU is generally not recommended for inference on images at all, getting a very high evaluation time for just 100 frames.
For 1000 frames, time taken = 34.1369 on the NVIDIA T4 GPU, ~28 FPS. 
This device gave very good inference times, capable of running the project reliably with real time image data of 24 FPS.
For 1000 frames, time taken = 30.0911 on the NVIDIA P100 GPU, ~33FPS. 
This device gave the best inference times, capable of running the project reliably with real time image data of 30 FPS.
Both these frame rates are mostly standard for monitoring cameras used, and give a realistic approach to solving this problem in real-time.

### Evaluation of Images

Mode 1: Single Frame Evaluation
For a single random frame, the bounding boxes, distances, and the Time-To-Collision (TTC) are computed and visualized. If the TTC is below the threshold, a warning is issued. 
Mode 2: Full Dataset Evaluation
The model is evaluated on all frames to detect cut-in events. When such an event is encountered, the image is plotted on a graph with the bounding boxes and the whole image is colored red, along with a text warning. The number of detected events and warnings issued are recorded and printed.
This mode is to be calibrated and used in case of real time-data, with speed or positional data and focal length set according to the camera used, and instead of plotting the images, it should be saved to a log file to maintain the real-time evaluation speed of the live feed.
From the dataset provided, the model was able to detect a cut-in event from all the images from “idd_mm_primary/idd_multimodal/primary/d0”, which was a dog running across the lane in front of the vehicle, which is shown in the first video link provided. The second model shows the validity of running the model, with working of both modes shown. 

## Video Demonstration
Below is a demonstration video of the vehicle cut-in detection system in action:

<video controls>
  <source src="https://drive.google.com/uc?export=download&id=1iRIpXYUv-ybY79hJH-l1ns2dDPnT_MMH" type="video/mp4">
  Your browser does not support the video tag.
</video>

The links for the videos are:
Visualisation 1:     (https://drive.google.com/file/d/1iRIpXYUv-ybY79hJH-l1ns2dDPnT_MMH/view)
Model Run on Kaggle: (https://drive.google.com/file/d/1foAdxbrIQcTHnOKb28OD-3hwTJyuOLlT/view)

## License

If you use this project or its code, please credit me as Satvik.
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
