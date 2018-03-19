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

[undistort]: ./output_images/undistort.png "Undistorted"
[test-undistort]: ./output_images/test-undistort.png "Road Transformed"
[color-thr]: ./output_images/bin_color.png "bin_color"
[bin_gradient]: ./output_images/bin_gradient.png "Binary gradient"
[bin_combin-pre]: ./output_images/bin_combin-pre.png "Binary bin_combin-pre"

[bird-view]: ./output_images/bird-view.png "Warp Example"
[poly]: ./output_images/poly.png "line Example"
[draw_lane-bound]: ./output_images/draw_lane-bound.png "Output"
[video1]: ./project_videoline.mp4 "Video"
## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

1. get the chessboard used cv2.findChessboardCorners: def get_chessboard(glob_dir='camera_cal/calibration*.jpg'). And return the objpoints, imgpoints
2. show the chessboard used cv2.drawChessboardCorners: def show_chess(imgs):
3. get the calibrate matrix used cv2.calibrateCamera :def calibrate_camera(objpoints, imgpoint, img):
4. calibrate the camera used cv2.undistort :  def calibrate_camera(objpoints, imgpoint, img):


![alt text][undistort]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][test-undistort]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `Advance Line Detected.ipynb`).  Here's an example of my output for this step.  
* only color threshold
![alt text][color-thr]
* only gradient threshold
![alt text][bin_gradient]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
def perspective_birds(img):
    image_size = (img.shape[1], img.shape[0])
    offset = 0
    source = np.float32([[490, 460],[810, 460],
                      [1250, 720],[40, 720]])
    destination = np.float32([[0, 0], [1280, 0], 
                     [1250, 720],[40, 720]])
    M = cv2.getPerspectiveTransform(source, destination)
    warped = cv2.warpPerspective(img, M, image_size)
    return warped, M
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 590, 460      | 0, 0        | 
| 810, 460      | 1280, 720      |
| 1250, 720     | 1250, 720      |
| 40, 720      | 40, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][bird-view]
combine the color shreshold && gradient threshold && perspective transform
![alt text][bin_combin-pre]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][poly]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `Advance Line Detected.ipynb`
```python
def get_curve_position(left_fit, right_fit, ploty):
    y_eval = np.max(ploty)
    left_curverad = ((1 + (2*left_fit[0]*y_eval + left_fit[1])**2)**1.5) / np.absolute(2*left_fit[0])

    right_curverad = ((1 + (2*right_fit[0]*y_eval + right_fit[1])**2)**1.5) / np.absolute(2*right_fit[0])    
    curver = (left_curverad + right_curverad)/2
    #get vehicle position
    right_x = right_fit[0]*720**2 + right_fit[1]*720 + right_fit[2]
    left_x = left_fit[0]*720**2 + left_fit[1]*720 + left_fit[2]     
    position = abs((right_x + left_x))/2
    dis2center = position - 640
    return curver, dis2center
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
ere is an example of my result on a test image:

![alt text][draw_lane-bound]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_videoline.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1. The threshold is very important! First time I can't find the right lane line, after modify
the color threshold and gradient threshold, it helps a lot.
2. Different color space for different environments.
