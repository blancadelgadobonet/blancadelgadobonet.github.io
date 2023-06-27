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


To reconstruct the scene, stereo vision is crucial. Two images, looking at the same scene, need to be acquired. They are also smoothed
using bilateral filtering (`cv2.bilateralFilter`), to remove noise.

To apply the attentive restriction, characteristic points from the image are chosen: edges. The edges are selected using the Canny filter
(`cv2.Canny`) and using the colored image as input. Automatic thresholds were successfully chosen according to the following formulae:

\begin{equation}
\label{eq:lower}
    lower = max(0, 0.7 * mean) 
\end{equation}

\begin{equation}
\label{eq:lower}
    upper = min(255, 1.3 * mean) 
\end{equation}

*Downsampling the image to one cannal - grey values - and smoothing it was also tried as input to the Canny filter, but the performance decreased. Introducing the color image directly to the filter proved better outcomes and was therefore chosen as part of the pipeline.*

Next, the epipolar restriction was applied. To that end, both the epipole and 
The 3D center of the cameras are computed, and transformed to homogeneous coordinates.

The epipoles are extracted:
e1 = P1 C2

Projecting the center of the second camera in the first camera, the epipole of the first image is obtained. Similarly, the epipole in the second image can be computed.

This is in optical coordinates, needs to be moved to graphic HAL.opticalToGrafic


Point in the first image is projected on the second image. This implies changing the point to homogeneous coordinates from graphic to optical HAL.graficToOptical
Then backproject HAL.backproject to the 3D scene (following the ray of the first image). Then project HAL.project back to th e second image (following the second ray).
Finally recovering the graphic coordinates. HAL.opticalToGrafic

The epipolar line

Up to now, for each point in the first image, all points in the second image corresponding to edges and lying in the line should be considered. But an additional requirement is introduced: maximum disparity. Only points (in the second image) that are close to the original index (in the first image) - that is, within a maximum disparity - are going to be analysed. In this project, a maximum disparity of 20 pixels was allowed.

*In addition, another restriction was tried. Points in the second image that had already found a match in the first image were excluded for posterior matches under the hypothesis that correspondences were 1:1. Ideally, the analysis would have speed up the processing and more accurate matches would have been obtained. Yet, the performance was worsen. Indeed, correspondes are not necessarily 1:1 so this restriction was removed from the reconstruction.*


Matching. Kernel 11, 11.

Threshold 30.

- stereo vision 3D reconstruction

- epipolar geometry: constraints
- geometric properties: 3D points and proyections in the cameras

Selection of characteristic points. Bilateral filtering? Edges. Canny edge detection algorithm.

For each point in one image, build the epipolar line in the second image.
Restrict the search to a region of maximum disparity.

Find the correspondences. Identify the neighborhoood of the point the first image and compare to every point in hte restricted area of the second image (and its corresponding neighboorhood).

Triangulate.
Identify correct triangulations.


