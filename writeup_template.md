## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/cars_noncars.jpg
[image2]: ./examples/all_cars_noncars.jpg
[image3]: ./examples/sub_sampling_123.JPG
[image4]: ./examples/sub_sampling_123_scaled.JPG
[image5]: ./examples/sampling rage 64.JPG
[image6]: ./examples/Sampling rate 128.JPG
[image7]: ./examples/test_image.JPG
[image8]: ./examples/test_image2.JPG
[image9]: ./examples/heat map individual.JPG
[image10]: ./examples/heat map consective.JPG
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the fourth code cell of the IPython notebook Vehicle.ipynb.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.feature.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`). I have a function get_hog_features which can extract the hog features and also can return the hog image.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and finally selected following parameters based on test accuracy:
orient = 11  # HOG orientations
pix_per_cell = 12 # HOG pixels per cell
cell_per_block = 2 # HOG cells per block
hog_channel = "ALL" # Can be 0, 1, 2, or "ALL"

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The code for this step is contained in the fourth code cell of the IPython notebook Vehicle.ipynb.  

I trained a linear SVM using LinearSVC, which runs efficiently and best fit for the real time predicition like self driving car.

### Hog Sub-sampling Window Search 

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to use Hog Sub-sampling Window Search to find all patches in the image and do predicition because only has to extract hog features once and then can be sub-sampled to get all of its overlaying windows.

Here is sample on how to do sub sampling search based on the sampling rate and search with steps defined in the code, here i used the steps as 2 with sampling rate 64 and i shows how it moves step by step.


![alt text][image3]

![alt text][image4]

Here I show how it looks like with scaling with 2 and the sampling rate is 128:

![alt text][image5]

![alt text][image6]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on 10 scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:


![alt text][image7]
![alt text][image8]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project5.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  For the threshold selection, I used the 40% percentile value of the heatmap, any value less than this value will be set to 0.

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image9]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image10]





---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

This project is quite challenge and need a lot of knowledge and i master the knowledge of how to extract of features from image and training them and predicate the result. Though i tried to stablize it and smooth the box detected with consolidating all boxes for the last six frames and also average the heatmap threshold value with last 30 values, there are still areas to improve:
1. The margin of the box it not perfectly matched to the car and it has room to improve the threshold value and also can try with other smooth values
2. The training data is not augmented and if I can do some histograms equalization and rotation, the accuracy may increase
3. I limited the interest of areas in the right part of image, but in real world, we don't know which area of our intered. If the car go to the shaded area, it could have lots of noises. Currently i still don't have any good solution for that.