## Writeup Template
---

**Vehicle Detection and Tracking**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, apply a color transform and append binned color features, as well as histograms of color, to HOG feature vector. 
* Normalize the features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run the pipeline on a video stream (starting with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car1.png
[image2]: ./examples/HOG_example1.png
[image3]: ./examples/sliding_window.png
[image4]: ./examples/test1_cars.png
[image5]: ./examples/test1_cars.png
[image6]: ./examples/sliding_windows_hm_test1.png
[image7]: ./examples/sliding_windows_hm_test5.png
[video1]: ./project_video_out.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the python file in utils folder called `lesson_functions.py`.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `RGB` color space and HOG parameters of `orientations=32`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I worked on various combinations of color spaces and parameters for HOG functions to get the best score for the SVM Classifier. The final HOG parameters for best SVM training were achived with RGB color-space,  `orientations=32`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using the default settings, the code is located in cell 5 of the notebook.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The find_cars() function implements the sliding window search. It performs a constrained search by using ystart = 400 and ystop = 660, as most cars are expected to be in that search area. It further uses window sizes of 64x64 with scale=1 and changes with different scales to bigger or smaller windows. The scale is varied between 1.2 to 3.0 to cover near objects as well as far objects that appear smaller.

The image below shows the range of search for every frame.

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on the scales array using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here is an example image from the test set provided:

![alt text][image4]

I tried various methods to do temporal filtering over some frames so as to track vehicles and then label those as cars. I created a deque for the heatmap count, that did not work very well for me. Then I tried changing the search radii around the heatmap based on the history of such heat maps over multiple frames. That worked good but was not able to detect cars once they moved away. I tried playing with the radii but that resulted in false positives.
Lastly, I used two thresholds to filter out false positives over 10 frames. That seems to work the best with the given settings.


---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_out.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions. I also kept track of car labels detected every 10 frames and refresh them as time proceeds. By using `scipy.ndimage.measurements.label()` the individual blobs in the heatmap were identified.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image6]


### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]


---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

This project involved a lot of manual tuning for the various parameters for good detection of vehicles. Data Augmentation could have helped. The method used here itself will not be robust for real time driving vehicle detections. Hence other and better methods needs to be explored.

References: https://github.com/asgunzi/CarND-VehicleDetection and the blog.
