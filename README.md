# CarND-AdvancedLaneLine-P2
Udacity Self-Driving Car Engineer Nanodegree Program<br>[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

# Overview

In this project, the goal is to write a software pipeline to identify the lane boundaries in a video from a front-facing camera on a car. The camera calibration images, test road images, and project videos are available in the project repository or in the repository folder included with the workspace.

<p align="center">
<img src="./docs/00_original_processed_img.png">
</p>

## Steps

The steps of this project are listed below. You can have a look at [Advanced_Lane_Lines.ipynb](Advanced_Lane_Lines.ipynb) for the code.
- Exploration Image
- Calibrating Camera
- Correcting for Distortion
- Color/gradient threshold
	- Applying Sobe
	- Magnitude of the Gradient
	- Direction of the Gradient
	- Combining Thresholds
	- HLS and Color Thresholds
	- Color and Gradient
- Perspective Transform
- Locate the Lane Lines
	- Peaks in a Histogram
	- Sliding Window
- Determine the lane curvature
- Inverse Transform
- Pipeline

The first thing you'll do is to compute the camera calibration matrix and distortion coefficients. You only need to compute these once, and then you'll apply them to undistort each new frame. Next, you'll apply thresholds to create a binary image and then apply a perspective transform.

My code for this project is publicly available and can be found here:<br> https://github.com/ednaldogoncalves/CarND-AdvancedLaneLines-P2


## Reflection

### Exploration Image

Lets have a look at a sample image, to be used for caliberation. Print out the size of the image.<br>
<p align="center">
<img src="./docs/01_example_exploration_img.png">
</p>


### Calibrating Camera

The first step in the project is to remove any distortion from the images by calculating the camera calibration matrix and distortion coefficients using a series of images of a chessboard.<br>
<p align="center">
<img src="./docs/02_calibration_camera.png">
</p>

### Correcting for Distortion

The first step in the project is to remove any distortion from the images by calculating the camera calibration matrix and distortion coefficients using a series of images of a chessboard.<br>
<p align="center">
<img src="./docs/03_correcting_distortion.png">
</p>

### Color/gradient threshold

#### Applying Sobe

Identify pixels where the gradient of an image falls within a specified threshold range.<br>
<p align="center">
<img src="./docs/04.1_sobe.png">
</p>

#### Magnitude of the Gradient

Identify pixels where the gradient of an image falls within a specified threshold range.<br>
<p align="center">
<img src="./docs/04.2_magnitude_gradiente.png">
</p>

#### Direction of the Gradient

When we play around with the thresholding for the gradient magnitude, we find what you might expect, namely, that it picks up the lane lines well, but with a lot of other stuff detected too. Gradient magnitude is at the heart of Canny edge detection, and is why Canny works well for picking up all edges.

In the case of lane lines, we're interested only in edges of a particular orientation. So now we will explore the direction, or orientation, of the gradient.<br>
<p align="center">
<img src="./docs/04.3_direction_gradient.png">
</p>

#### Combining Thresholds

Now consider how we can use various aspects of our gradient measurements (x, y, magnitude, direction) to isolate lane-line pixels. Specifically, we can use thresholds of the x and y gradients, the overall gradient magnitude, and the gradient direction to focus on pixels that are likely to be part of the lane lines.<br>
<p align="center">
<img src="./docs/04.4_combining_thresholds.png">
</p>

#### HLS, LUV, LAB Channel and Color Thresholds

Here we'll explore this a bit further to see why a color space like HLS, LUV, LAB can be more robust. Let's first take another look at some of the images.
Here we'll read in the same original image, convert to grayscale, and apply a threshold that identifies the lines.<br>
<p align="center">
<img src="./docs/04.5.01_hls_color.png"><br>
<img src="./docs/04.5.02_hls_color.png"><br>
<img src="./docs/04.5.03_hls_color.png"><br>
<img src="./docs/04.5.04_hls_color.png">
</p>

#### Color and Gradient

To combine what we know about color and gradient thresholding to get the best of both worlds.

At this point, it's okay to detect edges around trees or cars because these lines can be mostly filtered out by applying a mask to the image and essentially cropping out the area outside of the lane lines. It's most important that we reliably detect different colors of lane lines under varying degrees of daylight and shadow.

We can clearly see which parts of the lane lines were detected by the gradient threshold and which parts were detected by the color threshold by stacking the channels and seeing the individual components. We can create a binary combination of these two images to map out where either the color or gradient thresholds were met. Together, we applied the region of interest mask.<br>
<p align="center">
<img src="./docs/04.6.01_color_gradient.png"><br>
<img src="./docs/04.6.02_color_gradient.png">
</p>

### Perspective Transform

After manually examining a sample image, we extracted the vertices to perform a perspective transform. The polygon with these vertices is drawn on the image for visualization. Destination points are chosen such that straight lanes appear more or less parallel in the transformed image.

The transform and its inverse are computer, for use later. We then transform the binary image.<br>
<p align="center">
<img src="./docs/05_perspective_transform.png">
</p>

### Peaks in Histogram

#### Applying Sobe

After applying calibration, thresholding, and a perspective transform to a road image, we should have a binary image where the lane lines stand out clearly. However, we still need to decide explicitly which pixels are part of the lines and which belong to the left line and which belong to the right line.

Plotting a histogram of where the binary activations occur across the image is one potential solution for this.<br>
<p align="center">
<img src="./docs/6.1_histogram.png" width="636">
</p>

#### Sliding Window

I then perform a sliding window search, starting with the base likely positions of the 2 lanes, calculated from the histogram. I have used 10 windows of width 100 pixels.

The x & y coordinates of non zeros pixels are found, a polynomial is fit for these coordinates and the lane lines are drawn.

We can use the two highest peaks from our histogram as a starting point for determining where the lane lines are, and then use sliding windows moving upward in the image (further along the road) to determine where the lane lines go.<br>
<p align="center">
<img src="./docs/6.2_sliding_window.png" width="636">
</p>

#### Search from Prior

Using the full algorithm from before and starting fresh on every frame may seem inefficient, as the lines don't necessarily move a lot from frame to frame.
We need to do a blind search again, but instead you can just search in a margin around the previous line position, like in the above image. The green shaded area shows where we searched for the lines this time. So, once we know where the lines are in one frame of video, we can do a highly targeted search for them in the next frame.
This is equivalent to using a customized region of interest for each frame of video, and should help us track the lanes through sharp curves and tricky conditions.<br>
<p align="center">
<img src="./docs/6.3_search_from_prior.png" width="636">
</p>

### Determine the lane curvature

We have a thresholded image, where we've estimated which pixels belong to the left and right lane lines (shown in red and blue, respectively, above), and you've fit a polynomial to those pixel positions. Next we'll compute the radius of curvature of the fit.<br>
<p align="center">
<img src="./docs/7_lane_curvature.jpg" width="636">
</p>

### Inverse Transform

In this block of code we:

- Paint the lane area
- Perform an inverse perspective transform
- Combine the precessed image with the original image.<br>
<p align="center">
<img src="./docs/8_inverse_transform.png">
</p>

## Results:

### Pictures
Here some results on test images provided by Udacity:<br>

<p align="center">
straight_lines1.jpg<br>
<img src="./docs/9.2.01_teste_image.png">
</p>
<p align="center">
straight_lines2.jpg<br>
<img src="./docs/9.2.02_teste_image.png">
</p>
<p align="center">
test1.jpg<br>
<img src="./docs/9.2.03_teste_image.png">
</p>
<p align="center">
test2.jpg<br>
<img src="./docs/9.2.04_teste_image.png">
</p>
<p align="center">
test3.jpg<br>
<img src="./docs/9.2.05_teste_image.png">
</p>
<p align="center">
test4.jpg<br>
<img src="./docs/9.2.06_teste_image.png">
</p>
<p align="center">
test5.jpg<br>
<img src="./docs/9.2.07_teste_image.png">
</p>
<p align="center">
test6.jpg<br>
<img src="./docs/9.2.08_teste_image.png">
</p>

You can find the original pictures and the results in the folder ‘output_images’.


### Videos
Here some results on test videos provided by Udacity: You can find the video files on:<br>
<p align="center">
Result Project <br>
<img src="./output_videos/gifs/result_project_video.gif"><br>
<a href="./output_videos/result_project_video.mp4">Download video</a>
</p>

<p align="center">
Result Challenge<br>
<img src="./output_videos/gifs/result_challenge_video.gif"><br>
<a href="./output_videos/result_challenge_video.mp4">Download video</a>
</p>

<p align="center">
Harder Challenge<br>
<img src="./output_videos/gifs/result_harder_challenge_video.gif"><br>
<a href="./output_videos/result_harder_challenge_video.mp4">Download video</a>
</p>

## Discussion

In the video of the main Project, I did not have significant problems in detecting the lines of the road, because the video shows a road in basically ideal conditions, with very different lane lines and on a clear day, but I had problems of detection of the line on top of the bridge where the highway becomes lighter and soon there is a small shade of a tree.

In the first second of the Challenge video, the image processing function is confused with the imperfection of the road. Also, when it arrives in the second 3, because of the surplus caused by the viaduct, the image processing was not successful. Further on it continues to be lost because of the imperfection of the road. I did everything, but I could not solve this problem.

In the Harder Challenge video, the situation worsened, as the image processing function was completely lost in the curves and a white spot appears on the right and left side of the roadway mask. I also broke my mind to solve this problem, but unfortunately, I could not.

For me, this project was very challenging, but what I learned from it was that it's relatively easy to tune the software pipeline to work well under consistent road and weather conditions, but it's hard to find a single combination that produces the same result in any condition without adjusting the parameters. I have in mind, for further research, I plan to record some additional video streams that I drive myself under various conditions and continue refining my pipeline to work in more varied environments.