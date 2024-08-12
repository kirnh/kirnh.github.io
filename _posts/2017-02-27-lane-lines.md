---
layout: post
title: Finding lane lines using CV and Python
---

The goal of this project was to write a python script that worked as a basic pipeline to find lane lines on the road. The input to the pipeline is an image of the road and the output contains lane lines being highlighted to show its detection.

The pipeline consists of 5 steps:

Input image -> Grayscale -> Gaussian blur -> Canny edges -> Region of interest -> Hough lines -> Output image

1. The input image is first converted to grayscale and blurred using Gaussian blur to smoothen the edges before using Canny edge detection to detect the edges present in our image.
<div style="text-align:center"><figcaption>Grayscale</figcaption><img src="https://thekhblog.wordpress.com/wp-content/uploads/2017/02/gray.jpeg"></div>
<div style="text-align:center"><figcaption>Gaussian Blur</figcaption><img src="https://thekhblog.wordpress.com/wp-content/uploads/2017/02/gaussian_blur.jpeg"></div>
<div style="text-align:center"><figcaption>Canny edge</figcaption><img src="https://thekhblog.wordpress.com/wp-content/uploads/2017/02/edges.jpeg"></div>

<br/>

2. Then to remove edges other than those part of the lanes, we define a region of interest and select edges present within this region before using Hough transform to identify lines from the binary image containing the canny edges.
<div style="text-align:center"><figcaption>Region of interest</figcaption><img src="https://thekhblog.wordpress.com/wp-content/uploads/2017/02/lane_region_edges.jpeg"></div>
<div style="text-align:center"><figcaption>Hough Lines</figcaption><img src="https://thekhblog.wordpress.com/wp-content/uploads/2017/02/hough_lines.jpeg"></div>

<br/>

3. To identify the full extent of the lanes on the road, we need to extrapolate the lines identified as Hough lines. To do this, we modify the draw_lines() function used within our Hough lines detection step in our pipeline to accomodate extrapolating. First, we classify the Hough lines as either belonging to the left lane or the right lane depending on their slope values, store these values along with the calculated y_intercepts of each of the lines before finding the average values (median is used as a measure of center because the distribution of the values are skewed). We are then left with an average value for the slope and y_intercept of left lane and right lane. These can be used to define our lane lines in the form [y = slope * x + y_intercept]. By intersecting these lines with two horizontal limit lines defined by y co-ordinate values (upper limit and lower limit depending on the region of interest), we get end points for our lane lines that extend to the full length of our region of interest. This is then used to draw the extrapolated lines and augmented on the original image as required.

<div style="text-align:center"><figcaption>Extrapolated Hough lines
</figcaption><img src="https://thekhblog.wordpress.com/wp-content/uploads/2017/02/extrapolated_lines.jpeg"></div>
<div style="text-align:center"><figcaption>Output image with detected lane lines
</figcaption><img src="https://thekhblog.wordpress.com/wp-content/uploads/2017/02/final.jpeg"></div>

The above pipeline works on images as well as videos (videos are just a bunch of images stacked together!).

[This](https://github.com/kirnh/CarND-LaneLines-P1.git) is a link to the GitHub repository that contains the code.

Some shortcomings:
- One potential shortcoming would be what would happen when the contrast between lane lines and the road becomes very less or too varying across the road image due to various reasons. If this happens, the Canny edge detector would pick up edges apart from lanes in our region of interest. This might make the Hough tranform detect the noise as lines (which could have slopes very far from those of lane lines). Our pipeline that finds the lane lines by averaging the values of slopes of detected Hough lines would be drasticaly affected by this noise.
- Another shortcoming could be identifying lane lines that form steep curves in the image. Because we classify lines as either belonging to left or right lane depending on whether the slope is positive or negative in our current pipeline, the edges detected from the curved lines could potentially be misclassified. Our pipeline that finds the lane lines by averaging the values of slopes of detected Hough lines would be drasticaly affected by this misclassification.

Suggested possible improvements to the pipeline:
- A possible improvement would be to include a more robust method of classifying lines as either belonging to the left lane or right lane. One way to do this is to use two regions of interest to identify left and right lanes seperately and not use slopes to classify the lines. And we could use slopes further down the line to remove any noise (lines not belonging to the lanes detected as Hough lines) and make the pipeline more robust.
- Also if we can use any method to identify curves other than just straight lines instead of using Hough transform to detect straight lines and replace the draw_line() function with a function that could draw a curve, it would make the pipeline better at handling curved lane lines in the image.
