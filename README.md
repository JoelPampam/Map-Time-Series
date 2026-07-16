# Historical Map Verification using Shapelet-Based Time Series Analysis

> Transforming historical map boundaries into one-dimensional time series for automated geometric verification.

---

## Overview

Historical maps often vary in scale, projection, and drawing style, making direct geometric comparison difficult. This project presents a **shapelet-based time series classification** approach that converts polygon boundaries into radial distance signals and compares them using subsequence similarity.

Rather than comparing entire polygons directly, the boundary of each map feature is represented as a **radial distance profile** measured from the polygon centroid. A discriminative **shapelet** is automatically learned from the modern reference map and matched against historical maps to quantify geometric similarity.

The result is an automated workflow for evaluating whether historical maps preserve the geometry of the same geographic feature over time.

---

## Pipeline

```text
                Historical Maps
                       │
                       ▼
              Georeference (QGIS)
                       │
                       ▼
             Digitize Polygon Boundary
                       │
                       ▼
             Compute Polygon Centroid
                       │
                       ▼
          Create Radial (Hub) Lines
                       │
                       ▼
            Densify Boundary Sampling
                       │
                       ▼
      Extract Angle & Radial Distance Data
                       │
                       ▼
              Export CSV Measurements
                       │
                       ▼
        Interpolate to Common Angle Grid
                       │
                       ▼
          Learn Representative Shapelet
                       │
                       ▼
      Compare Historical Maps to Shapelet
                       │
                       ▼
        Generate Similarity Scores & Plots
```

---

# Data Preparation

All preprocessing was performed in **QGIS** before analysis in Python.

The workflow included:

- Georeferencing historical maps
- Digitizing the feature of interest as a polygon
- Computing the polygon centroid
- Extracting polygon vertices
- Creating radial (hub) lines from the centroid
- Densifying the hub lines
- Measuring radial distance at each angular position
- Exporting the measurements as CSV files

Each CSV contains the polygon represented as a one-dimensional radial distance signal.

---

# Dataset

Reference map:

- **2026 Cambridge**

Historical maps:

- 1574
- 1798
- 1950

Negative controls:

- Harvard
- Amsterdam

Each dataset contains radial distance measurements sampled around the polygon boundary.

---

# Methodology

## 1. Interpolation

Each polygon contains a different number of sampled boundary points.

To enable direct comparison, every radial distance profile is interpolated onto a common angular grid consisting of **100 equally spaced angles between 0° and 360°**.

---

## 2. Shapelet Discovery

Instead of learning from every map simultaneously, this project learns a representative shapelet **only from the modern (2026) reference map**.

Every possible subsequence of length **20 samples** is extracted using a sliding window.

For each candidate shapelet:

- z-normalization is applied
- The minimum distance to Harvard is computed
- The minimum distance to Amsterdam is computed
- The average of those distances becomes the candidate score

The subsequence that is **most different from unrelated maps** is selected as the learned shapelet.

---

## 3. Shapelet Matching

The selected shapelet is compared against every map.

For each map:

1. Sliding windows are generated.
2. Every window is z-normalized.
3. Mean squared distance to the learned shapelet is computed.
4. The minimum distance becomes that map's similarity measurement.

---

## 4. Similarity Scoring

Raw distances are converted into normalized similarity scores.

A decision boundary is calculated using

```text
(Target Maximum Distance + External Minimum Distance) / 2
```

Similarity is then computed as

```text
Similarity = 1 - Distance / Boundary
```

Scores are clipped between

```text
[-1, 1]
```

where

| Score | Meaning |
|--------|----------|
| 1 | Nearly identical |
| 0 | Decision boundary |
| -1 | Completely different |

---

# Visualizations

The program produces four figures.

### 1. Radial Distance Time Series

Displays every historical map as a radial distance signal.

The location where the learned shapelet best matches each map is highlighted.

---

### 2. Learned Shapelet

Displays the subsequence automatically selected from the reference map.

---

### 3. Historical Map Similarity

Bar chart showing similarity scores for

- 1574
- 1798
- 1950
- 2026

---

### 4. External Map Comparison

Compares the reference map against unrelated locations.

- Cambridge
- Harvard
- Amsterdam

This demonstrates that the learned shapelet distinguishes the correct geographic region from unrelated maps.

---

# Project Structure

```text
.
├── 1574 hub dense vert.csv
├── 1798 hub dense vert.csv
├── 1950 hub dense vert.csv
├── 2026 hub dense vert.csv
├── Harvard Hub dense vert.csv
├── Amsterdam Hub dense vert.csv
├── shapelet_analysis.py
├── README.md
└── figures/
```

---

# Technologies

- Python
- NumPy
- Pandas
- Matplotlib
- QGIS

---

# Example Output

The analysis produces:

- Learned reference shapelet
- Best matching location on every historical map
- Raw shapelet distances
- Normalized similarity scores
- Publication-quality figures

Example console output:

```text
Best shapelet index in 2026: 31

Raw distances

1574       0.118
1798       0.097
1950       0.043
2026       0.000
Harvard    0.391
Amsterdam  0.426
```

---

# Why Radial Distance?

Representing polygon boundaries as radial distance signals provides several advantages:

- Rotation-independent after alignment
- Converts spatial geometry into a one-dimensional signal
- Enables efficient subsequence comparison
- Compatible with time series machine learning algorithms
- Preserves characteristic boundary shapes

---

# Future Work

Possible extensions include:

- Multiple learned shapelets
- ROCKET / MiniROCKET classifiers
- Dynamic Time Warping (DTW)
- Automatic polygon extraction from raster maps
- Topological verification using graph representations
- Larger historical map datasets
- Deep learning approaches for map verification

---

# References

- Ye, L., & Keogh, E. (2009). *Time Series Shapelets: A New Primitive for Data Mining.*
- pyts: A Python Package for Time Series Classification
- QGIS Geographic Information System

---

