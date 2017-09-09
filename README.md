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

The project includes all required files and can be used to run the simulator in autonomous mode

My project includes the following files:
* Behavioral_Cloning.ipynb - the notebook with the data preprocessing and the model training
* model.py - the script used to create and train the model
* drive.py - for driving the car in autonomous mode
* model.h5 - containing the trained convolution neural network
* model.json - the architecture of the model as json
* writeup.md - summarizing the project and results
* video.mp4 - a video recording of the vehicle driving autonomously around the track

The images for camera calibration are stored in the folder called `camera_cal`. The images in `test_images` are for testing your pipeline on single frames.

The Advanced_Lane_Finding.ipynb file contains all the functions of the project and the main  pipeline I used for detecting the lane lins.
The sections are divided according to:
* Gathering the data
* Preprocessing the data by batchs
* Defining the Model
* Training the Model

## Output video

The output videos of can be found here:

[![video output](https://github.com/shmulik-willinger/behavioral_cloning/blob/master/readme_img/behavioral_cloning_simulator_track_2.gif)](http://www.youtube.com/watch?v=A1280XlpITA)


## Dependencies
This project requires **Python 3.5** and the following Python libraries installed:

- [Jupyter](http://jupyter.org/)
- [NumPy](http://www.numpy.org/)
- [Matplotlib](https://matplotlib.org/)
- [OpenCV](https://pypi.python.org/pypi/opencv-python#)
- [Glob](https://docs.python.org/3/library/glob.html)
- [Pickle](https://docs.python.org/3/library/pickle.html)
