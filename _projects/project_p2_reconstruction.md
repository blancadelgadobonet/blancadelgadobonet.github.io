---
layout: page
title: reconstruction
description: reconstructing a scene using two images
img: # assets/img/12.jpg
importance: 1
category: robotics
---

**Robots aim to interact with their surroundings**. To fulfil this objective, it is essential to know **where the robot is within the scene**. 
Localization is the process of determining where a mobile robot is with respect to its environment. Just like humans determine their 
position according to what they can see or hear, robots can use information from the scene to localize themselves. A key sensor to 
retrieve data from the robot’s surroundings is the camera, in fact, one way to obtain the position of a robot is by detecting known 
beacons in images captured from the scene with a calibrated camera. In this practice, **visual autolocalization based on beacons** will be 
implemented using Python.


## Geometric autolocalization using beacons

**Beacons** are patterns easy to distinguish and hard to confound. A popular design is the ArUco marker, a square of N-by-N black and white 
pixels, of known size. Autolocalization using beacons consists on estimating the position of the camera  -- from which the position of 
the robot can be computed -- by detecting a known marker with said known camera. Knowing the camera implies knowing its intrinsic parameters,
for which its **calibration** is necessary. Using the detected points of the maker in the image (and their corresponding points in the real 
world), the **Perspective-n-Point (PnP)** algorithm allows estimating the pose (i.e., position and orientation) of an object with respect to a 
known camera (or the pose of the camera with respect to the object). In sum, the process can be divided in three steps: *1) calibration* of 
the camera to obtain its intrinsic parameters, *2) detection of the marker* in the image, *3) computation of the pose* of the object (or camera) 
using the intrinsic parameters, the detected points of the marker in the image and the known points of the marker in the real world.

The final result looks as follows:

{% include youtube.html id="A_zOSGOU7Y4" %}

{% include youtube.html id="NuGvpLIXxqs" %}

The camera is first calibrated with a chessboard pattern. Once calibrated, the marker is detected within the image and the pose is estimated and
displayed. The position of the camera -- its center -- with respect to the reference point within the marker, is indicated by the red dot. The orientation
of the camera, i.e., its x, y and z axes, are represented in red, green and blue, respectively. Note that the camera is always pointing towards the 
marker (we would not be able to localize it otherwise!).


### Okey, but how?
