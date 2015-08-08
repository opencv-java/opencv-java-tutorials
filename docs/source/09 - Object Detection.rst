=================
Object Detection
=================

.. note:: We assume that by now you have already read the previous tutorials. If not, please check previous tutorials at `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_. You can also find the source code and resources at `<https://github.com/opencv-java/>`_

Goal
----
In this tutorial we are going to identify and track one or more tennis balls. It performs the detection of the tennis balls upon a webcam video stream by using the color range of the balls, erosion and dilation, and the findContours method.

A project, made in Eclipse (Luna), for  Some screenshots of the running project are available in the screenshots folder.


What we will do in this tutorial
--------------------------------
In this guide, we will:
 * Insert a checkbox to select the Haar Classifier, detect and track a face, and draw a green rectangle around the detected face.
 * Inesrt a checkbox to select the LBP Classifier, detect and track a face, and draw a green rectangle around the detected face.

Getting Started
---------------
Let's create a new JavaFX project. In Scene Builder set the windows element so that we have a Border Pane with:

.. todo:: complete
