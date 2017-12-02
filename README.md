## Advanced Lane Finding Project ##

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/road_undistorted.png "Road Transformed"
[image4]: ./examples/warped_straight_lines.png "Warp Example"
[image5]: ./examples/classifier_image.png "Color thresholding"
[image6]: ./examples/color_boundary_hue.png "Color boundaries in hue and saturation channels"
[image7]: ./examples/color_boundary_green_red.png "Color boundaries in red and green channels"
[image8]: ./examples/interactive.png "Preview of the results produced by the interactive UI"
[image9]: ./examples/drawPoly.png "Output"
[video1]: ./project_video_output.mp4 "Video"

#### [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

##### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first through fourth code cells of the IPython notebook located in "./Calibration.ipynb".  

I noticed that the number of edge points on the calibration images differs from image to image; because of that, first I create a list of visible board sizes.

After that for every calibration image I prepare "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

The code to undistort an image is actually just one line of code located in function `undistort()` in the third cell of the IPython notebook located in "`./ColorGradient.ipynb`".

This function is applied in the fourth cell of the same notebook and then the results are displayed.

Here's the result for the image above:

![alt text][image3]

The effects of undistortion can mostly be seen on objects around the edges of the image - the white car on the right and the car which the camera is installed on.

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `getPerspectiveTransform()`, which appears in cell 4 of the IPython notebook `ColorGradient.ipynb`.  

It returns the `perspectiveTransform` and `perspectiveTransformInverse` functions.
  
The source points are hardcoded, while the destination points are computed in a way that makes the horizontal center of the image to retain the x coordinate.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

(the bottom points appear at the very bottom of the warped image).

#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cells 5 through 7 in `ColorGradient.ipynb`).

For the color gradient, I wrote code that allows to click a bunch of image points in the notebook (using `%matplotlib nbagg` and attaching mouse click handlers to plots). Once clicks are recorded, the clicked coordinates are added to a datastructure called `clicks`, that I print using cell 6 and manually copy back into cell 5.

The recorded clicks are used to train a Decision Tree classifier that builds separating planes in the RGB color space. The classifier uses 3 classes:

* `pass`, represented in green: points that definitely belong to lane lines
* `lowpass`, represented in blue: points that are candidates for lane lines; these are filtered further using Sobel filters;
* `skip`, represented in red: points that are definitely not parts of lane lines.

Here's an example of the classification by the decision tree model of a road image, also showing 3 training points of different classes:

![alt text][image5]

Here are the decision boundaries over a couple of color palettes:

![alt text][image6]

![alt text][image7]

My implementation of gradient thresholding is in cells 7, 9, 11 and 13. In cell 13 I've implemented an interactive UI that allows to change configuration parameters and see how the pipeline results change.

Here's a preview of the images produced by the interactive UI:

![alt text][image8]

Top-left image shows the results of color thresholding.
Top-right shows the results of applying Sobel filters.
Bottom-right shows the histogram of the lower third of the image, used to determine starting positions of sliding windows.
Bottom-left demonstrates the results of the sliding windows algorithm.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Cell 10 contains the implementation of sliding window algorithm and fitting polynomials.

The code was mostly taken from the course materials; however, I tweaked it in order to:
* Use fits from the previous frame to determine starting position for sliding windows;
* Move the windows a bit according to the previously fitted polynomial;
* Detect the case when there are not enough points to produce a good fit and report this to the calling function.

Example output is shown above in section 3.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in cell 15, in a function called `convert_curvature`. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `drawPoly` in cell 10.  Here is an example of my result on a test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My approach was as follows:
* First, I realized that I could do perspectiveTransform before doing color and gradient thresholding. This allows to concentrate on processing the road alone.
* Next, I tried to maximize the value of color thresholding, by building the best possible decision boundary.
* After that, I kept tweaking the Sobel filter until no other lane lines get picked up instead of the lines I'm interested in.
* Unfortunately, even after I implemented brightness normalization (cell 4, `normalizeValue`), the lines would not get identified correctly in frames where there's little contrast between the line and the road.
* Because of that, I implemented moving average and removed low confidence fits (fits that only have points in the bottom part of the image).

Potential issues:
* Since the pipeline was only designed and tweaked for a specific combination of light/colors, changing weather conditions or road/lane line colors will likely pose a problem for this pipeline.

Improvements:
* It would be great to have an automatic way of evaluating the quality of lane detection. I tried to approximate that with inter-frame difference variance, but I was never able to get it down to one single number. Having a single number allows for automatic optimization of parameters.
* The heuristics used to define the lane lines are pretty obscure. It's well known that obscure and complex heuristics are better implemented with machine learning. Probably a convolutional neural network could help detect lines and might be easier to tweak.
