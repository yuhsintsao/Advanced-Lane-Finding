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

[image0]: ./output_images/calibration.jpg "Calibration"
[image1]: ./output_images/undistorted.png "Undistorted Image"
[image2]: ./output_images/binary.png "Binary Image"
[image3]: ./output_images/bird_eye.png "Bird Eye View"
[image4]: ./output_images/color_channel.png "Color Channel Threshold"
[image5]: ./output_images/rectangle_poly_plot.png "Polynomial Fitting"
[image6]: ./output_images/poly_output.png "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "p-lane-line-finding.ipynb" (refer to the section named 'Camera Calibration with OpenCV').  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained results saved to the folder called 'camera_cal'. 

![Camera Calibration][image0]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in the section named 'Undistortion and Transformation'.  

![Undistortion][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is contained in the section named 'Edge Detection'.  

I used a combination of HLS (L and S) and LAB (B only) color thresholds to generate a binary image. The threshold I used for the L channel is (225, 255), for S channel is (180, 255), for B channel is (155, 200).

Here's an example of my output for this step. 

![Binary Image][image2]

![Image Processing][image4]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step is contained in the section named 'Perspective Transformation'. 

The code for my perspective transform includes a function called `corner_unwarp()`. The `corner_unwarp()` function takes an image (`img`) as its input. A fixed source (`src`) point and a fixed destination (`dst`) point are set in the following manner:

```python
    h, w = image.shape[:2]
    src = np.float32([(750. /1280.*w, 480. /720.*h),
                      (530. /1280.*w, 480. /720.*h),
                      (1130. /1280.*w, 720. /720.*h),
                      (190. /1280.*w, 720. /720.*h)])         
    dst = np.float32([(1100. /1280.*w, 0. /720.*h), 
                      (180. /1280.*w, 0. /720.*h), 
                      (1000. /1280.*w, 720. /720.*h), 
                      (190. /1280.*w, 720. /720.*h)])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Top-Down Image][image3]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For each frame, I found all non-zero pixels of the binary output and used a sliding window as a boundary to make sure all seleted pixels were locating within the desired searching area. Then I applied the sanity checking to avoid any false detection. I set three main criteria for the sanity checking: (1) the ratio of two radii of curvature in the sequential frames should be no larger than 5 or smaller than 0.2; (2) the distance of the top ends of two lane lines should be less than 900 pixels before the perspective transformation; (3) the difference of the radii of curvatures in the sequential frames should be less than 300 meters.

![Polynomial Fitting][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the radius of curvature, I set the meters per pixel in y dimension as 7 divided by 720 and the meters per pixel in x dimension as 3.5 divided by 1280. To calculate the position of the vehicle with respect to center, I used the difference between the middle x pixel of the image, i.e. 640, and the mean point of two lane lines' top ends. 

The code for this step is contained in the section named 'Polynomial Curve Fitting' and the function called `draw_lane_line`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![Output Display][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In the following cases, my algorithm fails to detect the true lane: 

(1) The asphalt Pavement colors are different from lane to lane and can wear out. In the case that one lane consists of the worn out colored area being removed and then being replaced with hot mix asphalt and the color desired, it makes it hard to detect the lane accurately. 

(2) The sun light may be hidden by the clouds or the highway bridge. In that case, the contrast of the image decreases and make it hard to detect the real lane lane. The problem also happens when the car runs under the shadow of the trees.  

To fix the problem, I come out with some possible solutions:

(1) Check the lightness/brightness index of the sequential images, if the value of the index has an apparent change, then correct the lightness/brightness.

(2) Calibrate the value of the pixel at the same position in the sequential images.



