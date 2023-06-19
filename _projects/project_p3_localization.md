---
layout: page
title: localization
description: localizing a camera using beacons
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

First of all, we printed (in a standard printer): 1) a **calibration pattern** (Figure 1, right), that can be found in [OpenCV’s 
GitHub](https://github.com/opencv/opencv/blob/4.x/doc/pattern.png), and 2) a **marker** (Figure 2, left), that can be automatically generated at 
[ArUco markers generator!](https://chev.me/arucogen/)

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/robotics/aruco.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/robotics/pattern.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Figure 1. Left: ArUco marker (4-by-4, 100 mm per side). Right: calibration pattern (9-by-6, 25 mm per side).
</div>

Then, we created Python 3.10 Project (available in [GitHub](https://github.com/blancadelgadobonet/robotics.git)) using a MacBook Pro 2021 (Apple M1 Pro), that was comprised of one main file (`main.py`), 
two sets of functions (`calibration.py` and `localization.py`), and one figure (`aruco.png`).

The objective of the script was to **acquire real-time images** (either from the computer’s webcam or an iPhone plugged to the computer), 
**calibrate** the camera after capturing several images of a calibration pattern, **localizing** a marker (with a specific ID, set by the user) when 
present in said images and **displaying** the position of the camera with respect to the reference point of the marker. 

To **calibrate** a camera, images of a known pattern needed to be taken: detected 2D points of the image (in pixels) can be compared to known 3D 
points of the scene (in, e.g., millimeters), and the relation between 2D and 3D points can be extracted. To that means, the printed calibration 
pattern was captured by the camera (N=100 times, although less images can work); the bidimensional points of interest from the pattern were 
detected (using `cv2.findChessboardCorners`), displayed (using `cv2.drawChessboardCorners`, Figure 2), and matched to their corresponding 
three-dimensional points (generated using a custom function `tools.calibration.get_chessboard_points`); and the relation between all sets of 
captured 2D points and their known 3D points was approximated (using `cv2.calibrateCamera`). As a result, the intrinsic parameters of the camera, 
its distortion coefficients, and the root mean square-reprojection error was computed (Code 1).

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/robotics/eg-caligration.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="caption">
    Figure 2. Calibration pattern with detected points. 
</div>

\begin{equation}
\label{eq:intrinsics}
\begin{pmatrix} c & s \end{pmatrix} 
\end{equation}

Intrinsics:
 [[2233    0  729]
 [   0 2233  603]
 [   0    0    1]]

Distortion: [[ 0 -5  0  0 35]]
Root mean square re-projection error: 1.815

At this point, the camera was considered known, as the root mean square re-projection error was within an admissible margin (i.e., approximately 
below 2 pixels).

To localize the camera with respect to a known beacon, images of a said beacon (ID=4) were taken and the 2D corners of the beacon were detected 
(using `cv2.aruco.detectMarkers`) and matched to their 3D, real, pre-defined correspondences (mm_beacon_3D). Both 2D and 3D matches, along with the 
intrinsic parameters of the camera and its distortion were used to extract the extrinsic parameters of the camera (`localization.get_camera_position`): 
a perspective-n-point problem was solved with the given input to obtain the rotation and translation vectors (using `cv2.solvePnP`), the rotation 
vectors were transformed into a rotation matrix (using `cv2.Rodrigues`), and the center of the camera with respect to the beacon was computed (Equation 1).

center =  - (KR)^{-1} (Kt) 

Finally, to visualize the localization of the camera in 3D, the scene was set with an ArUco image setting the reference point of the scene (using a 
customed function, `localization.plot_scene`) and the real-time positions of the camera were plotted iteratively (using a custom function, 
`localization.plot_camera`).

Localizing the camera with respect to the beacon is interesting in two ways. On the one hand, localizing the camera allows localizing the robot, 
since the camera tends to be in a known position to the robot. On the other hand, localizing the beacon can be extended to localizing a certain point 
in a known map (if the position of the beacon with respect to a map is known). Hence, geometric autolocalization using beacons exploits its potential 
when the robot is in a known position with respect to the camera, the camera is in a known position with respect to the beacon, and the beacon with 
respect to the map: **the robot is located with respect to the map**.

*Last edit: Fri 16, June 2023*
