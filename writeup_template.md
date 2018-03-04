## Advaced Lane finding

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

[image1]: ./reference_images/original.png "Original"
[image8]: ./reference_images/undistorted.png "Undistorted"
[image2]: ./reference_images/undistorted_lanes.png "Road Undistorted"
[image3]: ./reference_images/binary_threshold.png "color threshold"
[image33]: ./reference_images/gradient_threshold.png "gradient threshold"
[image4]: ./reference_images/warped_lanes.png "Warp Example"
[image5]: ./reference_images/plotted_lines.png "Fit Visual"
[image6]: ./reference_images/lane_highlight.png "Output"
[video1]: ./test_videos_output/outvid.mp4 "Video"
[video2]: ./test_videos_output/outvid4.mp4 "Video"
[image10]: ./measurement_images/lane_lengt.png "length"
[image11]: ./measurement_images/lane_length.png "length"
[image12]: ./measurement_images/lane_length.png "length"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "image_pipeline.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]
![alt text][image8]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used only color thresholding to generate a binary image (thresholding steps in 4th code cell in "image_pipeline.ipynb").I used the LAB colorspace to easily detected the yellow colored lanes .I also attempted generating a binary image using gradient thresholding but found it to bee too noisy and decided to not use it.  Here's an example of my output for this step. 

![alt text][image3]
![alt text][image33]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is also located in the "image_pipeline.ipynb" file in the fourth code cell, The algorithm defines two sets of points: destination and source points , which are then used to calculate the perspective transform matrix that will be used to rectify road images.  I chose to hardcode the source and destination points in the following manner:

```python
image = cv2.copyMakeBorder(image,0,0,320,320,cv2.BORDER_CONSTANT,value=(0,0,0))
src = np.float32([[0,720],[863,450],[1057,450],[1920,720]])
dst = np.float32([[0,720],[0,0],[1920,0],[1920,720]])
```
I also padded the image from the right and the left because the 'cv2.getPerspectiveTransform(src, dst)' method takes only 4 points for both the destination and source , and I needed the source points to cover a large area of the road. 

I verified that my perspective transform was working as expected by testing the points on the straight lane images and used matplotlib cursor coordinates tool to verify that the warped lane lines appear parallel .

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used search windows to identify points along both left and right lane lines. and used the found points to generate a second order polynomial to represent the curvature of the lane lines. the plotted polynomial is shown in the "image_pipeline.ipynb" code cell number 5.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 7th code cell in "image_pipeline.ipynb" . I calculated the radius of curvature for both left and right lanes and defined the curvature of the lane as the average of both left and rigt line curvatures.
I also used sanity check by verifying that both curvatures are approximately parallel.

I measured the averages length of the dashed lane lines in pixels and found it to be 77 pixels:

![alt text][image10]
![alt text][image11]
![alt text][image12]

Then I set the dashed lane line length to be 3.66 meters according to the state of California traffic markings manual:
http://www.dot.ca.gov/trafficops/camutcd/docs/TMChapter6.pdf

Thus I was able to find a relation between pixels and meters and used that relation to convert image coordinates to world coordinates.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented the steps for highlighting the identified lane in the 7th code cell in "image_pipeline.ipynb"

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/outvid4.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Difficulties faced during implementation : 
1- Identifying lane lines using gradient thresholding was difficult , thus , was not used.
2- Badly computed frames resulted in anomalous road curvatures . This was solved by a sanity check previously discussed. 

Where the pipeline might fail : 
1- using  LAB colorspace is not the best option for significant variations in lighting . The project video shown here did not demonstrate much variation in lighting so the algorithm was successful, yet, It does not works as well in worse cases.

2- The pipeline does not work well for roads that are too curved . This can be addressed by improving the window search algorithm in the pipeline.

3- During rain , roads become reflective and this will sabotage the pipeline's ability to detect the lanes. Snow will also cause it to fail due to significant change in coloring.


