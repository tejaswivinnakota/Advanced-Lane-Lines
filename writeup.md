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
[binary]: binary.png
[perspective_transform]: perspective_transform.png
[polyfit]: polyfit.png
[formula]: formula.png
[curvature]: curvature.png
[warp_back]: warp_back.png

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first few code cells of the Jupyter notebook project.ipynb.  

The OpenCV functions findChessboardCorners and calibrateCamera are used for image calibration. A number of images of a chessboard, taken from different angles with the same camera, comprise the input. Object points corresponding to the indices of internal corners of a chessboard and image points corresponding to the pixel locations of the internal chessboard corners determined by findChessboardCorners are fed to OpenCV calibrateCamera which returns camera calibration and distortion coefficients. These can then be used by the OpenCV undistort function to undo the effects of distortion on any image produced by the same camera. Generally, these coefficients will not change for a given camera. 

The image below depicts the results of applying undistort, using the calibration and distortion coefficients, to one of the calibration images:

![alt text][undistort]

---

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The image below depicts the subtle effects of applying undistort to one of the project images:
![alt text][undistort2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I explored several combinations of Sobel gradient thresholds and color channel thresholds. The final image is a combination of binary thresholding the R channel (RGB) and binary thresholding the result of applying the Sobel operator in the x direction on the undistorted image. I chose a combination of binary thresholding of the R channel (RGB) and binary thresholding of the L channel of the HLS color space. 

![alt text][binary]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is titled 'Step 4: Perspective Transform'. It includes a function called `warper()`.  The `warper()` function takes an image (`img`) as input.  I chose the source and destination points in the following manner:

```python
src = np.float32([[[ 610,  450]], 
                  [[ 680,  450]], 
                  [[ img_size[0]-300,  680]],
                  [[ 380,  680]]])

    # offset for dst points
offset = 350 
dst = np.float32([[offset, 0], 
                  [img_size[0]-offset, 0], 
                  [img_size[0]-offset, img_size[1]], 
                  [offset, img_size[1]]])
```

Assuming that the camera position will remain constant and that the road in the videos will remain relatively flat, I carefully selected points using one of the test images for reference. The image below demonstrates the results of the perspective transform:

![alt text][perspective_transform]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The identification of lane pixels and polynomial fitting is calculated in the code cell titled "Step 5: Detect lane panels and fit polynomials to get lane boundaries".

I applied a convolution, which will maximize the number of "hot" pixels in each window. A convolution is the summation of the product of two separate signals, in our case the window template and the vertical slice of the pixel image.

We slide the window template across the image from left to right and any overlapping values are summed together, creating the convolved signal. The peak of the convolved signal is where there was the highest overlap of pixels and the most likely position for the lane marker.

The image below demonstrates how this process works:

![alt text][polyfit]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is calculated in the code cell titled "Step 6: Determine curvature and position of vehicle with respect to center".

Using the left and right points found with the find_window_centroids() function, I was able to define curvature() which returned the position of vehicle, center of the image as well as the left and right Radius of curvature using the following equation:

![alt text][formula]

The image below demonstrates how this process works:

![alt text][curvature]


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Using warp_original function and left_fitx and right_fitx obtained after fitting the lines with a polynomial of the binary image, I projected those lines onto the undistorted image

![alt text][warp_back]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I used a step by step approach which helped me understand each part of the pipeline. Parameter tuning is tougher due to high number of parameters and combinations of them. A few possible approaches for making my algorithm more robust may include programmatically thresholding based on the resulting number of activated pixels, programmatically setting up src and dst points, averaging the lines and rejecting the right fit that deviates too much even if the confidence in the left fit is high, enforcing roughly parallel fits.
