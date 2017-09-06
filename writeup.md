## Writeup: Advanced Lane Lines

### This is a writeup for Project 4 of Udacity's Self Driving Car Nanodegree titled "Advanced Lane Lines". In this project uses advanced technique such as camera calibration, image gradients, color spaces and perspective transform to vastly improve upon basic techniques that were used for lane finding in project 1. 

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

[image1]: ./output_images/removing_distortion.PNG "Undistorted"
[image2]: ./output_images/removing_distortion_test_images.PNG "Undistorted2"
[image3]: ./output_images/x_y_gradient_threshold.PNG "X Y Gradient Threshold"
[image4]: ./output_images/magnitude_gradient_threshold.PNG "Magnitude Gradient Threshold"
[image5]: ./output_images/directional_gradient_threshold.PNG "Directional Grandient Threshold"
[image6]: ./output_images/combined_gradient_threshold.PNG "Combined Grandient Threshold"
[image7]: ./output_images/s_channel_threshold.PNG "S Channel Threshold"
[image8]: ./output_images/combined_gradient_color_threshold.PNG "Combined Grandient Color Threshold"
[image9]: ./output_images/straight_line_with_src_points_for_perspective_transform.PNG "Src Points for Perspective Transform"
[image10]: ./output_images/warp_example.PNG "Example Warped Image"
[image11]: ./output_images/fit_poly.PNG "Identifying Lanes and Fitting Polynomial"
[image12]: ./output_images/draw_lanes.PNG "Drawing lanes on original image"
[video1]: ./output_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cells 1 through 6 of the IPython notebook located in "advanced_lane_lines.ipynb".  

I start by visualizing how the chessboard images taken by the camera look, followed by finding and drawing the chessboard corners on the original image. I then prepare "object points", which will be the (x, y, z) coordinates of the chessboard corners in the real world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

I then write a function that can be called to undistort any given image in code cell 5. 

### Pipeline (few test images at a time)

#### 1. Provide an example of a distortion-corrected image.

In code cell 6, I try the undistortion out on all the test images, here are couple of examples:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

In code cell 7 I provide function to take magnitude of gradient i.e. Sobel along both axes and apply given threshold. Here's one example:

![alt text][image3]

In code cell 8 I provide function to take gradient (Sobel) along given axis and apply given threshold. Here's one example:

![alt text][image4]

In code cell 9 I provide function to take direction of gradient and apply given threshold. Here's one example:

![alt text][image5]

In code cell 11 I provide a fuction to take combination of x, y, magnitude and directional gradients. Here are couple of examples on test images. As can be seen, the yellow line on brighter road in first image is completely missed by these grandients.

![alt text][image6]

In code cell 12 I provide a fuction to take the S channel from HSL color space representation of the image. Here's how thresholded S channel looks for first test image. It can be seen that the lost yellow line is correctly retrieved by the S channel

![alt text][image7]

I used a combination of color and gradient thresholds to generate a binary image in code cell 13.  Here's an example of my output for this step. 

![alt text][image8]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in the 15th code cell of the IPython notebook.  The `warp()` function takes as inputs an image (`img`), and uses source (`src`) and destination (`dst`) points computed and displayed in code cell 14.  I chose to hardcode the source and destination points in the following manner:

![alt text][image9]

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 730, 472      | 1000, 0        | 
| 1043, 671     | 1000, 700      |
| 277, 671      | 250, 700      |
| 556, 472      | 250, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image10]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used histogram and sliding window method to find lane pixels and then fit my lane lines with a 2nd order polynomial using `numpy.polyfit` function, resulting in image like this:
(see code cell 21 fucntion `sliding_window_fit_poly` of the Ipython notebook)

![alt text][image11]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cell 28 and 29 of the IPython notebook in fuction `compute_radius` and `compute_position` respectively. For computing radius of road in meters, I fit the polinomial again, this time multiplying the already computed fit with meters per pixel in both x and y direction. Looking at the Destination points I use for my `warp` function, for x axix, I'm mapping 3.7 meters of road width (using US standard width) to 750 fixels (1000 - 250). Also assuming each dotted line is 3 meters long I'm mapping about 12 meters of road to 700 vertical pixels. I use similar logic to also find the position of the vehicle w.r.t center. I'm assuming that the vehicle is in center in the straight line image used for calculating source and desination pixels for `warp` function.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 31 of the IPython notebook in the function `draw_lanes()`.  Here is an example of my result on a test image:

![alt text][image12]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Functions used in order are:
``` python

    # Step1 - undistorting the image
    undistorted = undistort(image)
    
    # Step2 - color and gradient thresholding the image
    thresh = combined_color_grad_threshold(undistorted)
    
    # Step3 - perspective transforming the image
    warped = warp(thresh)
    
    # Step4 - fitting polynomial to the image
    left_fit, right_fit, left_lane_inds, right_lane_inds = sliding_window_fit_poly(warped)
    
    # Step5 - printing curvature of road and position of vehicle on the image
    left_radius, right_radius = compute_radius(warped, left_fit, right_fit)
    position = compute_position(warped, left_fit, right_fit)
    draw_curvature_position(image, left_radius, right_radius, position)
    
    # Step6 - drawing lanes on the original image
    result = draw_lanes(image, warped, left_fit, right_fit)
```

Here's a [link to my video result](./output_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 

I feel fairly confident about coming up with accurate camera matrix and being able to undistort the images taken by camera. I also feel ok about the perspective transform although I would still play around with the hardcoded values if the lanes are curvier than the ones in project_video.

I feel that if I have to pursue this project further I would work on the combination of gradient and color thresholding. I think that can be made more robust to work under all lighting conditions and road surfaces. The code for example doesnt do as good on the challenge videos.
