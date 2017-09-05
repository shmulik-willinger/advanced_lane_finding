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

## Camera Calibration

Image distortion occurs when a camera looks at 3D objects in the real world and transforms them into a 2D image. This affect can change the apparent of size and shape of objects in the image. The first step in analyzing camera images, is to undo this distortion so that we can get correct and useful information out of them.

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/pinhole_camera_model.jpg?raw=true)

A chessboard is great for calibration because its regular high contrast pattern makes it easy to detect automatically, and we know how an undistorted flat chessboard looks like.
I start by taking multiple pictures of chessboard against a flat surface and preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/measuring_distortion.jpg?raw=true)

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to a test image using the `cv2.undistort()` function and obtained this result:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/chessboard_undistorted.jpg?raw=true)

Here is an example of the distortion correction on an image from the car camera:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/road_undistorted.jpg?raw=true)

## Perspective Transform

Objects appears smaller the farther away they are from a viewpoint, and parallel lines seems to converge to a point. A perspective transform maps the points in a given image to different, desired, image points with a new perspective. We're interesting in the bird’s-eye view transform that let’s us view a lane from above.

I used the cv2.getPerspectiveTransform() and cv2.warpPerspective() functions to transform the apparent Z coordinate of the object points. Warp an image and effectively drag points towards the camera to change the apparent perdpective.

I apply a perspective transform that zooms in on the farther object on the road images, and lets us change our perspective to view the same scene from different viewpoints and angles.

The code for my perspective transform includes a function called `perspective_transform()`, which takes as inputs an image, dynamically calculate the source and destination points and apply the cv2 functions on the image. I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
Here is an example of a bird's-eye view transform I did on an example image:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/perspective_transform.jpg?raw=true)

In the image below we can see the source points on the original image.
By looking at the transformed image we can see that both lines curve a little to the left.

## Thresholded binary image

To create a thresholded binary image we need to use various aspects of your gradient measurements like thresholds of the x and y gradients, the overall gradient magnitude, and the gradient direction to focus on pixels that are likely to be part of the lane lines. I used a combination of the following techniques:

1. Sobel - Using the Sobel operation to identify pixels where the gradient of an image falls within a specified threshold range.

2. Magnitude of the Gradient - The absolute value of the gradient is the square root of the squares of the individual x and y gradients.

3. Direction of the Gradient - The direction of the gradient is the inverse tangent (arctangent) of the y gradient divided by the x gradient (The direction of the gradient is much noisier than the gradient magnitude).

4. Saturation (S channel) - a measurement of colorfulness. As colors get lighter and closer to white, they have a lower saturation value, whereas colors that are the most intense, like a bright primary color (bright yellow), have a high saturation value. the S channel is doing a fairly robust job of picking up the lines under very different color and contrast conditions (while other selections sometimes look messy).

5. Value (R channel) - represent a way to measure the relative lightness or darkness of a color. the R channel does rather well on the white lines

6. Histogram equalization - improve the contrast of our images, in order to improve the white lines detection.

Here is an example of performing the above techniques on the example image:

![]( https://github.com/shmulik-willinger/advanced_lane_finding/blob/master/readme_img/thresholding_samples.jpg?raw=true)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

In each case, I've tweaked the threshold parameters to do as good a job as possible of picking out the lines. Where we can really see a difference in results, however, is when we step to a new frame, where there are shadows and different colors in the pavement.
