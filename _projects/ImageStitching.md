---
layout: page
title: Image Stitching
description: to create a panorama image
img: assets/img/ImageStitching/best/stitched_hynet.png
importance: 3
category: Computer Vision
---
Let's create a panorama image!

Given 2 photos of UT tower, we are going to stitch these two images to create a panorama image.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ImageStitching/data/uttower_left.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ImageStitching/data/uttower_right.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Steps for Image Stitching

1. Load the images into grayscale image
2. Detect key points in both images
3. Extract features for key points
4. Match features between Images
5. Calculate homography
6. Refit with inliers
7. Warp one image onto the other.

<br>

## Details

## 1. Load the images into grayscale image
Load the images and convert them into grayscale image.

## 2. Detect key points in both images

Extract corner key points using Harris Corner Detector.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ImageStitching/best/img1_keypoint_vis.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ImageStitching/best/img1_keypoint_vis.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## 3. Extract features for key points

Extract local neighborhoods around every keypoint in both images.


## 4. Match features between images

Once features are detected, match them across adjacent images. Compute L2 distances between every descriptor in one image and every descriptor in the other image. Then, select putative matches based on the matrix of pairwise descriptor distances.

<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ImageStitching/best/inlier_matches_pixel.png" title="example image" class="img-fluid rounded z-depth-1" %}
</div>

## 5. Calculate homography
With matched features, compute the homography matrix, which maps points from one image to the corresponding points in the next. To estimate the homography matrix, we use RANSAC algorithm. 
Why is RANSAC Needed for homography calculation?
This is because when matching feature points between two images, not all matches are correct—some are outliers caused by noise, repeated patterns in the image, and errors in feature detection or matching.
If these outliers are included in the homography computation, the resulting transformation will be inaccurate. 

#### RANSAC helps by:
- Filtering out incorrect matches (outliers).
- Focusing on the most consistent matches (inliers).

#### How RANSAC is used in calculating homography:
Given matched feature points between two images,

1. Randomly select a small subset of matched points (usually 4, as 4 points are sufficient to calculate a homography).
2. Compute a candidate homography matrix using these points. 
3. Measure how well the candidate homography aligns the rest of the points (based on a threshold for reprojection error).
4. Keep the homography matrix that produces the highest number of inliers.


## 6. Refit with inliers
Once homography matrix and inliers are obtained, refit the homography matrix using only the inliers.

## 7. Warp one image onto the other

Using the homography matrix obtained from previous step, warp images into the same coordinate space so that they align perfectly.


Then, image stiching is done and we have our panorama image!

## Using Hynet Descriptor
Instead of using pixel descriptor, we can use pre-trained HyNet descriptor. HyNet takes in a 32 × 32 patch and outputs a feature vector of size 128.

Hynet usually provides better results than that of using pixel descriptor with different parameter settings (patch size, number of iterations, threshold, etc.)

## Results

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
            {% include figure.liquid loading="eager" path="assets/img/ImageStitching/best/stitched_pixel.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
            {% include figure.liquid loading="eager" path="assets/img/ImageStitching/best/stitched_hynet.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: pixel, Right: Hynet
</div>