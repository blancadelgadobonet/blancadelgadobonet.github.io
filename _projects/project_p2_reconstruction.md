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

To apply the <u>attentive restriction</u>, characteristic points from the image are chosen: edges. The edges are selected using the Canny filter
(`cv2.Canny`) and using the colored image as input. Automatic thresholds were successfully chosen according to the following formulae:

\begin{equation}
\label{eq:lower}
    lower = max(0, 0.7 * mean) 
\end{equation}

\begin{equation}
\label{eq:upper}
    upper = min(255, 1.3 * mean) 
\end{equation}

*Downsampling the image to one cannal - grey values - and smoothing it was also tried as input to the Canny filter, but the performance decreased. Introducing the color image directly to the filter proved better outcomes and was therefore chosen as part of the pipeline.*

Next, the <u>epipolar restriction</u> was applied. To that end, for every point in the first image, the epipolar line in the second image - joining the epipole of the second camera and the point of the first camera projected in the second camera - is computed. 

Projecting the **center** of the first camera (in homogeneous coordinates) in the second camera, the **epipole** of the second image is obtained:

\begin{equation}
\label{eq:epipole}
    e_2 = P_1 * C_2
\end{equation}

This point, in optical coordinates, is then transformed to graphic coordinates (`HAL.opticalToGrafic`).

To **project the point of the first camera in the second camera**, the point needs to be written in homogeneous coordinates and transformed from graphical to optical coordinates (`HAL.graficToOptical`). It is then backprojected to the 3D scene (`HAL.backproject`) and projected again but to the second camera (`HAL.project`). Lastly, it is transformed back from optical to graphic coordenates (`HAL.opticalToGrafic`).

Using the episode and the point of interest (both in the same camera), the **epipolar line** in homogeneous coordinates (i.e., l1x+l2y+l3z=0) is computed through a cross product.
To get the segment of the line lying in the image, the crossing points with the left and right borders of the image is calculated. The left point should be (x0, ?, 1) and the right point (x1, ?, 1),  both in homogeneous coordinates, such that

\begin{equation}
\label{eq:left}
   y_l = - (l1 x0 + l3 )/ l2
\end{equation}

\begin{equation}
\label{eq:right}
   y_r = - (l1 x1 + l3) / l2
\end{equation}

Given the segment, it is plotted on a mask with a determined margin, i.e., 10 pixels of thickness (`cv2.line`).

Up to now, for each point in the first image, all points in the second image corresponding to edges and lying in the line should be considered. But an additional requirement is introduced: <u>maximum disparity</u>. Only points (in the second image) that are close to the original index (in the first image) - that is, within a maximum disparity - are going to be analysed. In this project, a maximum disparity of 20 pixels was allowed.

*In addition, another restriction was tried. Points in the second image that had already found a match in the first image were excluded for posterior matches under the hypothesis that correspondences were 1:1. Ideally, the analysis would have speed up the processing and more accurate matches would have been obtained. Yet, the performance was worsen. Indeed, correspondes are not necessarily 1:1 so this restriction was removed from the reconstruction.*

Next, points in the second image (and their neighbourhood), corresponding to edges, lying within the line, and with a predefined maximum disparity are compared to the point in the first image (and its neighbourhood). To compare the possibilities, both kernels (each point and its neighbourhood, i.e., 11x11) are transformed to HSV and the mean squared error is computed. The match with the smallest error is chosen.

Having chosen two points, one on each image, they are back-projected to the 3D scene. Using the centers of both cameras and the two 3D points, <u>triangulation</u> is performed. Two rays are formed and the (approximated) intersection between both gives the 3D position of the matching 2D points. 

One ray is formed joining the center of one camera with the reference point in said camera; another ray is formed joining the center of a second camera with the corresponding point in the second camera. Finally, the crossing point between both rays - the desired 3D point - is approximated through least squares error.

The parametric equation of the first ray, *R = (r1, r2, r2)'*, is:

\begin{equation}
\label{eq:ray1}
   R_1 = P_1 + s V_1
\end{equation}
        
where *P1 = (p1, p2, p3)'* is the point in the reference image and V1 is the vector director joining the center of the first camera with point in said camera (i.e., *V1 = CP1 = P1 - C = (p1, p2, p3)' - (c1, c2, c3)'*).

Equivalently, the parametric equation of the second ray is:

\begin{equation}
\label{eq:ray2}
   R_2 = P_2 + t V_2
\end{equation}

Hence, the interesction between the two rays is given by:

\begin{equation}
\label{eq:interesection}
   (x, y, z)' = P_1 + s V_1 = P_2 + t V_2
\end{equation}

that is, 

\begin{equation}
\label{eq:mse-intersection}
  [V_1, -V_2] (s, t)' - (P_2 - P_1) = 0
\end{equation}

Yet, the rays do not necessarily cross, and the equation is not necessarily solvable. Hence, a least squares error problem can be posed, such that,

\begin{equation}
\label{eq:mse}
  Ax - b = 0
\end{equation}

where *A = [V1, -V2] (3-by-2)*, *x = (s, t)'* (2-by-1, and *b = P2 - P1* (3-by-1).

After computing *s* and *t* we can obtain the intersection using either equation of the lines. Yet, because the approximation is not necessarily exact, solutions might differ. The mean between the solutions is therefore returned, and the distance between both solutions (i.e., both points) is given as an error metric. Finally, triangulations with an error below a threshold (of 30), are projected to the 3D scene (`HAL.project3DScene`) and plotted (`GUI.ShowNewPoints`), using the color from the first image.
