## Writeup

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

[image1]: ./undistort_calib_output.png "Undistorted"
[image2]: ./undistort_output.png "Undistorted"
[image3]: ./binary.jpg "Binary Example"
[image4]: ./warped_straight_lines.png "Warp Example"
[image5]: ./color_fit_lines.png "Fit Visual"
[image6]: ./output.jpg "Output"
[video1]: ./project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first and the second code cell of the IPython notebook located in "./line_detection.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in sections 'Apply gradient thresholds', 'Apply color thresholds' and 'Applying all thresholds').  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is included in the section "Perspective transform" in the iPython notebook line_detection.ipynb.  I used the undistorted image for test_images/test1.jpg as the source image.  I chose to hardcode the source and destination points in the following manner:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 275, 700      | 450, 700      |
| 400, 600      | 450, 600      |
| 950, 600      | 900, 600      |
| 1100, 700     | 900, 700      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto the source image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I implemented this step in the section "Sliding window to find the lane lines" in the iPython notebook line_detection.ipynb. 

In order to find the lane line pixels, I first plotted a histogram of where binary activations occur across the bottom half of the image. Once this was done, the highest peaks of the histogram in the left half of the image and the right half of the image were considered to be the position of the base of the left and right lanes respectively.

The next step was to consider 9 sliding windows moving upwards in the image to find the lane line pixels. A margin of 50 pixels was defined for each sliding window and the first sliding window at the bottom of the image was centered at the base of the lane found using the histogram in the previous step.

The non-zero pixels occurring within a sliding window were considered to be part of the lane lines. Whenever the number of non-zero pixels within a sliding window exceeded 50, the next sliding window is recentered based on the mean position of these pixels.

Once all these lane-line pixels were found, a quadratic polynomial x = Ay<sup>2</sup> + By + C was fit to both the left and right lane-line pixels using `np.polyfit` function.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the sections "Finding Radius of Curvature", "Finding offset" and "Showing radius of curvature and offset on lane detected image" in the iPython notebook line_detection.ipynb.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

At the end of the section, "Showing radius of curvature and offset on lane detected image", I saved an example output image in the file `output.jpg` which is shown below:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
