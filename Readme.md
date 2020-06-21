# **Finding Lane Lines on the Road** 

[//]: # (Image References)

[image0]: ./write_up/fig1.PNG "Overview"   
[image1]: ./write_up/pipeline.png "Pipeline"
[image2]: ./write_up/Lines.png "Line Aggregation"   

---
## Description
This repository contains my solution of the first Project of the Udacity Self-Driving Car Nanodegree.

The aim of this project is to find the lane markings in a video and vizualize the result in a resulting video.
My solution consists of a Jupyter Notebook.

![alt text][image0] 
### Reflection

### 1. Pipeline description

My pipeline consists of 4 basic steps which are explanied below. The output looks like this for the sample pictures.
 
![alt text][image1] 
#### Step 1: Grayscale, Canny Edge detection
The first step in my pipeline is to convert to grayscale. I did not make use of the colors of the image.
On the grayscale image, I used [Canny edge detection](https://en.wikipedia.org/wiki/Canny_edge_detector) to find edges. See column 1 in the grid image to see the output of this step.

#### Step 2: Cropping and vertical edge detection
The second step starts with limiting the image only to the region that is most probably the road. Then another edge detection is performed.
As the lane lines are typically rather vertical, I use a simple sobel mask to find only vertical edges in the 
masked Canny output image. This suppresses many detected lines that are not caused by lanes lines. See column 2 in the grid image to see the output of this step.

#### Step 3: Hough Transform
By performing the [Hough transform](https://en.wikipedia.org/wiki/Hough_transform) I detect points that lie on the same line in the image. The output can be seen in column 3 of the grid image.
Detected lines are drawn red here.

#### Step 4: Lane line detection
In order to draw a single line on the left and right lanes, I did not modify the draw_lines() function. I implemented the 
logic in the `find_two_lane_lines()` function. The output is seen in the last column of the grid image.
This last step tries to make up the right and left lane marking by the output of Hough transform,
which is mostly a set of many lines. Some of the lines are caused be the right lane line, some be the left, and some also
by other sources that are not lane markings.

My idea is, to remove those lines that are most probably not caused by lane markings, and cluster the rest by their heading.
I calculate a heading for each of the detected lines. If the heading is not typically a lane line I remove it from the set.
A typical lane line for me is either between 5.5 and 5.8 rad, or between 0.4 and 0.6 rad. After 
removing those lines that seem to be outliers, I cluster with [K-Means](https://en.wikipedia.org/wiki/K-means_clustering), where k = 2. This yields labels
for each line. Lines with the same label belong to the same lane marking. According to the position, I mark which lines
are the right lane marking, and which are the left ones. (Actually more for debugging reasons)

The most extreme positions from the found lines for each lane give me the position of the overall lane. This is shown
the following sketch:
![alt text][image2] 
The sketch shows how I choose the resulting lane line to be positioned by using the extreme values.
It seems a bit strange maybe, but works well in the videos.

In order to make the lines fit to the bottom of the image, I extrapolate downwards. I could have done the same upwards,
but I want to make it transparent how far my pipeline can "look" ahead and where the limit is.


### 2. Potential shortcomings with my current pipeline


One shortcoming is that the pipeline always expects to find both lane markings. If it fails to do so, k-means will still
try to find two groups, resulting in a messed up lane marking for one side.

Another shortcoming could be that if outliers are not removed sufficiently, the detected lane marking is also messed up.
The way I choose the lane markings to be aggregated, is not good with averaging out these lines ( as it does not average at all).
This leads to bad results in some situations in the challenge video.


### 3. Possible improvements to my pipeline

A possible improvement would be to limit detected lanes to a more strict area and tune the edge detection to be more sensitive.
Also making use of the color channels can be a good mean to remove false positives more early in the pipeline.

A much better approach is probably to transform the image into a birds eye perspective. The lane markings where probably much easier to
detect since they should be completely vertical. Also this would be great in curves but I had the idea too late.
