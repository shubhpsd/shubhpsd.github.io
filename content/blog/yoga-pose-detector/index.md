---
title: "Teaching Machines Yoga: My PGDSMA Capstone Project"
date: 2025-05-25
draft: false
description:
  "Built a real-time yoga pose detector for my capstone at Indian Statistical
  Institute. From B.Com to machine learning - sometimes the best projects
  combine your past and present."
tags:
  [
    "machine-learning",
    "computer-vision",
    "python",
    "pytorch",
    "mediapipe",
    "yoga",
    "capstone",
    "isi",
  ]
toc: true
cover:
  src: ./yoga-cover.webp
  alt: "People in movie theatre."
---

## What I Built

This was my capstone project for the PGDSMA program at Indian Statistical
Institute - a real-time yoga pose detection system that recognizes 107 different
poses. Pretty wild journey from a B.Com(Hons.) background to building computer
vision models that can tell the difference between warrior pose and tree pose.

{{< figure src="yoga-app-preview.png" caption="The app catching Bakasana - one of the trickier poses to detect" alt="Yoga Pose Detector Application Preview" class="media-container" >}}

## Why This Project Made Sense for Me

Here's the thing - I started my fitness journey with yoga before transitioning
into serious weightlifting and gym training. While I'm more about bicep curls
and bench press these days, those yoga foundations taught me a lot about body
awareness, form, and the importance of proper alignment.

When it came time to choose a capstone project for my PGDSMA, I wanted something
that combined my technical learning with personal experience. Having spent
countless hours perfecting poses and understanding the nuances of form, I
thought - why not teach a computer to do the same thing?

Plus, coming from a B.Com background, I needed a project that would really
showcase the machine learning and computer vision skills I'd developed during
the program.

## The Technical Implementation

### Step 1: Understanding Human Movement (MediaPipe Magic)

Coming from a fitness background, I understand how important proper form is -
whether you're doing a perfect warrior III or nailing a deadlift. The first
challenge was teaching a computer to "see" human movement the way a trainer
would.

Enter Google's MediaPipe. This library extracts 33 body landmarks in real-time,
essentially giving the computer a skeleton view of the human body:

```python
import mediapipe as mp
import cv2

# Initialize the pose detection
mp_pose = mp.solutions.pose
pose = mp_pose.Pose(
    static_image_mode=False,
    model_complexity=1,  # Balance between speed and accuracy
    min_detection_confidence=0.7
)
```

### Step 2: Feature Engineering (The Real Challenge)

Raw landmark coordinates are like having gym measurements without understanding
what they mean. The magic happens in feature engineering - converting those 33
points into meaningful biomechanical information:

```python
def extract_pose_features(landmarks):
    # Extract all coordinate points
    coords = np.array([[lm.x, lm.y, lm.z] for lm in landmarks.landmark])

    # Calculate joint angles - crucial for pose recognition
    angles = calculate_joint_angles(coords)

    # Normalize relative to body size (everyone's built differently)
    normalized_coords = normalize_pose(coords)

    # Combine into feature vector
    return np.concatenate([normalized_coords.flatten(), angles])
```

Having trained with different body types in the gym, I knew normalization was
crucial. A 5'4" person doing tree pose looks very different from a 6'2" person,
but the angles and proportions should be similar.

### Step 3: The Neural Network Architecture

For my capstone, I chose a relatively simple but effective PyTorch architecture
that could handle real-time inference:

```python
class YogaPoseClassifier(nn.Module):
    def __init__(self, input_size=132, num_classes=107):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(input_size, 256),
            nn.ReLU(),
            nn.Dropout(0.3),  # Prevent overfitting
            nn.Linear(256, 128),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(128, num_classes)
        )

    def forward(self, x):
        return self.network(x)
```

I kept it simple intentionally - sometimes the best solutions are the most
elegant ones.

## The Smart Move: Why MediaPipe Instead of Raw Image Training

Here's where I made a strategic decision that probably saved me months of work
and tons of computational resources. Instead of training a model directly on raw
images (like most computer vision projects), I used MediaPipe as a preprocessing
step. This was honestly brilliant - let me explain why:

### Traditional Approach vs. My Approach

**Traditional Image Classification:**

- Feed raw 224x224x3 images directly into a CNN
- Need massive datasets (millions of images)
- Requires powerful GPUs for weeks of training
- Model size: 50-500MB
- Inference time: 100-500ms per frame

**My MediaPipe + Neural Network Approach:**

- Extract 33 pose landmarks (132 features) using MediaPipe
- Train on structured numerical data instead of pixels
- Need smaller datasets (thousands of examples)
- Trains on CPU in hours, not days
- Model size: 1-5MB
- Inference time: 5-10ms per frame

### Why This Was Fast AF

```python
# Traditional approach - processing raw pixels
image = cv2.resize(frame, (224, 224))  # 150,528 features
prediction = cnn_model(image)  # Millions of parameters

# My approach - structured pose data
landmarks = mediapipe_pose.process(frame)  # 33 landmarks
features = extract_pose_features(landmarks)  # 132 features
prediction = lightweight_model(features)  # Thousands of parameters
```

The computational difference is insane. Instead of processing 150k+ pixel
values, I'm working with 132 engineered features. MediaPipe already did the
heavy lifting of understanding human anatomy - I just needed to classify the
pose based on body geometry.

### Training Speed Benefits

- **Dataset size**: Instead of needing millions of pose images, I could work
  with thousands of pose landmark sequences
- **Training time**: What would take days on GPUs took hours on my laptop
- **Memory usage**: Could train on 16GB RAM easily instead of needing 32GB+
- **Iteration speed**: Quick experiments meant I could try different
  architectures rapidly

### The 2D Problem: Current System Limitations

It's important to highlight the limitations honestly, this approach has a
fundamental limitation that needs to be addressed. The current system sees poses
in **2D space only**. MediaPipe gives me x, y, z coordinates, but the z (depth)
information is pretty limited compared to x and y.

**What this means in practice:**

- **Warrior III vs. Standing Forward Fold**: From certain camera angles, these
  can look identical in 2D
- **Twisted poses**: Hard to distinguish twists that primarily happen in the
  depth dimension
- **Side vs. Front view confusion**: Same pose from different angles can confuse
  the model
- **Depth-dependent poses**: Poses where the key distinguishing feature is how
  far forward/back limbs extend

```python
# Current limitation example
pose_1 = [arm_raised_x, arm_raised_y, limited_z]  # Could be multiple poses
pose_2 = [arm_raised_x, arm_raised_y, limited_z]  # Same 2D projection, different 3D pose
```

The MediaPipe strategy was perfect for a capstone project - fast iteration,
quick results, real-time performance. But for production-level accuracy, I'll
need to solve the 3D understanding problem.

## Capstone Results: The Good and the Room for Growth

For my ISI capstone defense, I presented results showing **70-75% accuracy** on
validation data. Not bad for a first deep dive into computer vision, but
definitely room for improvement. Here's what worked:

- **Real-time performance**: 30+ FPS - crucial for practical applications
- **Lightweight model**: Only 1-5MB - could potentially run on mobile devices
- **107 yoga poses**: Comprehensive coverage from basic to advanced poses
- **Functional web interface**: Streamlit app that actually works

{{< figure src="accuracy-chart.png" caption="Training curves from my capstone - plateau around 75% showed where I needed to dig deeper" alt="Model Accuracy Chart" class="media-container" >}}

The 70-75% accuracy was honestly a bit frustrating. Coming from a fitness
background, I know how crucial precision is - you wouldn't accept a spotter
who's wrong 25% of the time!

## The Web App (Streamlit Does the Job)

Built a simple web interface so people can actually try it without setting up
Python environments:

```python
import streamlit as st

st.title("🧘‍♀️ Yoga Pose Detector")
st.write("Point your camera at yourself doing yoga and let's see what happens")

run = st.checkbox('Start Detection')
confidence_threshold = st.slider('Confidence Threshold', 0.1, 1.0, 0.7)

if run:
    # Real-time webcam magic happens here
    frame = capture_frame()
    pose, confidence = predict_pose(frame)

    if confidence > confidence_threshold:
        st.success(f"Detected: {pose} ({confidence:.2%} confident)")
    else:
        st.warning("Not sure what pose this is... 🤔")
```

It's basic but functional. You can try it yourself at
[detectpose.streamlit.app](https://detectpose.streamlit.app/)

## Post-Capstone Improvements: Taking It to the Next Level

Finishing my PGDSMA gave me the foundation, but I've got serious plans to push
this beyond academic project territory. Here's my roadmap to get this thing to
85-90%+ accuracy:

### SMOTE for Data Balancing

The dataset is heavily imbalanced - some poses have thousands of examples while
others have maybe 50. As someone who's programmed workout splits, I know balance
is everything:

```python
from imblearn.over_sampling import SMOTE

# Balance the dataset by generating synthetic samples
smote = SMOTE(random_state=42, k_neighbors=3)
X_balanced, y_balanced = smote.fit_resample(features, labels)

print(f"Original dataset: {Counter(labels)}")
print(f"Balanced dataset: {Counter(y_balanced)}")
```

This should help with those advanced poses like "Eight-Angle Pose" or "Firefly"
that barely show up in training data.

### Advanced Algorithms Worth Exploring

**Random Forest with Biomechanical Features**: My fitness background tells me
that pose recognition is about understanding relationships between muscle groups
and joint angles. Ensemble methods might capture these complex patterns better.

**Transformer Architecture**: For sequence-like data with spatial relationships,
attention mechanisms could help the model focus on the most important landmarks
for each pose - like how a trainer's eye goes straight to hip alignment in
warrior poses.

**Gradient Boosting (XGBoost/LightGBM)**: These work surprisingly well on
feature-engineered data. Plus they're interpretable - I could actually see which
angles and landmarks matter most for each pose.

**Temporal Smoothing**: Adding time-series analysis to consider multiple frames.
Yoga poses are held positions - this should reduce the jittery predictions I saw
during testing.

### Data Augmentation Tricks

**Perspective Augmentation**: Simulate different camera angles since people
won't always be perfectly centered.

**Noise Injection**: Add small amounts of noise to landmark coordinates to make
the model more robust.

**Mirror Flipping**: Create left/right variants of asymmetric poses to double
the training data.

The goal is to push this from "decent party trick" to "actually useful yoga
assistant."

## What This Capstone Taught Me

### From B.Com to Computer Vision

The transition from commerce background to building ML models wasn't easy, but
this project really solidified everything I learned during PGDSMA. Understanding
the math behind neural networks is one thing - implementing them to solve real
problems is completely different.

### Fitness Experience Actually Helped

My background in yoga and gym training gave me insights that pure CS students
might miss. I understood why certain poses are easily confused (similar joint
angles), why some are harder to hold (stability requirements), and how
individual body differences affect form.

### Real-Time Constraints Change Everything

Academic projects can take hours to process one sample. Real applications need
30+ FPS or users get frustrated. This forced me to balance model complexity with
performance - a crucial lesson for production ML.

### Data Quality Beats Fancy Algorithms

Having 10,000 perfect examples beats 50,000 messy ones. Some poses in the
dataset had questionable labels that definitely hurt performance. Clean data is
everything.

### The Value of Domain Knowledge

Combining technical skills from PGDSMA with personal fitness experience made
this project way stronger than either alone would have been.

## Capstone Defense and Beyond

Presenting this at ISI was nerve-wracking but rewarding. The professors
appreciated the practical application and the technical depth. Getting through
the defense with a project I actually cared about was worth it.

The code is all open source, and the web app is live if you want to test it:

- [GitHub Repository](https://github.com/shubhpsd/yoga-pose-detection) -
  Complete codebase, training notebooks, and capstone documentation
- [Live Demo](https://detectpose.streamlit.app/) - Try it with your webcam
- [Dataset on Kaggle](https://www.kaggle.com/datasets/shrutisaxena/yoga-pose-image-classification-dataset) -
  Original dataset used

---

_From B.Com to building computer vision models - sometimes the best career moves
are the ones that scare you a little. Now back to the gym where the only AI I
need is my training app counting reps, I should get to building that._
