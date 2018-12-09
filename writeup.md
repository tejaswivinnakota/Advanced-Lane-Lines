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

[undistort]: undistort.png
[undistort2]: undistort2.png
[unwarped]: unwarped.png
[perspective_transform]: perspective_transform.png
[polyfit]: polyfit.png
[histogram]: histogram.png
[search]: search.png
[drawlane]: drawlane.png

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first few code cells of the Jupyter notebook project.ipynb.  

The OpenCV functions findChessboardCorners and calibrateCamera are used for image calibration. A number of images of a chessboard, taken from different angles with the same camera, comprise the input. Object points corresponding to the indices of internal corners of a chessboard and image points corresponding to the pixel locations of the internal chessboard corners determined by findChessboardCorners are fed to OpenCV calibrateCamera which returns camera calibration and distortion coefficients. These can then be used by the OpenCV undistort function to undo the effects of distortion on any image produced by the same camera. Generally, these coefficients will not change for a given camera. 

The image below depicts the results of applying undistort, using the calibration and distortion coefficients, to one of the chessboard images:

![alt text][undistort]

---

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The image below depicts the subtle effects of applying undistort to one of the project images:
![alt text][undistort2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. I explored several combinations of Sobel gradient thresholds and color channel thresholds. I chose to use the L channel of the HLS color space together with the S channel or the gradient direction together with the gradient magnitude. 

![alt text][unwarped]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is title 'Perspective Transform'. It includes a function called `unwarp()`.  The `unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the source and destination points in the following manner:

```python
src = np.float32([(570,465),
                  (700,465), 
                  (325,675), 
                  (1075,675)])
dst = np.float32([(400,0),
                  (w-400,0),
                  (400,h),
                  (w-400,h)])
```

Assuming that the camera position will remain constant and that the road in the videos will remain relatively flat, I carefully selected points using one of the test images for reference. The image below demonstrates the results of the perspective transform:

![alt text][perspective_transform]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I developed functions sliding_window_polyfit and polyfit_using_prev_fit to identify lane lines and fit a second order polynomial to both right and left lane lines. The first of these computes a histogram of the bottom half of the image and finds the bottom-most x position (or "base") of the left and right lane lines. These locations were identified from the local maxima of the left and right halves of the histogram. The function then identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. 

The image below demonstrates how this process works:

![alt text][polyfit]

The image below depicts the histogram generated by sliding_window_polyfit:

![alt text][histogram]

This effectively "follows" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. 

![alt text][search]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is calculated in the code cell titled "Radius of Curvature and Distance from Lane Center Calculation" as below:

```
curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])
```

`fit[0]` is the first coefficient (the y-squared coefficient) of the second order polynomial fit
`fit[1]` is the second (y) coefficient
`y_0` is the the bottom-most y - the position of the car in the image
`y_meters_per_pixel` is the factor used for converting from pixels to meters

The position of the vehicle with respect to the center of the lane is calculated as below:

```
lane_center_position = (r_fit_x_int + l_fit_x_int) /2
center_dist = (car_position - lane_center_position) * x_meters_per_pix
```

`r_fit_x_int` and `l_fit_x_int` are the x-intercepts of the right and left fits, respectively

This requires evaluating the fit at the maximum y value i.e. 719px, the bottom of the image. Assuming that the camera is mounted at the center of the vehicle, the car position is the difference between these intercept points and the image midpoint .

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the code cells titled "Radius of Curvature and Distance from Lane Center Calculation" using draw_lane and draw_data functions in the Jupyter notebook project.ipynb. A polygon is generated based on plots of the left and right fits and warped back to the perspective of the original image using the inverse perspective matrix Minv. Below is a sample image:

![alt text][drawlane]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The problems I encountered were almost exclusively due to lighting conditions, shadows, discoloration, etc. Threshold parameters are tuned to get the pipeline perform well on the original project video. When I tried to extend the same pipeline to the challenge video, it tended to create problems with large noisy areas activated in the binary image when the white lines didn't contrast with the rest of the image.

A few possible approaches for making my algorithm more robust may include programmatically thresholding based on the resulting number of activated pixels and rejecting the right fit that deviates too much even if the confidence in the left fit is high, enforcing roughly parallel fits.