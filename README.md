# Advanced Lane Finding

[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

Overview
---
In this project, the goal is to write a software pipeline to identify the lane boundaries in a video, and to measure things like how much the lane is curving and where the vehicle is with respect to center. The algorithm will handle complex scenarios like curving lines, shadows and changes in the color of the pavement.

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/mapped_lane.png?raw=true)

The Goals
---
The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


## Details About the Files

The project includes all required files and can be used to run the pipeline on new images and video frames.

My project includes the following files/folders:
* Advanced_Lane_Finding.ipynb - the notebook with the main pipeline and all methods, including images demonstrating the output for each step.
* writeup.md - Details of the entire project, the steps, the results and conclusions
* test_images - the folder contain the test images and output images of the pipeline
* test_videos - the folder contain the test videos and output of the pipeline
* camera_calibration - this folder contains the images I took for camera calibration

## Output video

The advanced lane finding video with extended data display can be found on the link below:

[![video output](https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/project_video_extended.gif)](http://www.youtube.com/watch?v=k4Y-3ckoSmU)


## Dependencies
This project requires **Python 3.5** and the following Python libraries installed:

- [Jupyter](http://jupyter.org/)
- [NumPy](http://www.numpy.org/)
- [Matplotlib](https://matplotlib.org/)
- [OpenCV](https://pypi.python.org/pypi/opencv-python#)
- [Glob](https://docs.python.org/3/library/glob.html)
- [Pickle](https://docs.python.org/3/library/pickle.html)
