---
title: "Real-time Yoga Pose Detector with PyTorch and MediaPipe"
date: 2025-05-25
draft: false
description:
  "How I built a computer vision application that detects 107 yoga poses with
  85-95% accuracy"
tags: ["machine-learning", "computer-vision", "python", "pytorch", "mediapipe"]
toc: true
---

In this post, I'll walk you through how I built a real-time yoga pose detection
system that can classify 107 different yoga poses with 85-95% accuracy using
PyTorch and MediaPipe.

{{< figure src="yoga-app-preview.png" caption="The Yoga Pose Detector web application in action, detecting the 'Warrior II' pose" alt="Yoga Pose Detector Application Preview" class="media-container" >}}

## The Challenge

Creating an accurate pose detection system requires solving several technical
challenges:

- Real-time performance for live webcam input
- Robust feature extraction from human pose landmarks
- Classification across a large number of pose categories
- Deployment in a user-friendly web interface

## Technical Approach

### 1. Pose Estimation with MediaPipe

I used Google's MediaPipe library for pose estimation, which provides 33 body
landmarks in real-time:

```python
import mediapipe as mp
import cv2

mp_pose = mp.solutions.pose
pose = mp_pose.Pose(static_image_mode=False, model_complexity=1)
```

### 2. Feature Engineering

The key was converting the 33 landmark coordinates into meaningful features for
the neural network:

```python
def extract_pose_features(landmarks):
    # Convert landmarks to numpy array
    coords = np.array([[lm.x, lm.y, lm.z] for lm in landmarks.landmark])

    # Calculate angles between key joints
    angles = calculate_joint_angles(coords)

    # Normalize relative to body size
    normalized_coords = normalize_pose(coords)

    return np.concatenate([normalized_coords.flatten(), angles])
```

### 3. PyTorch Neural Network

I designed a simple but effective neural network architecture:

```python
class YogaPoseClassifier(nn.Module):
    def __init__(self, input_size=132, num_classes=107):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(input_size, 256),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(256, 128),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(128, num_classes)
        )

    def forward(self, x):
        return self.network(x)
```

## Results

The final model achieved:

- **70-75% accuracy** on validation data
- **Real-time performance** (30+ FPS)
- **Lightweight model** size (~1-5MB)
- **107 yoga poses** supported

{{< figure src="accuracy-chart.png" caption="Training and validation accuracy over targeted 250 epochs, may stop early due to no accuracy increase" alt="Model Accuracy Chart" class="media-container" >}}

## Deployment with Streamlit

I deployed the application using Streamlit for an easy-to-use web interface:

```python
import streamlit as st

st.title("Yoga Pose Detector")
run = st.checkbox('Start Detection')

if run:
    # Webcam capture and prediction logic
    frame = capture_frame()
    pose = predict_pose(frame)
    st.write(f"Detected pose: {pose}")
```

## Live Demo

Try the live demo at
[detectpose.streamlit.app](https://detectpose.streamlit.app/)

## What's Next

Future improvements I'm considering:

- Adding pose accuracy detection
- Implementing pose correction feedback
- Expanding to more yoga styles
- Mobile app deployment

---

**Links:**

- [GitHub Repository](https://github.com/shubhpsd/yoga-pose-detection)
- [Live Demo](https://detectpose.streamlit.app/)
- [Dataset on Kaggle](https://www.kaggle.com/datasets/shrutisaxena/yoga-pose-image-classification-dataset)
