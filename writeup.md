**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./output_images/hog_features.png
[image3]: ./output_images/test_image_boxes.png
[image4]: ./output_images/test_image_1.png
[image42]: ./output_images/test_image_2.png
[image43]: ./output_images/test_image_3.png
[image44]: ./output_images/test_image_4.png
[image45]: ./output_images/test_image_5.png
[image46]: ./output_images/test_image_6.png
[image5]: ./output_images/heat_map_image.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the second code cell of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  To determine the optimal feature set I examined the raw and normalized features for random images within the set. The YCrCb image space was determined to be the most robust due to the uniqueness of the individual features. Below is an example of an image with its raw and normalized features:

![alt text][image2]



#### 2. Explain how you settled on your final choice of HOG parameters.

The final number of orientations used for the HOG feature extraction was 8. This lower number was used due to the SVM converging on a non-generalized solution at larger numbers of orientations. The final number of pixels per cell was determined to be 8 while each cell had two blocks. These values were found by trial and error to minimize the test set error.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The code for this step is contained in the third code cell of the IPython notebook.

I trained a linear SVM using the GTI, KITTI, and Extras datasets. The SVM classifier was trained using an 80/20 train/test split using a random state initialization to ensure that the chosen hyperparameters were broadly applicable. The LinearSVC() function from the sklearn package was used over a tree based system through iterated testing.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The code for this step is contained in the fifth code cell of the IPython notebook.

I began by searching between the interval of 0.5 and 10 for scales on the lower half of the image. Given the size of each scale on the images used I narrowed the final scales down to seven from 0.75 to 5 and restricted the range for each scale independently. This way each scale was only applied to vehicles within the reasonable range of pixels for that scales pixel size. The overlap was decided to be the size of a single HOG cell to reduce redundant processing.

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector. A minimum threshold of one was used with a heat map to remove false positives. Here are some example images:

![alt text][image4]
![alt text][image42]

![alt text][image43]
![alt text][image44]

![alt text][image45]
![alt text][image46]


---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_processed.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions, found in code block six.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a frame of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here is a frame and its corresponding heatmaps:

![alt text][image5]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main problems faced in this project were finding a proper set of scales for the search and thresholds for the heat system. Eventually I found that a combination of many scales with a relatively high heat threshold worked well. In order to remove most of the remaining artifacts I used the previous video frame to to increase certainty of current predictions. Rather than using full heat values from the previous frame I rather used half their value to account for the difference in time. The main areas this system could fail would be for styles of car's that the system has never seen and images that appear to be cars but are not. These could be accounted for by combining the current system with either a lidar based system or a second video feed to recieve three dimensional data.

