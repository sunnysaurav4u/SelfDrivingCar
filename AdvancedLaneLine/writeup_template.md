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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "Advanced_lane_finding.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

[image1]: ./CarND-Advanced-Lane-Lines/Pipeline_output/Distortion_Removal.png

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
[image1]: ./CarND-Advanced-Lane-Lines/Pipeline_output/Distortion_Removal.png 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.
For finding the gradient threshold i used the sobel operator. I found out the gradient of image in both X and Y direction.
After this i deduced the threshold to the overall magnitude of the gradient, in both x and y using the given formulae. 
Also for finding the edges of a particular orientation we used the the direction/orientation, of the gradient.

After calculating all the gradient i put them together in one single condition to find the binary output.
for detecting the yellow lines i converted the orignal image into HLS image and find out only the 'S' component because it picks up the yellow lines very well.

After this i combined the both binary images produced by S_channel and sobel gradiend method to produce the final image.
Take the value of Sobel kernel size lage enough to produce the smoothening image but not that enough because it will blurr the image.

Here's an example of my output for this step.  (note: this is not actually from one of the test images)

[image1]: ./CarND-Advanced-Lane-Lines/Pipeline_output/Binary_image.png 

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `per_transform(image)`. The `per_transform(image)` function takes as inputs an image (`image`),] I chose the hardcode the source and destination points in the following manner:

```python
    src_bottom_left = [220,720]
    src_bottom_right = [1120, 720]
    src_top_left = [570, 470]
    src_top_right = [722, 470]
    
    
    dest_bottom_left = [320,720]
    dest_bottom_right = [920,720]
    dest_top_left = [320,1]
    dest_top_right = [920,1]
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570, 470      | 320, 1        | 
| 220, 720      | 320, 720      |
| 1120, 720     | 920, 720      |
| 722, 470      | 920, 1        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

[image1]: ./CarND-Advanced-Lane-Lines/Pipeline_output/Tranformed_image.png

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identified the lane line pixels in my code using `find_lane_pixels(binary_warped)` function providing it the `binary_warped` image which we get from previous function.
For this i used the sliding window method.for this we used the histogram method . we sum the binary pixel value along the 'Y' direction and wherever there is lane there sum is more so we get the maxima of the histagram. we seperate the left and right line using this histogram.
we also make sure to recentre our window whenever the value changes in either side upto a smal margin values.

[image1]: ./CarND-Advanced-Lane-Lines/Pipeline_output/Lane_detection.png

once the lane is detected using the sliding window we could use the left_fit value and right_fit value to produce the result by serarching from the previous output.

[image2]: ./CarND-Advanced-Lane-Lines/Pipeline_output/search_prior.png

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the readius of Curvature in `measure_curvature_real(binary_warped )` function using the given formulae and getting the left_fitx and right_fitx value from the `fit_polynomial(binary_warped)` function.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in  `wrapback(image,warped,M,left_fitx,right_fitx)`  function.  Here is an example of my result on a test image:

[image2]: ./CarND-Advanced-Lane-Lines/Pipeline_output/plot_line.png

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

I used the Sliding Window technique to identify the lines.
My pipeline is failing in the very steep turn in that case vehicle will go out of the road.

I think some also there is not  stablity in line detection ,it seems ike line is saking a bit somewhere.

I think all the points which i have decleared above can be a little bit implemented in correct way by choosing the appropriate value. doing good transformation.