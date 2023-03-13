---
layout: page
title: visual control
description: programming a formula 1 to follow lines
img: # assets/img/12.jpg
importance: 1
category: robotics
---

This projects consists on programming a robot (i.e., a formula 1 car) to follow the road line in order to complete a circuit. The robot is expected to follow the line as perfectly as possible, that is, 1) without vibrating and 2) without short-cutting! Additionally, the robot should complete the circuits in the least time possible.


## Fri 10, March 2023 - Setting up the environment

To focus on the "intelligence" of the robot, we will be using a simulation provided by [Unibotics](https://unibotics.org). For that, I signed up (username: blancadelgado). 

Unibotics can be used through a remote server or locally.

For the use remote server, I needed to share my username with the professor so he could grant me access. Note that the simulation will only work if you [allow Unibotics to use insecure content in the browser](https://forum.unibotics.org/t/allow-insecure-content-in-the-browser/651).

For the use of my local resources, I first installed docker and then tried pullying both the lastest robotics-academy docker and the 3.2.13 version, with the next commands:

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


## Thu 16, March 2023 - Programming the brain of my robot


