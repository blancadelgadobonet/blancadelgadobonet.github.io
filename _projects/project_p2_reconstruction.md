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
nevertheless, reconstruct it using additional information, i.e., two images. In this practice, **3D reconstruction** using two images as input will be implemented using Python.


## Software-Based 3D Reconstruction

Every 3D point can be seen from two rays; one ray comes from the first image and a second ray comes from the second image. These rays emerge from the center of each camera, go through a point in the 2D image (where the 3D point is projected) and reach a 3D point in the scene. They can be visualized as two lasers meeting at a unique 3D point. This concept is the basis of 3D reconstruction. **Given the two 2D points** (in the images) in which a single 3D point is projected, **we can triangulate to approximate the position of the 3D point** in the scene. Hence, given two images we can approximate the 3D scene.

Now, finding all correspondences in the 2D images is costly. It takes time and matching errors can be produced. To optimize the process, different strategies have been adopted. On the first hand, a sparse or a dense approach can be posed. On the other hand, geometric - epipolar - properties can be considered. 

A dense approach consideres all points in the 2D images, whereas a sparse approach selects the most characteristic points (e.g., edges) in the images to build the 3D scene. The former allows a more complete, but much slower, reconstruction. The latter tends to be less complete, but might retain all essential information if points are well chosen. Because less points are considered, it results much faster. **Attentive restriction** consists on focusing on certain pixels in the images, instead of reconstructing all of them.

Epipolar geometry is the geometry derived from stereo vision, when two cameras (or one moving camera) view the 3D scene from two distinct positions. The **epipolar restriction** defines, for each point in the first image, a line (or a margin, to consider possible erros) in the second image in which its correspondence should be found. Furthermore, a **maximum disparity** can be established within the line: this is the maximum distance in the horizontal axis between the point in one image and the point in the other image. Hence, the correspondence can be reduced to a line in the horizontal axis (epipolar line) and a line in the vertical axis (with a width according to the horizontal axis). 

Using these restrictions, the matches can be identified much more efficiently and used for triangulation to calculate the 3D positions.
The final result looks as follows:

{% include youtube.html id="xRQ7DFDMx-A" %}

First, **feature points** are detected in both images (attentive constraint). Points from one image are then matched to their corresponding points in the second image applying the **epipolar constraint** (the point in the first image needs to be in the epipolar line of the second image) and the **maximum disparity** (only points in the second image, within the epipolar line, close to the point in the first image are accepted). Then, correspondences are used to **triangulate** so as to obtain the 3D point. Finally, the best approximations are plotted to reconstruct the scene.


### Okey, but how?

- stereo vision 3D reconstruction

- epipolar geometry: constraints
- geometric properties: 3D points and proyections in the cameras

Selection of characteristic points. Bilateral filtering? Edges. Canny edge detection algorithm.

For each point in one image, build the epipolar line in the second image.
Restrict the search to a region of maximum disparity.

Find the correspondences. Identify the neighborhoood of the point the first image and compare to every point in hte restricted area of the second image (and its corresponding neighboorhood).

Triangulate.
Identify correct triangulations.


