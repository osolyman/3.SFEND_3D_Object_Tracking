# SFND 3D Object Tracking

Welcome to the final project of the camera course. By completing all the lessons, you now have a solid understanding of keypoint detectors, descriptors, and methods to match them between successive images. Also, you know how to detect objects in an image using the YOLO deep-learning framework. And finally, you know how to associate regions in a camera image with Lidar points in 3D space. Let's take a look at our program schematic to see what we already have accomplished and what's still missing.

<img src="images/course_code_structure.png" width="779" height="414" />

In this final project, you will implement the missing parts in the schematic. To do this, you will complete four major tasks: 
1. First, you will develop a way to match 3D objects over time by using keypoint correspondences. 
2. Second, you will compute the TTC based on Lidar measurements. 
3. You will then proceed to do the same using the camera, which requires to first associate keypoint matches to regions of interest and then to compute the TTC based on those matches. 
4. And lastly, you will conduct various tests with the framework. Your goal is to identify the most suitable detector/descriptor combination for TTC estimation and also to search for problems that can lead to faulty measurements by the camera or Lidar sensor. In the last course of this Nanodegree, you will learn about the Kalman filter, which is a great way to combine the two independent TTC measurements into an improved version which is much more reliable than a single sensor alone can be. But before we think about such things, let us focus on your final project in the camera course. 

## Dependencies for Running Locally
* cmake >= 2.8
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* OpenCV >= 4.1
  * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors.
  * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory in the top level project directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./3D_object_tracking`.


# Final Project Report

## Task FP.1 : Match 3D Objects

* implement the method "matchBoundingBoxes", which takes as input both the previous and the current data frames and provides as output the ids of the matched regions of interest. Those matches are the ones with the highst numbers of keypoints.

* the idea is to loop through pevious and current boundingboxes after that looping through all matches, then checking if those bounding boxes contain those keypoints. If yes we add their ids to a created map. After that we loop through our new map to get back the best boxes, which are the boxes containing the maximum matches.

* the code can be found in FinalProject_Camera.cpp lines(225 - 229) && camFusion_Student.cpp lines(277 - 318)

## FP.2 : Compute Lidar-based TTC

* compute the time-to-collision for all matched 3D objects based on Lidar measurements alone.

* By using median filtering we find the closest distance to Lidar points within a predefined ego lane, where we iterate throud previous and current lidar points. After sorting the vectors of our new x coordinates we calculate the median of each previous and current x to compute the Time to Collision (TTC) of lidar points.

* the code can be found in FinalProject_Camera.cpp lines(260 - 269) && camFusion_Student.cpp lines(153 - 187)

## FP.3 : Associate Keypoint Correspondences with Bounding Boxes

* (implement -> clusterKptMatchesWithROI) in order to calculate the TTC Camera we need to find all keypoint matches that belong to each 3D object.

* By looping through the keypoint matches we check each keypoint, if it's within the ROI to add them to a created vector of those matches. then we calculate the mean distance. This mean distance will be used in another loop to remove the outliers and just keep the inliers.

* the code can be found in FinalProject_Camera.cpp lines(272 - 280) && camFusion_Student.cpp lines(138 - 184)

## FP.4 : Compute Camera-based TTC

* Once keypoint matches have been added to the bounding boxes, the next step is to compute the TTC estimate.

* computes the time-to-collision (TTC) based on keypoint correspondences in successive images by calculating distance ratios between matched keypoints and then estimating the TTC using the median distance ratio, accounting for outliers and ensuring robustness against estimation errors.

* the code can be found in FinalProject_Camera.cpp lines(272 - 280) && camFusion_Student.cpp lines(187 - 237)

## FP.5 : Performance Evaluation 1

* The Time-to-Collision (TTC) derived from Lidar data has been visualized in "Performance_Evaluation1.xlsx". Upon examining the plotting of the minimum x-coordinate (xmin) over time, it's evident that there are no abrupt changes in relative speed between the ego vehicle and the preceding one. The calculated Lidar TTC demonstrates a decreasing trend overall. However, anomalies are observed in frames 5 and 6, marked by sudden spikes in the plotted data.

* These anomalies may be attributed to inaccuracies in the time intervals. In our analysis, we utilized a constant frame rate, which assumes uniform intervals between frames. However, in reality, the frame rate of any sensor may vary slightly over time.
    
## FP.6 : Performance Evaluation 2

* According to the results provided in Performance_Evaluation2.xlsx, Frame 6 in FAST&BRISK Combination is showing a non accurate TTC Value of 53.0675. This could be because of Feature Descriptor Sensitivity: Different feature descriptors (such as FAST and FREAK) have varying sensitivities to different scene conditions (e.g., lighting changes, scale changes, viewpoint changes).

* in Combination (ORB&BRISK) Frames 6, 18, Combination(ORB&ORB) Frames 13, Combination(ORB&FREAK) Frames 16, Combination(ORB&SIFT) Frames 5, 13 are giving -inf as a result. this could happen because of reaching outliers in certain frames, this specially probable for certain detector/descriptor pairs which provide more unstable results.

* based on the results of almost all combinations of detectors / descriptors I would choose 1. (SHITOMASI & SIFT) 2. (SHITOMASI & ORB) 3. (FAST & ORB) as the best combination for computing TTC_Camera-based.
