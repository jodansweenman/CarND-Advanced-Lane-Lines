**Advanced Lane Finding Project**

The goal of this project was to write an image pipeline to identify the lane boundaries in a video from a front-facing camera installed on a car.

[//]: # (Image References)

[image1]: ./camera_cal_results/found_corners.png "Found Chessboard Corners"
[image2]: ./camera_cal_results/undistored.png "Undistorted Chessboard"
[image3]: ./test_images/straight_lines2.jpg "Straight Road"
[image4]: ./camera_cal_results/perspective_road.png "Before Perspective Shift"
[image5]: ./camera_cal_results/warped_road.png "After Perspective Shift"
[image6]: ./camera_cal_results/thresholding.png "Threshold Image"
[image7]: ./camera_cal_results/rectangles.png "Sliding Window"
[image8]: ./camera_cal_results/right_fit.png "Right Lane Fit"
[image9]: ./camera_cal_results/left_fit.png "Left Lane Fit"
[image10]: ./camera_cal_results/fill_poly.png "Drawing the Shape"
[image11]: ./camera_cal_results/perShiftpoly.png "Shape after Perspective Shift"
[video1]: ./project_output.mp4 "Video Result"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "AdvLaneFinding.ipynb". This code stems directly from the lessons preceding this project.  

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, obj3d is just a replicated array of coordinates, and `obj3dpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img2dpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. I slightly modified the values from the lesson to help me differentiate the object space that they are played around with.

Here is one example of calibration image 10 with the corners found:

![alt text][image1]

I then used the output `obj3dpoints` and `img2dpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

### Pipeline

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]

For this, I hardcoded source and destination points much like the stop sign in the lesson preceding this project. Using the hardcoded source and destination lines, I shifted the lines into a rectangle on the new image. Below is an example of the before and after of the perspective shift.

![alt text][image4]
![alt text][image5]

#### 2. The actual code

The rest of the code is seen in the 3rd cell of the Jupyter notebook. I created a class, 'Line', to hold all of the functions for image processing. This class containeds functions for everything, including initialization, distortion, threshold, curvature calculations, fitting the lines, polgygon creation between the lines, and performing a perspective shift. The main function is call process_image and runs through the image pipeline. Most of the code is dervied from the information provided in the lessons.

##### Distortion Function

Lines 26-28 descrive the function used to distort the image. Examples of this are shown above.

##### Threshold Function

Lines 31-54 follow the same method from the lessons, and I used a combination of color and gradient thresholds to generate a binary image. Show below is an example of the output.

![alt text][image6]

##### Perspective Transform

Using the precalculated matrices, I shifted the image to a top-down perspective. This was described above with the example of the road perspective transform.

##### Lane Line Fitting

Two lane fitting functions were used to indentify pixels and fit a second degree curve. The first function, in lines 74-143 was used almost primary to find lane lines from scratch from an image with no previous information. It performs a histogram on the bottom half of the binary image, and uses the greatest values on the right and left as a primary guess for the bottom of the lane line. It utilizes sliding windows to step up the image, using the horizontal mean of the window's positive pixels to calculate the middle for the next window. After all of the pixels have been identified in the sliding window, a second degree curve is fit to the pixels. Below is an example.

![alt text][image7]

The next function in lines 146-169 fits a line from an image utilizing the previous image's curve as an initial estimate. Window boundaries are set with a margin dictated from the previous fit, and any pixel within the margin is used to create a second degree curve. Examples for the right and left lines are shown below.

![alt text][image8]
![alt text][image9]

The second function is only utilized if it is determined through the average value of the fit the first estimation of the line from scratch does not fit the curve good enough. If the second estimation is still bad, the first estimation is used, as it was on average, more of an accurate method.

##### Displaying Lane Offset and Curvature Information

Lane offset from center and cruvature are displayed on each image. The offset is calculated as the difference between the mean lane line psotion and the the lane center in pixels. This was transofrmed to meters using the ratio of pixels in the warped image lane width to an average lane size of 3.7m, which is the US Highway standard width. The curvature was calculated using the method described in the lessons on lines 57-71. The value displayed on the image is updated every 20 images to make it not change super rapidly, and the value displayed is the mean of 20 measurements for both offset and curvature.

##### Drawing The Shape onto the Lane

Using the curved calculated in the pipeline, a filled lane is drawn on the onto the warped image and shifted into the original perspective on lines 172-192. Below is an example of the result of this.

![alt text][image10]
![alt text][image11]


### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Check out the Video Below!

![alt text][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

This approach works pretty well for the project video, but fails considerably on the challenge video. If the edge detection was complimented by a different method, such as looking at the actual color of the lane line, this might help the pipeline better manage the challenge video.

Some checks to make sure that the overlayed shape is realistic would also help this pipeline, as there are n real checks to make sure that the shape is realistic and that the detected lane is actually uniform and close to a detected width of 3.7m.

I think the thresholding employed in my pipeline could use some work in order to be successful. Overall, with a little more work, this pipeline could be changed to actually work a bit better in more varying conditions.
