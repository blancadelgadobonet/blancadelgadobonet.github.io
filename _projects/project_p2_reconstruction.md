---
layout: page
title: reconstruction
description: reconstructing a scene using two images
img: # assets/img/12.jpg
importance: 1
category: robotics
---

**Robots aim to interact with their surroundings**. To do so, information is retrieved from the environment using sensors. One of the cheapest sensors,
able to capture a vast quantity of information, is the camera. Yet, the camera reduces the reality to a 2D version. To recover the real scene we can, 
nevertheless, reconstruct it additional information, i.e., two images. In this practice, **3D reconstruction** using two images as input will be implemented
using Python.


## Software-Based 3D Reconstruction

- stereo vision 3D reconstruction

- epipolar geometry: constraints
- geometric properties: 3D points and proyections in the cameras


Every 3D point can be seen from two rays; one ray comes from the first image and a second ray comes from the second image. These rays emerge from the center of each camera, go through a point in the 2D image (where the 3D point is projected) and reach a 3D point in the scene. They can be visualized as two lasers meeting at a unique 3D point. This concept is the basis of 3D reconstruction. Given the two 2D points (in the images) in which a single 3D point is projected, we can triangulate to approximate the position of the 3D point in the scene. Hence, given two images we can approximate the 3D scene.




-
re patterns easy to distinguish and hard to confound. A popular design is the ArUco marker, a square of N-by-N black and white 
pixels, of known size. Autolocalization using beacons consists on estimating the position of the camera  -- from which the position of 
the robot can be computed -- by detecting a known marker with said known camera. Knowing the camera implies knowing its intrinsic parameters,
for which its **calibration** is necessary. Using the detected points of the maker in the image (and their corresponding points in the real 
world), the **Perspective-n-Point (PnP)** algorithm allows estimating the pose (i.e., position and orientation) of an object with respect to a 
known camera (or the pose of the camera with respect to the object). In sum, the process can be divided in three steps: *1) calibration* of 
the camera to obtain its intrinsic parameters, *2) detection of the marker* in the image, *3) computation of the pose* of the object (or camera) 
using the intrinsic parameters, the detected points of the marker in the image and the known points of the marker in the real world.

The final result looks as follows:

{% include youtube.html id="xRQ7DFDMx-A" %}


The camera is first calibrated with a chessboard pattern. Once calibrated, the marker is detected within the image and the pose is estimated and
displayed. The position of the camera -- its center -- with respect to the reference point within the marker, is indicated by the red dot. The orientation
of the camera, i.e., its x, y and z axes, are represented in red, green and blue, respectively. Note that the camera is always pointing towards the 
marker (we would not be able to localize it otherwise!).


### Okey, but how?
