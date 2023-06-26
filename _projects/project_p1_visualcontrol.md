---
layout: page
title: visual control
description: programming a formula 1 to follow lines
img: # assets/img/12.jpg
importance: 1
category: robotics
---

This projects consists on programming a robot (i.e., a formula 1 car) to follow the road line in order to complete a circuit. The robot is expected to follow the line as perfectly as possible, that is, 1) without vibrating and 2) without short-cutting! Additionally, the robot should complete the circuits in the least time possible.


### Fri 10, March 2023 - Setting up the environment

To focus on the "intelligence" of the robot, we will be using a simulation provided by [Unibotics](https://unibotics.org). For that, I signed up (username: blancadelgado). 

Unibotics can be used through a remote server or locally.

For the use **remote** server, I needed to share my username with the professor so he could grant me access. Note that the simulation will only work if you [allow Unibotics to use insecure content in the browser](https://forum.unibotics.org/t/allow-insecure-content-in-the-browser/651).

For the use of my **local** resources, I first installed docker and then tried pullying both the lastest robotics-academy docker and the 3.2.13 version, with the next commands:

{% raw %}
docker pull jderobot/robotics-academy:latest 
{% endraw %}

{% raw %}
docker run --rm -it -p 8000:8000 -p 2303:2303 -p 1905:1905 -p 8765:8765 -p 6080:6080 -p 1108:1108 jderobot/robotics-academy
{% endraw %}

{% raw %}
docker run --rm -it -p 7681:7681 -p 2303:2303 -p 1905:1905 -p 8765:8765 -p 6080:6080 -p 1108:1108 jderobot/robotics-academy:3.2.13 --no-server
{% endraw %}

Note that to switch between the latest version and version 3.2.13, the already established connection had to be removed using the following commands (first to see the connections and then to remove the unwanted connection):

{% raw %}
docker container ls
docker rm -f name_of_connection
{% endraw %}

Either way, I was not able to start the simulation locally (a black window in Unibotics would be shown instead). Therefore, I will go with the remote environment for now.


### Thu 16, March 2023 - Programming the brain of my robot

To control a robot we need to first analyse the scene and then respond to changes in said environment; we do this in an infinite loop, sensing and acting.
Hence, the brain of my robot is going to be divided in two nuclei: the interpreter and the actuator.

The **interpreter** is going to identify the contour of the (red) line in the scene, to be followed. Then, it is going to find the top-most, left-most and right-most points of the line (Figure 1). Two key segments are going to be defined using the horizontal components of the points and normalized using the distance between the left and right points:

\begin{equation}
\label{eq:a}
    a = \frac{top - left}{right - left}
\end{equation}

\begin{equation}
\label{eq:b}
    b = \frac{right - top}{right - left}
\end{equation}

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/robotics/fig-MainPosition.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Figure 1. Top-most, left-most and right-most points of the line, reduced to the vertices of a triangle.
</div>

Given the three points, the deviation of the car is going to be approximated. 

If the car is going *straight*, the top-most point is going to fall in the grey region and the contribution of the *a* and *b* segments is going to be 50% each (actually, 50.8% and 49.2% respectively, since the center is slightly deviated). The straightness factor is going to be a real number in the range [0,1]. To obtain the factor, the contribution of the *a* segment is going to be used as reference and normalized with respect to the center (with a certain margin), and moved to the range [-0.5, +0.5], so that the contribution is going to be minimum (-0.5) at one side of the margin and maximum (+0.5) at the other side of the margin. Then, the absolute contribution in the range [0, +0.5] is going to be considered, since the interpretation at one side of the center (within the margin) is equivalent to that at the other side of the center (also within the margin). Finally, the factor [0,1] is going to be computed according to the absolute contribution in the range [0, +0.5], following the equation in Figure 2.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/robotics/fig-straightness.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Figure 2. Equation determining the straightness factor (y-axis), according to the normalised, absolute, contribution of segment *a* (x-axis).
</div>

If the contribution of the *a* segment is less than 50.8% (minus a margin), the car should go *left* and the amount of deviation is going to be given by the *b* segment. Contrarily, if he contribution of the *b* segment is less than 49.2% (minus a margin), the car should go *right* and the amount of deviation is going to be given by the *a* segment. With this approximation, we are going to define the left and right deviations (Figure 3).

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/robotics/fig-Deviations.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Figure 3. Posibilities for the deviation of the car: 1) left-most, 2) left, 3) right, 4) right-most.
</div>

Once interpreted the deviation of the car, two controllers are designed depending on the situation: a) if the car is going straight, b) if the car is turning right or left, c) if there was an error. 

- If the car is going straight, angular velocity is going to be null and linear velocity is going to be equal to a minimum velocity (i.e., 2) plus an additional contribution depending on the straightness factor. For more smooth transitions, 95% of the velocity in the previous step is going to be kept, and the new velocity is only going to be determining the remaining 5% (with the idea that several iterations will result in the ideal velocity).

- If the car is turning, a PDI controller is going to be defined to determine the angular velocity, and the linear velocity is going to be set depending on the computed angular velocity. The PDI controller is two have 3 components. The most predominant component (60%) is going to be proportional to the error (i.e., the deviation factor) in the current position. Then, the current and previous positions (with a memory of 8 frames), are going to give the integrative (i.e., sum of previous errors) and the derivative (difference between previous errors) components. The integrative component is meant to fix small, ongoing, errors (e.g., if the car is slowly deviating left). The derivative component is essential to avoid a vibrating car (e.g., if the car is going left it is rectified, but if it is being rectified, the effect is reduced to avoid bouncing to the right side).

The final result looks as follows:

{% include youtube.html id="1L82gEoz-cY" %}

It is needed 6:20 minutes to complete the turn using the *remote* simulators; way above the optimal timing. The car is slow due to the difficulting in adjusting parameters with three points; the deviations could not be precisely computed with three points, and a unique point should have been used for the (simpler) interpreation of the scene. 

In conclusion, two key take-ways were extracted from this first practice:
1. When working with simulators, it is necessary to come up with <u>a working solution as fast as possible</u>. Until the robot is not moving, we do not really know the possibilities we are going to be facing. 
2. When designing interpreters, <u>the simpler the better</u>. A more complex interpretation of the contex may have greater potential to obtain the ideal response but, in practice, the more complex the interpretation, the more parameters to manually adjust and the more caotic and frustrating the design of the actuator.





