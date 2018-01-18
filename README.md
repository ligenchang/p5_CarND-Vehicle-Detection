[//]: # (Image References)
[image1]: ./examples/cars_noncars.jpg
[image2]: ./examples/all_cars_noncars.jpg
[image3]: ./examples/sub_sampling_123.JPG
[image4]: ./examples/sub_sampling_123_scaled.JPG
[image5]: ./examples/sampling_rate_64.JPG
[image6]: ./examples/Sampling_rate_128.JPG
[image7]: ./examples/test_image.JPG
[image8]: ./examples/test_image2.JPG
[image9]: ./examples/heat_map_individual.JPG
[image10]: ./examples/heat_map_consective.JPG
[image11]: ./examples/color_space.JPG
[video1]: ./project_video.mp4
[image20]: ./examples/test_image_6.JPG
# Vehicle Detection
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


The Project
---

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

Here are links to the labeled data for [vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/vehicles.zip) and [non-vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/non-vehicles.zip) examples to train your classifier.  These example images come from a combination of the [GTI vehicle image database](http://www.gti.ssr.upm.es/data/Vehicle_database.html), the [KITTI vision benchmark suite](http://www.cvlibs.net/datasets/kitti/), and examples extracted from the project video itself.   You are welcome and encouraged to take advantage of the recently released [Udacity labeled dataset](https://github.com/udacity/self-driving-car/tree/master/annotations) to augment your training data.  

Step 1, Select features
---
To recognize the vehicle in the image, firstly we need to identify what features we want to estract from the image as the identification to differentiate it's a vehicle or not. 

I tried with histogram as the only factor for recognizing car, the accuracy is around 52%, which means this feature doesn't help too much in recognizing car, but we better still need it as the accuracy is still more than 50%. 

Next I tried with spatial feature with spatial size (16 * 16) only and the accuracy is 91.98%, which is nice and it helps us to classify image based on this feature.

The I tried with HOG feature only and the recognize accuracy is 97.94%

If I combine HOG feature, spatial feature and histogram features together, the accuracy will be reached at around 98.6%.

I explored what does it looks like for different color spaces and based on that I find that the color space YCrCb keep the most of the features and still clear enough to recognize.
![alt text][image11]

Here is the example of HOG features with YCrCb color space and HOG parameters of orientations=11,pixels_per_cell=(8, 8) and cells_per_block=(2, 2):
![alt text][image2]

Step 2 Training with liner svc
---

There are several options to be served as classifier like svc, liner svc, naive bayes, support vector machine. But of which the liner svc is the fastest one and it can support for self driving car in real time. After training it, it tested in on  6 test images and result looks good and here is 2 of the test images.

![alt text][image7]

![alt text][image8]

Here is the predicition result for all the images in test image folder:
![alt text][image20]

Step 3 Hog Sub-sampling Window Search 
---
I decided to use Hog Sub-sampling Window Search to find all patches in the image and do predicition because only has to extract hog features once and then can be sub-sampled to get all of its overlaying windows.

Here is sample on how to do sub sampling search based on the sampling rate and search with steps defined in the code, here i used the steps as 2 with sampling rate 64 and here i shows how it moves step by step.


![alt text][image3]

![alt text][image4]

Here I show how it looks like with scaling with 1 and the sampling rate is 64:

![alt text][image5]

This image is for scaling rate equal to 2: 

![alt text][image6]

Step 3 Scaling rate selection
---
The default sampling rate is 64*64 and we need to scale it to make it easier to recognize the big car and i decided to set the max scale value as 2 and increase 0.1 every time and get a random value between each zone. The code is as follow:
```python
scale = 2
    recs=[]
    my_draw_img = np.copy(img)
    
    # use different scales to find cars
    for i_scale in np.arange(1,scale,0.1):
        i_scale_random=random.uniform(i_scale, i_scale+0.1)
        out_img, rec = find_cars(img, ystart, ystop, i_scale_random, svc, X_scaler, orient, pix_per_cell, cell_per_block, spatial_size, hist_bins)
        recs = recs+rec 
```


Step 4 Find threshold value for heatmap
---

It's hard to find a fixed threshold values for all different frames. After sevral try, i used the 40% percentile value of the heatmap as threshold after removing all values less than 15. So far it works fine.

### Here are six frames and their corresponding heatmaps:

![alt text][image9]

### Here is the output of on the integrated heatmap from all six frames:
![alt text][image10]

Step5 project output video
---

Here is my final output video for this project. 
Here's a [link to my video result](./project5.mp4)