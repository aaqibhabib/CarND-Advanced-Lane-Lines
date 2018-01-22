## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[calibration]: ../output_images/01-calibration.png "Undistorted"
[undistort_checkerboard]: ../output_images/02-undistort_chessboard.png "Checkerboard Transformed"
[undistort_lane]: ../output_images/03-undistort.png "Road Transformed"

[colorspace_exploration]: ../output_images/05-colorspace_exploration.png "colorspace_exploration"
[sobel_magnitude_direction]: ../output_images/06-sobel_magnitude_direction.png "sobel_magnitude_direction"
[sobel_x_y]: ../output_images/07-sobel_x_y.png "sobel_x_y"
[sobel_mag_dir_x_y]: ../output_images/08-sobel_mag_dir_x_y.png "sobel_mag_dir_x_y"
[undistort_lane]: ../output_images/03-undistort.png "Road Transformed"
[pipeline_all_test_images]: ../output_images/09-pipeline_all_test_images.png "pipeline_all_test_images"

[unwarp]: ../output_images/04-unwarp.png "unwarp"
[sliding_window_polyfit]: ../output_images/10-sliding_window_polyfit.png "sliding_window_polyfit"
[polyfit_from_centroid]: ../output_images/11-polyfit_from_centroid.png "polyfit_from_centroid"

[color_fit_lines]: ../examples/color_fit_lines.jpg "Fit Visual"
[radius_curvature]: ./radius-curvature.png "Radius Curvature"
[draw_data]: ../output_images/13-draw_data.png "draw_data"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in [`advanced-lane-finding.ipynb`](../advanced-lane-finding.ipynb).

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

The blank images are test images where the complete checkerboard was not found.

![alt text][calibration]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][undistort_checkerboard]
![alt text][undistort_lane]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at code blocks 15 through 19 in [`advanced-lane-finding.ipynb`](../advanced-lane-finding.ipynb)).  Below are examples from the different combinations I had explored. Ultimity, I chose to convert the RGB color image to HLS and use the S channel to as the input to the Sobel gradient calculations.

![alt text][colorspace_exploration]
![alt text][sobel_magnitude_direction]
![alt text][sobel_x_y]
![alt text][sobel_mag_dir_x_y]

Color transform and thresholding on test images.

![alt text][pipeline_all_test_images]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp()`, which appears in code block  13 and 14 in the file [`advanced-lane-finding.ipynb`](../advanced-lane-finding.ipynb).  The `unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([(555,464),
                  (737,464),
                  (228,682),
                  (1100,682)])
dst = np.float32([(450,0),
                  (w-450,0),
                  (450,h),
                  (w-450,h)])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 555, 464      | 450, 0        |
| 737, 464      | 810, 0        |
| 228, 682      | 450, 720      |
| 1100, 682     | 810, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][unwarp]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The functions `sliding_window_find_lines`,  `find_window_centroids`, and `polyfit_using_prev_fit`, which identify lane lines and fit a second order polynomial to both right and left lane lines; found in code blocks 22, 24, and 24 respectively.

*sliding_window_find_lines*
![alt text][sliding_window_polyfit]

*find_window_centroids*
![alt text][polyfit_from_centroid]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

We fit each lane line with a 2nd order polynomial. The image below is good visualization of the polynomial parameter.

![alt text][color_fit_lines]

 We can approximate the curvature of lane by the following equation:

![alt text][radius_curvature]

In python, this can be represented as:

```python
# y_eval => location to calculate curvature
curve_rad = ((1 + (2 * A * y_eval + B) ** 2 )** 1.5) / np.absolute(2 * A])
```

See code cell 27 for lane curvature calculations.



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 28 through 31 in the notebook. Function `draw_lane` projects the found lanes back into the original image, while `draw_data` adds lane curvature and center line information.

![alt text][draw_data]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](../project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Given more time, I would tune the thresholding parameters to work better in more of the test images. The values I chose perform well for simple, high contrast roads, but not so well in low contrast situations. As seen in the video, I do a decent job finding the solid left lane, while my right lane wobbles significantly. Although we could keep the car in the center of the lane given the information we have, we can apply smoothing to reduce the right lane jumps.

While building the pipeline, I spent significant time exploring the different color spaces and fiddling with different Sobel thresholds that would work with most of test images.