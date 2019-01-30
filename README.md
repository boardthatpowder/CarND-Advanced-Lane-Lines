## Advanced Lane Finding Project

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

[camera_cal1]: ./camera_cal/calibration1.jpg "Original"
[camera_cal2]: ./output_images/calibration1-undistorted.jpg "Undistorted"

[straight_original]: ./test_images/straight_lines1.jpg "Original"
[straight_undistorted]: ./output_images/straight_lines1-undistorted.jpg "Undistorted"
[straight_colored]: ./output_images/straight_lines1-color_binary.jpg "Colored Binary"
[straight_thresholded]: ./output_images/straight_lines1-thresholded_binary.jpg "Thresholded Binary"
[straight_masked]: ./output_images/straight_lines1-masked.jpg "Masked"
[straight_warped]: ./output_images/straight_lines1-warped.jpg "Warped"
[straight_lane_boundaries]: ./output_images/straight_lines1-lane_boundaries.jpg "Lane Boundaries"
[straight_result]: ./output_images/straight_lines1-result.jpg "Result"

[test1_original]: ./test_images/test1.jpg "Original"
[test1_undistorted]: ./output_images/test1-undistorted.jpg "Undistorted"
[test1_colored]: ./output_images/test1-color_binary.jpg "Colored Binary"
[test1_thresholded]: ./output_images/test1-thresholded_binary.jpg "Thresholded Binary"
[test1_masked]: ./output_images/test1-masked.jpg "Masked"
[test1_warped]: ./output_images/test1-warped.jpg "Warped"
[test1_lane_boundaries]: ./output_images/test1-lane_boundaries.jpg "Lane Boundaries"
[test1_result]: ./output_images/test1-result.jpg "Result"



## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is located within the code cells within the section titled _Camera Calibration'.

For each of the calibration test images I read in the image, convert to gray scale, then use the _cv2.findChessboardCorners()_ method to find the chessboard corners within the image using dimensions of (9,6) for (x,y).  Once I have found the corners, I add it to the colleciton _imgpoints_ which I later use with _cv2.calibrateCamera()_ to calculate the distortion coefficients and camera matrix.

Once calculated, I then use _cv2.undistort()_ to generate an undistorted view of the original image.

Example images: 

| Original | Distortion Corrected |
:---:|:----:
| ![original][camera_cal1] | ![undistorted][camera_cal2] | 

### Pipeline (test images)

#### 1. Provide an example of a distortion-corrected image.

Within the method _distortion_correction()_ I use _cv2.undistort()_ along with my previously calculated distortion coefficients and camera matrix to correct the distortion of images.

Example images:

| Original | Distortion Corrected |
:---:|:----:
| ![original][straight_original] | ![undistorted][straight_undistorted] | 
| ![original][test1_original] | ![undistorted][test1_undistorted] | 


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Within the _thresholded_binary_image()_ method I create a thresholded binary image using the following steps:
- convert undistorted image to HLS color space, and extract the S channel
- convert undistorted image to gray scale, and use this to generate a sobelx image
- using (different) thresholds for both the S and sobelx images, filter out uninteresting pixels
- combine the above to create a thresholded binary image

Example images (note that the thresholded image has been colorized to show the 2 different underlying combined images):

| Distortion Corrected | Thresholded Binary |
:---:|:----:
| ![undistorted][straight_undistorted] | ![thresholded][straight_colored] | 
| ![undistorted][test1_undistorted] | ![thresholded][test1_colored] | 

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Within the section titled _Perspective Transformation_ I have a method _perspective_transform_ which applies a persepctive transform to an image giving a camera matrix, along with another section of code I used to manually identify my source points and define my destination points.

I chose the hardcode the source and destination points in the following manner:

```python
top_right_point=(667, 440)
bottom_right_point=(1105, 720)
bottom_left_point=(205,720)
top_left_point=(611, 440)

src = np.float32([
    top_left_point,
    top_right_point,
    bottom_right_point,
    bottom_left_point
 ])
 
offset=350
h=img.shape[0]
w=img.shape[1]

dst = np.float32([
    [offset, 0], 
    [w-offset, 0], 
    [w-offset, h], 
    [offset, h]
])
```

Before applying the perspective transform, note that I also mask the image based on a region of interest I also manually defined.

Example images:

| Thresholded Binary | Masked | Warped |
:---:|:----:|:----:
| ![thresholded][straight_thresholded] | ![masked][straight_masked] | ![warped][straight_warped] | 
| ![thresholded][test1_thresholded] | ![masked][test1_masked] | ![warped][test1_warped] | 

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The methods _fit_to_lane_boundary()_ and _fit_to_lane_boundary_prior()_ are responsible for identifying the lane-line pixels.

The first method uses the sliding windows approach to identify lanes by:

- generating a histogram of the bottom half of the (warped thresholded) image
- identifying the x positions, to the left and right of the midpoint, where the maximum nonzero pixels are located
- using sliding windows (9 high in total), traverse up the image to each side of the midpoint, identifying the non zero pixels in the window
- fit a second order polynomial to the extracted left and right pixel positions

The second method is not used in the image processing pipeline, but is used in the video processing pipeline.  This method instead of using sliding windows, searches within a pre-defined margin of the last known fitted polynomial to descrease processing time.

Example images:

Warped | Masked | 
:---:|:----:
| ![warped][straight_warped] | ![lane_boundaries][straight_lane_boundaries] | 
| ![warped][test1_warped] | ![lane_boundaries][test1_lane_boundaries] |

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The method _measure_curvature()_ measures the curvature of the left and right detected lane lines.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The method _overlay_original_image()_ adds the detected lane lines to the original image by first inverse warping the detected polynomials using _cv2.warpPerspective()_, then constructing a new image using _cv2.addWeighted()_ to add the inverse warped polynomials to the original image.


| Original | Lane Detection |
:---:|:----:
| ![original][straight_original] | ![result][straight_result] | 
| ![original][test1_original] | ![result][test1_result] |

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](output_images/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

##### Perspective Transformation

The closer the distance to the horizon I use to perform the perspective transformation, the greater the possibly error due to the resolution of the image and/or surrounding objects.  This can be improved by reducing the distance (not ideal), increasing the resolution, or perhaps applying a variable region of interest mask to help separate the distant lane from surrounding objects.

##### Lane Changes

The fitting of the polynomials assumes that the left side of the lane is located to the left of the midpoint, and and the right hand side of the lane located to the right.  During changing lanes, this rule may no longer apply as one side of a lane passes from one side of the midpoint to another.  It is even possible that only one side of a lane may be visible during changing lanes.  This could be improved by removing the assumption that the vehicle is located near the midpoint of a lane, and adding intelligence to detect multiple lanes during changing.

##### Road Texture

A change in road materials, such as freshly laid new tarmac, may affect lane detection.  This could be improved by altering the threshold binary image generation to exclude perceived vertical lines in the road where line markings do not exist.



