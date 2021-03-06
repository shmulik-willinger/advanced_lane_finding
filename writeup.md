# **Advanced Lane Finding**

**Building an advanced lane lines finding algorithm**

## The goals

The goals / steps of this project are the following:
* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

---

## Pipeline

 The first thing we'll do is to compute the camera calibration matrix and distortion coefficients. We only need to compute these once, and then we'll apply them to undistort each new frame in the pipeline. Next, we'll pick four points in a trapezoidal shape (similar to region masking) that would represent a rectangle when looking down on the road from above, and apply a perspective transform on the image. The next step is to apply thresholding by various combinations of color and gradient thresholds to generate a binary image where the lane lines are clearly visible. The next step is to calculate the polynomial and curvature using a sliding window that will slice each frame to horizontal lines and will look for the lines in them.

The main pipeline receiving an image and performs the following steps on it:
1. Applying the distortion correction
2. Applying a perspective transform to rectify binary image ("birds-eye view").
3. Creating a thresholded binary image
4. Using the sliding window to detect lane pixels and fit to find the lane boundary.
5. Calculating the polynomial to determine the curvature of the lanes
6. Drawing the detected lane boundaries back onto the original image

Let's go over the steps:

## Camera Calibration

Image distortion occurs when a camera looks at 3D objects in the real world and transforms them into a 2D image. This affect can change the apparent of size and shape of objects in the image. The first step in analyzing camera images, is to undo this distortion so that we can get correct and useful information out of them.

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/pinhole_camera_model.jpg?raw=true)

A chessboard is great for calibration because its regular high contrast pattern makes it easy to detect automatically, and we know how an undistorted flat chessboard looks like.
I start by taking multiple pictures of chessboard against a flat surface and preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/measuring_distortion.jpg?raw=true)

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction on a test image using the `cv2.undistort()` function and obtained this result:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/chessboard_undistorted.jpg?raw=true)

Here is an example of the distortion correction on an image from the car camera:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/road_undistorted.jpg?raw=true)

## Perspective Transform

Objects appears smaller the farther away they are from a viewpoint, and parallel lines seems to converge to a point. A perspective transform maps the points in a given image to different, desired, image points with a new perspective. We're interesting in the bird’s-eye view transform that let’s us view a lane from above.

I used the cv2.getPerspectiveTransform() and cv2.warpPerspective() functions to transform the apparent Z coordinate of the object points. Warp an image and effectively drag points towards the camera to change the apparent perspective.

I applied a perspective transform that zooms in on the farther object on the road images, and lets us change our perspective to view the same scene from different viewpoints and angles.

The code for my perspective transform includes a function called `perspective_transform()`, which takes as inputs an image, dynamically calculate the source and destination points and apply the cv2 functions on the image. I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart, to verify that the lines appear parallel in the warped image, whether they are straight or curved.

Here is an example of a bird's-eye view transform I did on an example image:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/perspective_transform.jpg?raw=true)

For the image above the transformation uses the following points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 670, 440      | 960, 0        |
| 1030, 666      | 960, 720      |
| 280, 666     | 320, 720      |
| 610, 440      | 320, 0        |

By looking at the transformed image we can see that both lines curve a little to the left.

## Thresholded binary image

To create a thresholded binary image we need to use various aspects of the gradient measurements like thresholds of the x and y gradients, the overall gradient magnitude, and the gradient direction to focus on pixels that are likely to be part of the lane lines. I used a combination from the following techniques:

1. Sobel - Using the Sobel operation to identify pixels where the gradient of an image falls within a specified threshold range.

2. Magnitude of the Gradient - The absolute value of the gradient is the square root of the squares of the individual x and y gradients.

3. Direction of the Gradient - The direction of the gradient is the inverse tangent (arctangent) of the y gradient divided by the x gradient (The direction of the gradient is much noisier than the gradient magnitude).

4. Saturation (S channel) - a measurement of colorfulness. As colors get lighter and closer to white, they have a lower saturation value, whereas colors that are the most intense, like a bright primary color (bright yellow), have a high saturation value. the S channel is doing a fairly robust job by picking up the lines under very different color and contrast conditions (while other selections sometimes look messy).

5. Value (R channel) - represent a way to measure the relative lightness or darkness of a color. The R channel performing well on the white lines.

6. Histogram equalization - improve the contrast of our images, in order to improve the white lines detection.

7. LUV (L channel) - attempted perceptual uniformity. The L channel performing well on the bright images.

The main thresholding step can be found in the notebook in the function thresholding() which responsible for the thresholded binary image procedure.

Here is an example of performing the above techniques on a test image:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/thresholding_samples.jpg?raw=true)

## Locate the Lane Lines

We now have a thresholded warped image and we're ready to map out the lane lines.
We need to decide explicitly which pixels are part of the lines, and also if they  belong to the left or right line.

I used a histogram to add up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so I assumed the two most prominent peaks in this histogram will describe the x-position of the base of the lane lines. From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.

The locate_lane_lines() method in the notebook is handling the lane lines locating step. I used 20 sliding windows on each image frame, with margin of 65 pixels for each window. I also set a minimum of 50 pixels for a group in the window to recenter the next window on the mean position.

Here is an example of this step on a test image:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/locate_lane_lines.jpg?raw=true)

One of the techniques I used in this step is to save the last 10 locations for a good lines detection. For difficult frames (where the algorithm couldn't find most of the line) I used the previous results to calc the mean of the lines.

## Measuring Curvature

 So now we have an image, where we've estimated which pixels belong to the left and right lane lines and we've fit a polynomial to those pixel positions. Now we need to compute the radius of curvature of the fit.

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/radius_of_curvature.jpg?raw=true)

For the images of this project we can assume the following:
1. The lane is about 30 meters long and 3.7 meters wide.
2. The lane width is 12 feet (3.7 meters) according to the U.S. regulations requirment
3. The dashed lane lines are 10 feet (3 meters) long each.
4. The camera is mounted at the center of the car, such that the lane center is the midpoint at the bottom of the image between the two lines

So we need to define conversions in x and y from pixels space to meters and to calculate the radius of curvature after correcting for scale in x and y.

The method measuring_curvature() in the notebook is handling this step, and the output of this procedure on a test image looks like this:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/measuring_curvature.jpg?raw=true)

## Tracking

We need to track things like where our last several detections of the lane lines were, and what the curvature was, so we can properly treat new detections.
The Line() class in the notebook will keep track of all the interesting parameters we measure from frame to frame.

## Drawing the lane

After we found the lines and computed the radius of curvature, we can draw the lane into the original frame.

First, we will draw the lane. I used Green color for the safety region and Yellow for the rest of the lane. Next, we will use the inverse perspective matrix to warp the lane back to original image space. The last part will be to combine the painted lane with the original image frame.

The function draw_lane() in the notebook is handling this step. The output looks as follow:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/lane_finding.jpg?raw=true)

## Testing the Pipeline

The output result for the test images can be found on the [test_images](https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/test_images) folder. Here is an example of the output:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/mapped_lane.png?raw=true)

I was satisfied that the image processing pipeline that was established to find the lane lines in images, also successfully processed the video. The standard output video can be found on the [test_videos](https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/test_videos) folder.
The advanced lane finding video with extended data display can be found on the link below:

[![video output](https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/project_video_extended.gif)](http://www.youtube.com/watch?v=hj4r5QzZNMY)

---

### Discussion

Some issues I have faced with during the project implementation:  

In this project I have experienced a lot with gradient, color channnels (RGB, HSL, HSV) and combinations of thresholding techniques. I've tweaked the threshold parameters to do the best job as possible of picking out the lines. Even though, in some frames the algorithm sometimes encounters difficulties with shadows, different colors in the pavement, and even if the lines are too bright.

The lanes in the challenge video are quite different in the following manner: There is a vertical line that cuts the track (the concrete) in the middle. The colors are much brighter, and for some frames (like in the tunnel) the algorithm dosen't detect the lines at all.

What could you do to make the pipeline more robust?
1. For the line detection I used an averaging over the last 10 frames. It turns out that although this is a good decision for most of the cases, for frames where the lane changes direction more often - It's better to use smaller number of frames to average with. I guess it can be invested with more dynamically.
2. Perspective transform section must be smaller when dealing with high curvature lanes.
3. Lighting conditions and shadows are significant for finding the path. We can find additional techniques for finding the lines with more accuracy

### I enjoyed working on the project and learned a lot from it
### Cheers!
