## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/output_7_0.png "Undistorted"
[image2]: ./output_images/output_9_0.png "Road Transformed"
[image3]: ./output_images/output_13_0.png "Binary Example"
[image4]: ./output_images/output_17_0.png "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/output_27_1.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell named `Calibrate Camera` of the IPython notebook located in "project.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

To undistort a real image I simply used the `mtx` and `dist` output from `cv2.calibrateCamera` to call the `cv2.undistort` function.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_shift()` in the `Perspective Shift` section in the IPython notebook. 

First I use the source and destination corners to generate a transformation matrix `M` by using the `cv2.getPerspectiveTransform` function.

Then I warp the image using `cv2.warpPerspective` and the transformation matrix `M` from the previous step.

I selected the following source and destination points for my perspective shift:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 225, 720      | 275, 720      | 
| 590, 450      | 275, 0        |
| 690, 450      | 1025, 0       |
| 1025, 720     | 1025, 720     |

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for identifying and drawing lane-lines on an image is under the `Draw Curve Lines on Image` section of the notebook. Using the approach from the `Finding the Lines` video section, I used a histogram and sliding window to find those pixels which are, most likely, part of the lane lines. 

From that we take the left and right pixel points and apply a second order `polyfit` function to the points. After calculating these, we can draw them onto the perspective shifted image.

Then we can use the inverse transformation matrix to reverse the effects of perspective shift.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the `Calculate Radius of Curvature` section of the notebook. I took the arrays of x and y points for the left and right lines and applied the conversion values to take the data from pixel space to real space.

The actual radius of curvature was calculated using the code from the `Measuring Curvature` video section in Lesson 16.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

When testing out my implementation on the challenge video it failed quite harshly. The lane lines were jumping all around the place without a cohesive and consistent location. I believe this is due to rapidly changing lighting and environment conditions, like color of the road. 

In order to combat this quite large problems a couple things could be done:

- As mentioned in my first submision review, a deep learning approach could be implemented to more robustly handle shadows and color changes as the video progresses. The output labels could be threshold ranges, such that images with more exposure to the light have an overall higher threshold range for the S channel in the HLS çolor space.

- Averaging the overall brightness of an image could be determined using the L channel in the HLS color space to modify the thresholding ranges in a more hardcoded approach than deep learning. Less computationally intesnse than deep learning but also, most likely, less performant on average.

- More work could been done to smooth out the line changes from frame to frame by averaging together "good" lines in order to combat those outliers which were made quite obvious in the challenge video.

- Combining more color spaces and ditching the computationally intense process of using Sobel for gradients in the x direction. Utilizing more color spaces than just the S and L channels in the HLS color space would allow for a more generalized a cohesive approach for determing lane lines.
