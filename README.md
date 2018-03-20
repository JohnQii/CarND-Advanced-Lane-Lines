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
    src = np.float32([[545, 460],
                    [735, 460],
                    [1280, 700],
                    [0, 700]])

    dst = np.float32([[0, 0],
                     [1280, 0],
                     [1280, 720],
                     [0, 720]])
    M = cv2.getPerspectiveTransform(src, dst)
    warped = cv2.warpPerspective(img, M, image_size)
    return warped, M
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 545, 460      | 0, 0        | 
| 735, 460      | 1280, 0      |
| 1280, 700     | 1280, 720      |
| 0, 700      | 0, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][bird-view]
Finaly, I only combine the color shreshold and perspective transform, but not gradient. It does better.
![alt text][bin_combin-pre]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:
1. Using the histogram of binary wraped img, then find the first lane line in the bottom column(peaks).
`histogram = np.sum(img[img.shape[0]//2:,:], axis=0)`
2. Using Sliding Window. I use the peak position as a starting point, then use slide window to find the lane line.
3. Fit a second order polynomial.
```python 
left_fit = np.polyfit(lefty, leftx, 2)
right_fit = np.polyfit(righty, rightx, 2)
```
Finally, the econd order polynomial is got as this:
![alt text][poly]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
1. position of the vehicle with respect to center:
```python
    #get vehicle position
    right_x = right_fit[0]*720**2 + right_fit[1]*720 + right_fit[2]
    left_x = left_fit[0]*720**2 + left_fit[1]*720 + left_fit[2]     
    center = abs((right_x + left_x))/2
    dis2center = 640 - center # 1280/2=640, if dis2center>0: on the left of center
    road_width_pixel =  abs(right_x - left_x)
    print("road_width_pixel: ", road_width_pixel)
    dis2center = dis2center/road_width_pixel * 3.7
```
2. I did this in lines # through # in my code in `Advance Line Detected.ipynb`
```python
def get_curve2_positon(left_fit, right_fit, ploty):
    #get vehicle position
    right_x = right_fit[0]*720**2 + right_fit[1]*720 + right_fit[2]
    left_x = left_fit[0]*720**2 + left_fit[1]*720 + left_fit[2]     
    center = abs((right_x + left_x))/2
    dis2center = 640 - center # 1280/2=640, if dis2center>0: on the right of center
    road_width_pixel =  abs(right_x - left_x)
    print("road_width_pixel: ", road_width_pixel)
    dis2center = dis2center/road_width_pixel * 3.7
    
    y_eval = np.max(ploty)
    quadratic_coeff = 3e-4 # arbitrary quadratic coefficient
    # For each y position generate random x position within +/-50 pix
    # of the line base position in each case (x=200 for left, and x=900 for right)
    left_y0 = left_fit[0]*720**2 + left_fit[1]*720 + left_fit[2]
    right_yo = right_fit[0]*720**2 + right_fit[1]*720 + right_fit[2]
    leftx = np.array([left_y0 + (y**2)*quadratic_coeff + np.random.randint(-50, high=51) 
                              for y in ploty])
    rightx = np.array([right_yo + (y**2)*quadratic_coeff + np.random.randint(-50, high=51) 
                                for y in ploty])
    leftx = leftx[::-1]  # Reverse to match top-to-bottom in y
    rightx = rightx[::-1]  # Reverse to match top-to-bottom in y
    # Fit a second order polynomial to pixel positions in each fake lane line
    left_fit = np.polyfit(ploty, leftx, 2)
    left_fitx = left_fit[0]*ploty**2 + left_fit[1]*ploty + left_fit[2]
    right_fit = np.polyfit(ploty, rightx, 2)
    right_fitx = right_fit[0]*ploty**2 + right_fit[1]*ploty + right_fit[2]
    ym_per_pix = 30/720 # meters per pixel in y dimension
    xm_per_pix = 3.7/road_width_pixel # meters per pixel in x dimension

    # Fit new polynomials to x,y in world space
    left_fit_cr = np.polyfit(ploty*ym_per_pix, leftx*xm_per_pix, 2)
    right_fit_cr = np.polyfit(ploty*ym_per_pix, rightx*xm_per_pix, 2)
    # Calculate the new radii of curvature
    left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
    # Now our radius of curvature is in meters
    curver = (left_curverad + right_curverad)/2
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
