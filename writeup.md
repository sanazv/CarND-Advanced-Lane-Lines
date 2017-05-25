## Advanced Lane Finding 

In this writeup I will summerize the steps taken to complete the project and arrive to the final video included in this repo.


The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images (done on both chessboard images and test images provided by Udacity).
* Use various color and gradiant thresholding combinations to create a thresholded binary image.
* Apply a perspective transform to rectify binary image and convert it to bird's eye view.
* Detect lane pixels and fit to find the lane boundary. Search the first frame thouroughly and then search in the vicinity of found lanes in the second frame onwards.
* Determine the curvature of the lane and vehicle position with respect to center of lanes.
* Warp the detected lane boundaries back onto the original image to dislay the image from the dashcam point of view.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
* Apply the resulting pipeline to video frames and combine the results to a final video, with the goal of detected lines displaying the lane lines correctly without much wobble.

I will explain each step in more details supported by figures in the following. The code for all these steps is also in this repo: *Advanced_Lane_Finding.ipynb*

[//]: # (Image References)

[binary_bird_drive]: ./writeup_images/binary_driver_and_birds.png 
[first_frame_fit]: ./writeup_images/binary_first_frame_fit.png 
[second_frame_fit]: ./writeup_images/binary_second_frame_fit.png 
[CB_after_perspective]: ./writeup_images/chess_board_after_pt.png 
[CB_before_after_calib]: ./writeup_images/chessboard_before_after_calib.png 
[CB_before_perspective]: ./writeup_images/chess_board_before_pt.png 

[color_thresh]: ./writeup_images/Color_threshold.png 
[driver_and_birdseye]: ./writeup_images/driver_and_birdseye_view.png 
[finding_corners]: ./writeup_images/finding_corners.png 
[combined_threshold]: ./writeup_images/image_and_combined_threshold.png 
[image_green_lanes]: ./writeup_images/image_and_green_lanes.png 
[mask]: ./writeup_images/mask_file.png 


[persp_t_coordinates]: ./writeup_images/persp_transform_coords_for_vidframes.png 
[sobel_dir_thresh]: ./writeup_images/Sobel_direction.png 
[sobel_x_thresh]: ./writeup_images/Sobel_direction_x.png 
[sobel_y_thresh]: ./writeup_images/Sobel_direction_y.png 
[Sobel_mag_thresh]: ./writeup_images/Sobel_magnitude.png 
[test_camera_calib]: ./writeup_images/testimage_before_after_calib.png 


[video1]: ./finding_advanced_lanes.mp4 "Video"



## Camera Calibration
The first step of the pipeline is to correct for the distortion caused by camera lenses on the frames taken by the dashcam.
In order remove this distortion, we were provided of a set of chessboard images taken from the same camera at different angles with 9x6 corners.
Using cv2 functions such as ```cv2.findChessboardCorners()``` I got a a list of object points and image points to be then used with ```cv2.calibrateCamera()``` to get calibration matrix for the camera. The matrix is then used to correct for the distortion caused by camera lens.
The result of this correction can be seen in the figure below:
![alt text][CB_before_after_calib]
As it can be seen the rounded edges of the chessboard pattern get straghtened after undistorting the images with the calibration matrix.
I then apply the same calibration matrix to undistort images of the lane lines taken by the same camera. All images for this project are 1280x720 pixels.The figure below shows the effetcs of undistortion on one of the test images:
![alt text][test_camera_calib]


## Masking
As the next step I masked the regions of the image that do not contain lane lines. I set all the corners of the mask as ```vertices```. The shape of the mask applied to all frames is:

![alt text][mask]

## Thresholding
In this stage, the goal is to arrive at a binary image where the lane lines are the most pronounced and the pixel value all other regions are the image is nearly zero. In order to achive this task, I process each image and take gradient in different orientations, take the direction and magnitude of that gradiant. Also looking at the image in HSL space and focusing on S channel (Saturtation) seems to be a powerful way of isolating the lane line. In the end I make a combined image where each of these individual thresholds contibute to the final binary image.
### Gradiant Thresholding
I first conver the color image to gray scale. Then I take the gradiant of the image in x and y direction enhancing vertical and horizontal lines respectively using ```cv2.Sobel()``` function. I also take the gradiant and magnitude of gradiant at each pixel.
The results of each of these operations are shown below:

![alt text][sobel_x_thresh]
![alt text][sobel_y_thresh]
![alt text][sobel_mag_thresh]
![alt text][sobel_dir_thresh]


### Color Thresholding
Also looking at the images in HSL colorspace, indicates that lanelines (especially the yellow ones), appear nicely in S channel. For this reason I convert the images from RGB to HSL colorspace using ```cv2.cvtColor(image, cv2.COLOR_RGB2HSL)```
and then take the S channel and apply threshold values to it.
The figure below shows the impact of thresholding S channel:

![alt text][color_thresh]

As it can be seen the lane lines apprear clearly.
### Combined Binary Image
Finally I combine all these indiviual thresholded binary images to make one final image. Below I listed the threshold values and kernel sizes for each indivudual layer.

| Conversion        | Kernel   | Threshold min  | Threshold max   | 
|:-------------:|:-------------:|:-------------:|:-------------:| 
| Gradiant x      | 9       | 20 | 200 |
| Gradiant y      | 9       | 20 | 200 |
| Gradiant direction     | 11     | 40 | 200 |
| Gradiant magnitude      | 11    | 0.7 | 1.3 |
| Color S channel      | --        | 85 | 255 |

The figure below shows the result of the combination of thresholding steps. 
![alt text][combined_threshold]


## Perspective Transform
In the next step the goal is to transform the point of view of the image from the dashcam (almost driver's point of view) to bird's eye view, so that the effect of converging lines is removed and any curvature in the lines represent the real curvature of the lines.

For this purpose I use ```cv2.getPerspectiveTransform()``` which takes a set of source and destination coordinates and returns a transformation matrix which maps the image from driver's point of view to bird's eye view.
I have selected the source (blue) and destination (red) points as below:
![alt text][persp_t_coordinates]

The figure below shows the two perspectives in color images where it can be seen that the lane lines appear nearly parallel:
![alt text][driver_and_birdseye]

While this figure shows the two perspectives in binary image which was the result of combined thresholing from the previous stage of the pipeline. 
![alt text][binary_bird_drive]

## Fitting Polylines Lane Lines
So at this point I have a binary image from bird's eye perspective, where the lane lines appear as two curved lines. The next step is to fit a polynomial to the points that make up each of the lines. In an attemp to speed up the process and smoothen the lanes discovered, the image processing on the second frames onwards takes advantage of the results obtained in the previous frames. Here I first explain how the polynomial fit is done on the first frame and then will go over the second one.

### First Frame
For the first frame, I take the histogram of all the pixels along the x axis and determine where the two peaks of the distribution are. Then I apply search windows (code provided by Udacity) to search for pixels with values of 1 in that window, and then slide the windows vertically to follow the shape of the line.
Then I fit a secong order polynomial to the pixels of right and left lane separatly.
The figure below shows the result of the window search on the first frame. teh yellow lines show the fit.

![alt text][first_frame_fit]

### Second Frame onwards
For the second frame onwards, I start the search in the vicinity of the last frame, since the video is a timeseries and there is no need to assume that the subsequent frames would be massivly different.
In the figure below I show the fit (in yellow) as also the area of search (in green) which is obtained based on prvious frame. I have found this stage extermeley helpful in allowing the algorithm to handel tricky lighting conditions.
![alt text][second_frame_fit]

Now that the lanes are indentifed the task is almost done.

## Radius of Curvature and Vehicle Position
One last step is to calculate the radius of curvature and vehicle position. Radius of curvature of calculated using the polynomial fit parameters which are returned from the previous step when I fit the right and left lanes. 
I take the average of left and right radii of curvature and then convert that from pixel units to distance units, under the assumption that there are 3.7 meters per every 700 pixels of the image. In orders of magnitude the radius of curvature accorss the whole video is ~ 1000 m.


For the vehicle position, I assume that the camera is installed on the center of the hood, so that the offset of the camera from the midpoint of the lanelines is the same as the offset of the vehicle itself.
I find the midpoint pixel between the left and right lanes (after averaging over the entire length of the lane) and then compute its difference with the center pixel (640). I then convert this value back to meters with the same assumption mentioned above. 
As it can be from the video, the car poisition is always negative (meaning that the canter of the car is a bit to the left of the lane) by a few centimeters, which is good news, both for the driver and for sanity check of my work.

## Unwarp the Final Lane Lines
As a final step I unwarp (use the inverse of the perspective transform) to convert the detected lane lines back to drivers perspective and overlay them with the original image.
The result for a sample frame is shown below:
![alt text][image_green_lanes]


## Video Processing
And finally the entire pipeline is applied to all frames of the video provided and the results can be found in *finding_advanced_lanes.mp4* file.






