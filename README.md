# Box Dimension Estimation from Depth Camera Point Clouds

This project implements a computer vision pipeline to estimate the physical dimensions of a box using depth-camera data. The input data consists of registered amplitude images, distance images, and 3D point clouds. The pipeline detects the floor plane and the top surface of the box, segments the relevant box region, and estimates the box height, length, and width from 3D geometry.

## Project Overview

The main goal of this project is to measure a box from noisy 3D point-cloud data. Instead of relying on individual distance values, the solution fits geometric plane models to the scene. The floor plane is detected first, then removed from the point cloud. A second plane is detected from the remaining points and interpreted as the top surface of the box. The final box dimensions are calculated using the distance between the floor and top planes, along with the spatial extent of the detected box-top region.

## Features

- Load and visualize amplitude images, distance images, and 3D point clouds
- Filter invalid depth measurements from the point cloud
- Implement custom RANSAC plane fitting from scratch
- Detect the dominant floor plane in the scene
- Remove floor points and detect the top plane of the box
- Generate binary masks for detected plane inliers
- Improve masks using morphological opening and closing
- Extract the largest connected component for the box top
- Estimate box height, length, and width in centimeters
- Compare standard RANSAC with MLESAC-style scoring
- Experiment with Preemptive RANSAC for runtime and robustness analysis

## Methodology

### 1. Data Loading and Visualization

The dataset is provided in `.mat` files containing:

- Amplitude image
- Distance image
- Registered 3D point cloud

Since the images and point cloud are registered, each pixel position corresponds to the same location across all data representations. The amplitude and distance images are visualized in 2D, while the point cloud is visualized in 3D using Matplotlib.

### 2. Invalid Point Filtering

The point cloud contains invalid measurements where the `z` coordinate is equal to zero. These points are removed before applying plane detection to improve reliability and avoid incorrect model fitting.

### 3. Floor Plane Detection

A custom RANSAC implementation is used to detect the dominant plane in the valid point cloud. Random triplets of 3D points are sampled to estimate candidate planes. For each candidate plane, point-to-plane distances are computed and points within a fixed threshold are counted as inliers. The plane with the best inlier support is selected as the floor plane.

### 4. Mask Refinement

The inlier points of the detected floor plane are converted into a binary image mask. Morphological opening and closing are applied to reduce noise and fill small gaps in the detected floor region.

### 5. Box Top Detection

After removing the floor points, RANSAC is applied again to the remaining point cloud to detect the top surface of the box. A normal-direction constraint is used so that the detected top plane is approximately parallel to the floor plane.

### 6. Connected Component Extraction

The top-plane inliers are converted into a binary mask. Connected-component analysis is then used to select the largest connected component, which is assumed to represent the actual top surface of the box.

### 7. Dimension Estimation

The box height is calculated from the distance between the floor plane and the top plane. The length and width are estimated from the 3D coordinates of the detected box-top points using PCA-based extent estimation in the XY plane.

## Results

| Dataset | Height | Length | Width |
|---|---:|---:|---:|
| Example 1 | 19.44 cm | 49.12 cm | 31.62 cm |
| Example 2 | 18.81 cm | 48.04 cm | 31.16 cm |
| Example 3 | 5.27 cm | 176.33 cm | 71.37 cm |
| Example 4 | 1.30 cm | 102.84 cm | 69.15 cm |

The first two examples produced stable and realistic box measurements. The third and fourth examples show less reliable dimensions, likely due to challenging segmentation, background clutter, or incorrect connected-component selection.

## Technologies Used

- Python
- NumPy
- SciPy
- Matplotlib
- 3D Point Cloud Processing
- RANSAC
- MLESAC-style scoring
- Preemptive RANSAC
- Morphological Image Processing
- Connected Component Analysis
- PCA-based Dimension Estimation

## How to Run

1. Clone the repository:

```bash
git clone <repository-url>
cd <repository-name>
```

2. Install the required dependencies:

```bash
pip install numpy scipy matplotlib
```

3. Place the dataset files inside a `data/` folder.

4. Open and run the notebook:

```bash
jupyter notebook Exercise_1_Solution.ipynb
```

Update the dataset paths in the notebook if your local folder structure is different.

## Project Structure

```text
.
├── Exercise_1_Solution.ipynb
├── data/
│   ├── example1kinect.mat
│   ├── example2kinect.mat
│   ├── example3kinect.mat
│   └── example4kinect.mat
└── README.md
```

## Limitations

- The method is sensitive to RANSAC threshold selection.
- Noisy or incomplete point-cloud data can affect plane detection.
- Background objects may be incorrectly included in the box-top mask.
- The largest connected component assumption may fail in cluttered scenes.
- Length and width estimation depends on accurate segmentation of the box top.

## Future Improvements

- Add automatic parameter tuning for RANSAC thresholds.
- Improve box-top segmentation using stronger geometric constraints.
- Use outlier removal before plane fitting.
- Add oriented bounding box estimation for more accurate length and width.
- Convert the notebook into a reusable Python module or command-line tool.

## Key Learning Outcomes

This project provided hands-on experience with 3D computer vision, point-cloud processing, robust plane fitting, and geometric measurement. It also demonstrated the importance of preprocessing, mask refinement, and parameter selection when working with real-world depth-camera data.
