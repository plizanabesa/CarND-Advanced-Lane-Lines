##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

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

[image1]: ./output_images/undist_cal.png "Undistorted calibration image"
[image2]: ./output_images/undist.png "Undistorted test image"
[image3]: ./output_images/binary_src.png "Binary image"
[image4]: ./output_images/binary_warped_dst.png "Binary warped image"
[image5]: ./output_images/sliding_windows.png "Sliding windows and fitted lines"
[image6]: ./output_images/result.png "Lane lines fitted in original image"
[video1]: ./output.mp4 "Output video"
[video2]: ./output_challenge.mp4 "Output challenge video"
[video3]: ./output_harder_challenge.mp4 "Output harder challenge video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for the camera calibration is contaned in lines 746-752 of the python script P4.py, in the main statement, and in lines 35-78. The first step is to get the objects points (3D) and image points (2D) by finding the chessboard corners in the calibration images. With this information it is possible to calibrate the camera (using `cv2.calibrateCamera()`) so as to undistort images taken with that camera. The following is an example of a undistorted calibration image

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
The distortion correction to one of the test images is seen in the following image:
![alt text][image2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I tried several combinations using gradients in both x and y axis, gradient magnitude, gradient direction and color thresholds (S channel and gray). The parameter thresholds and kernels also differed in each type of binary threshold. 

For the project and harder challenge videos I used a threshold that combined gradient in x with direction, and color with gray. For the challenge, the gray was used solely, since this channel was the only capable of detecting the subtle white lane marks in between all of the confusion genereated by the tire marks and the different pavement textures (asphalt and concrete). Here's an example of binary image:

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 85 through 93 in the file `P4.py`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  The source and destination points differed for the project and challenges (see lines 625-628 in `P4.py`)

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. The following is an example of a warped image with destination points drawn on it:

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To indentify the lane-line pixels I used first a sliding window algorithm (`get_lines_from_scratch()` in lines 256-348). If the lane-lines found were valid, then in the next frame I made a narrow search using as a basis the previous lane lines (`get_lines_with_previous_fit()` in lines 350-410). However, if the lane-lines found were not valid, I kept the previous lanes for at most 3 frames, point in which I used again the sliding window approach (lines 663 through 676).

To validate the lines I used three conditions (lines 518 through 571): (i) curvature difference, (ii) lane size at the bottom and top of the warped image and (iii) the lane size both at the bottom and top of the image are similar to the last valid lane size estimation.

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature and offset from the lane center is calcuated in lines 489-516 of `P4.py`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I plotted the lane lines back in the original image in lines 573-602 of `P4.py`. An example of result is shown as follows

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here there is a [link to my video result](./output.mp4), [link to my challenge video result](./output_challenge.mp4) and [link to my harder challenge video result](./output_harder_challenge.mp4)


---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

From trial and error in the three videos, I concluded that the binary thresholds cannot be the same for each type of road enviroment, hence they must be adapated dynamically. The same with the source and destination points. To mitigate part of this problem, in the harder challenge I applied a mask region, however, this region also needs to be dynamic. For instance, the region is likely to differ when going straight as to when taking a closed curve (as in the harder challenge video). The lane line validation parameters also differed in each type of enviroment.

A solution could be to have a road enviroment classifer, and hence adapt the lane finder parameters to that type of road. Or perhaps more easily, try several combinations of parameters with a very stringent line validation.

