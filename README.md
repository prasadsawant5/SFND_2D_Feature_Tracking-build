# SFND 2D Feature Tracking

<img src="images/keypoints.png" width="820" height="248" />

The idea of the camera course is to build a collision detection system - that's the overall goal for the Final Project. As a preparation for this, you will now build the feature tracking part and test various detector / descriptor combinations to see which ones perform best. This mid-term project consists of four parts:

* First, you will focus on loading images, setting up data structures and putting everything into a ring buffer to optimize memory load. 
* Then, you will integrate several keypoint detectors such as HARRIS, FAST, BRISK and SIFT and compare them with regard to number of keypoints and speed. 
* In the next part, you will then focus on descriptor extraction and matching using brute force and also the FLANN approach we discussed in the previous lesson. 
* In the last part, once the code framework is complete, you will test the various algorithms in different combinations and compare them with regard to some performance measures. 

See the classroom instruction and code comments for more details on each of these parts. Once you are finished with this project, the keypoint matching part will be set up and you can proceed to the next lesson, where the focus is on integrating Lidar points and on object detection using deep-learning. 

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
2. Make a build directory in the top level directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./2D_feature_tracking`.

## Rubic
### MP.1
The idea is to implement a queue. Would have been easier if we could have used the queue APIs provided by c++. But since the requirement was to introduce a FIFO behavior in a vector, we limited its capacity to the buffer size and blindly pushed element if the size of the vector was less than the buffer size. If the vector was full then we popped the foremost element and moved rest of the elements to one position to the left, thereby creating an empty slot at the end of the vector which gets occupied by the new element.

### MP.2
A minor change in this implementation was that we introduced Enums as a selector for various detector algorithms on `match2D_Student.cpp` side. Then OpenCV's build-in detector classes were initialized with some parameters and scanned the whole image to detect keypoints. Also, the detection time was logged for performance evaluation. And the total detection time for our input ten images was logged as well at the end of the program.

### MP.3
For this project, we will focus more on the preceding vehicle, so we set up a certain region of interest around the middle of the whole image. We will only keep the key-point that is in this area to reduce noise and save computation power for following steps.

### MP.4
Descriptor is a vector of values, which describes the image patch around a keypoint. Similar to step 2, we implement a function to detection descriptors based on the input string. We still use OpenCV build-in descriptors (BRIEF, ORB, FREAK, AKAZE and SIFT) class with default parameters to uniquely identify keypoints. Similar to step 2, we log the descriptor extraction time for performance evaluation.

### MP.5
User can choose which matching method to use: Brute-force matcher(MAT_BF) or FLANN matcher(MAT_FLANN). Note that we need to convert our image to CV_32F type due to a bug in current OpenCV implementation if we choose FLANN matching.

### MP.6
Both nearest neighbor (best match) and K nearest neighbors (default k=2) are implemented. For KNN matching, we filter matches using descriptor distance ratio test to remove some outliers. In current implementation, only following match will be accepted:

`knn_match[0].distance < 0.9 * knn_match[1].distance`

### Benchmark (MP.7, MP.8, MP.9)

#### Keypoints in 10 images

| Detectors | Number of Key-points |
| :-------: | :------------------: |
| SHITOMASI |        13423         |
|  HARRIS   |         1737         |
|   FAST    |        17874         |
|   BRISK   |        27116         |
|    ORB    |         5000         |
|   AKAZE   |        13429         |
|   SIFT    |        13862         |



#### Matched keypoints in 10 Images

| Detectors\Descriptors | BRISK |  BRIEF  |      ORB      | FREAK | AKAZE | SIFT |
| :-------------------: | :---: | :-----: | :-----------: | :---: | :---: | :--: |
|       SHITOMASI       |  347  | **413** |      398      |  341  |  N/A  | 405  |
|        HARRIS         |  142  |   206   |      162      |  144  |  N/A  | 163  |
|         FAST          |  335  |   336   |      332      |  295  |  N/A  | 291  |
|         BRISK         |  304  |   314   |      266      |  292  |  N/A  | 279  |
|          ORB          |  346  |   267   |      347      |  327  |  N/A  | 364  |
|         AKAZE         |  341  |   392   |      345      |  353  |  343  | 348  |
|         SIFT          |  185  |   249   | Out of Memory |  195  |  N/A  | 294  |




#### Keypoint detection and descriptor extraction time consumption (in ms)

| Detectors\Descriptors |  BRISK  |    BRIEF    |      ORB      |  FREAK  |  AKAZE  |    SIFT    |
| :-------------------: | :-----: | :---------: | :-----------: | :-----: | :-----: | :--------: |
|       SHITOMASI       | 98.8398 |   82.6777   |    91.0227    | 328.525 |   N/A   |  180.6775  |
|        HARRIS         | 106.512 |   96.1124   |    108.656    | 338.423 |   N/A   |  151.436   |
|         FAST          | 12.7961 | **9.92533** |    12.1023    | 267.232 |   N/A   |  116.5025  |
|         BRISK         | 262.799 |   257.95    |    262.838    | 510.137 |   N/A   | 337.020527 |
|          ORB          | 53.0014 |   52.4011   |    58.3677    | 294.063 |   N/A   |  270.4355  |
|         AKAZE         | 387.531 |   383.136   |    378.456    | 584.215 | 753.823 |  458.371   |
|         SIFT          | 607.335 |   623.61    | Out of Memory | 805.025 |   N/A   |  1072.55   |



#### Efficiency (matches/ms)

| Detectors\Descriptors |  BRISK   |    BRIEF    |      ORB      |  FREAK   |  AKAZE   |   SIFT   |
| :-------------------: | :------: | :---------: | :-----------: | :------: | :------: | :------: |
|       SHITOMASI       | 3.51073  |   4.9953    |    4.37254    | 1.03797  |   N/A    | 2.24156  |
|        HARRIS         |  1.3238  |   2.14332   |    1.49094    | 0.425504 |   N/A    | 1.07636  |
|         FAST          | 21.9598  | **33.8528** |    27.4329    | 1.10391  |   N/A    | 2.49780  |
|         BRISK         | 1.05023  |   1.21729   |    1.01203    | 0.572395 |   N/A    | 0.827842 |
|          ORB          | 6.39606  |   5.09532   |    5.94507    | 1.11201  |   N/A    | 1.34597  |
|         AKAZE         | 0.900572 |   1.02314   |   0.911598    | 0.60423  | 0.455014 | 0.759210 |
|         SIFT          | 0.330954 |  0.399288   | Out of Memory | 0.242228 |   N/A    | 0.274113 |



Following are the best detector-descriptor combinations,
1. FAST + BRIEF
2. FAST + ORB
3. FAST + BRISK
