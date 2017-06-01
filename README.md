# **Finding Lane Lines on the Road** 
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


![alt text][image1]
![alt text][image2]
![alt text][image3]

[image1]: ./examples/white.gif "white"
[image2]: ./examples/yellow.gif "yellow"
[image3]: ./examples/extra.gif "extra"

The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps.

* First, I converted the images to grayscale
* then Gaussian Blur is applied to the image to facilitate the canny edge detection.
* The third step is canny edge detection.
* Afterwards, a ploygon region is selected as the region of interest.
* The last step is hough transform to detect lines.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function:

* First I filter the end points (x1,y1,x2,y2) by their slope:
  * only the slope that are within a certain ranges gets populated into the input vector x and y for np.ployfit; this helps reduce the impact of some outliers returned from hough transform.
* after that, using np.ployfit, degree = 1, I calulate the slope and offset for the lane line on the left and right;
* once we have those parameters, we can decide the start / end point of the lane lines by
  * y = kx + b => x = (y - b) / k, where y can be the y at the top and bottom of the region of interest.
* last step is two cv.line calls which draw the line from top to the bottom.


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when the camera moves or input image size changes:

* now the region of interest selection has hardcoded parameters in it. so any change in the input image size or relative location of the lane within the image would cause the selection of ROI to fail.

Another shortcoming could be false edge / line within the ROI:

* as tested with the chanllege.mp4 video, the boundary of the hood, other objects in the ROI (tree, curb, light condition changes) would generate some other edges which will also be treated as lines after hough transform. I added some more logic to ignore those if the slope of the line is abnormal. But still some false lines remain occasionally.

One more thing is that in the current pipeline all the processing happens offline. And based on the average cpu time elapsed when processing the white / yellow / challenge video are 9.9ms, 10.1ms and 18.6ms respectively. The latency value may or may not be good enough for a real time application. Of course when processing real videos we can use some temporal info to reduce the computation. e.g only process the adjacent region of lane detected from the last frame.


### 3. Suggest possible improvements to your pipeline

Some possible improvement would be to:

* Avoid hardcoded region of interest detection. One thing I can think of would be not to crop the image in the first few frames and then determine the ROI when the lane line are detected.

* use temporal info to avoid some unnecessary computation. i.e one can assume the change of lane line location / shape happens continuously so in the next frame we can only search with in a certain range of pixels where update could be possible given current speed.

* take light condition into consideration; add some other parameters to gauge the current light condition and adjust parameters when transforms

* detect the color / dotedness of the lane line. They do have some meaning. this can be achieved by color selection and some more logic in the hough results to measure the continuity of the lines.



