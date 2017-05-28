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

[orig_cb]: ./img/orig_cb.png "Original Image"
[cor_cb]: ./img/cor_cb.png "Corrected Image"
[orig_road]: ./img/orig_road.png "Original Road Image"
[cor_road]: ./img/cor_road.png "Corrected Road Image"
[binary_road]: ./img/binary_road.png "Binary Road Image"
[warped_road]: ./img/warped_road.png "Road image after perspective transform"
[warped_fit]: ./img/warped_fit.png "Polynomials fit onto transformed image"
[orig_test]: ./img/orig_test.png "Original Road Image"
[result_test]: ./img/result_test.png "Road Image with lane drawn"
[numbers_road]: ./img/numbers_road.png "Road Image with lane drawn and info text"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook P4.ipynb. 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][orig_cb]
![alt text][cor_cb]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][orig_road]

In the fifth code cell of the notebook I first plot the above image after distortion correction with the following result:
![alt text][cor_road]

Close to the edges the change in the image is visible.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. These functions are defined in the fourth code cell of the notebook. In `combined_bin()` I first apply distortion correction before converting to HLS for extracting lane lines based on color and grayscale for gradient based detection. `grad_im()` creates a binary image using combined gradient tresholds of gradient with respect to x, magnitude and gradient direction. In `color_im()` I create a binary image using the saturation channel for color based detection. I also perform region of interest masking to exclude the surroundings. Here's an example of my output for this step. 

![alt text][orig_road]
![alt text][binary_road]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes the provided function called `warper()`, which can be found in the code cells following the label "Perspective Transform".  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner:

```python
ofs = 100
src = np.float32([(32,img_size[1]),(578, 450), (699, 450), (img_size[0]-32,img_size[1])])
dst = np.float32([[ofs, img_size[1]], [ofs, 0], [img_size[0]-ofs, 0], [img_size[0]-ofs, img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 32, 720      | 100, 720        | 
| 578, 450      | 100, 0      |
| 699, 450     | 1180, 0      |
| 1248, 720      | 1180, 720        |

I verified that my perspective transform was working as expected using the straight road images. I did not draw out the points as the `dst` points can easily be verified by their values and the `src` points were validated using the road image. The results are shown below.

![alt text][orig_road]
![alt text][warped_road]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial

I use `find_window_centroids()` to find points on the lane lines. In a first step the function looks at the bottom quarter of the image to establish starting positions for the lane search using a histogram calculated with `np.sum()`. After that `np.convolve()` is used over 12 vertical slices to find the points on the lane lines. The y-coordinate of each point is calculated from the window height in `lane_points()`. I then fit a second degree polynomial onto the points using `np.polyfit`. The code can be found in the code cell following the label "Lane Detection". An example result image is shown below.

![alt text][orig_road]
![alt text][warped_fit]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code to calculate the curve radius and vehicle position can be found following the label "Curvature and Vehicle Postion". The curve radius is calculated in `calc_cr()` by a unit conversion from pixels to meters followed by the analytical calculation of the radius using the polynomial's derivatives. The vehicle positions offset from the lane center is calculated in `calc_vehicle_pos()`. I assume a centered mounting of the camera so the offset is proportionate to the difference between the image width and the sum of the x-coordinate of the points on both curves with the same y-coordinate.

![alt text][numbers_road]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the code cell labeled "Output Visualization". I use `cv2.fillPoly()` to draw the lane in bird's eye view, then transform it back to the original image perspective using the `warper()` function with the `dst` and `src` arguments swapped. Finally I overlay the original image with the lane marking using `cv2.addWeighted`. Here is an example of my result on a test image:

![alt text][orig_test]
![alt text][result_test]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./vid_out.mp4) (vid_out.mp4 located in the repo)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I used the `Line` class to retain some state between frames and introduce filtering. I also implemented checks to to discard found lane lines when they started on the wrong side or were to far off from those found in the previous frame. 

This may cause unfavorable results when the car drives off the current lane, i. e. when switching lanes. To avoid this, only a certain count of found lane lines should be allowed to be discarded consecutively or a constant change in coefficients over several frames could be recognized and the checks modified accordingly.

