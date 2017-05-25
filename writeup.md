## Advanced Lane Finding Writeup 
In this writeup I will summerize the steps taken to complete the project and arrive to the final video included in this repo.
---

**Advanced Lane Finding Project**

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

I will explain each step in more details supported by figures in the following.

[//]: # (Image References)

[binary_bird_drivee]: ./writeup_images/binary_driver_and_birds.png 
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



## Masking


## Thresholding

### Gradiant Thresholding

### Color Thresholding

### Combined Binary Image
| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

## Perspective Transform


## Fitting Polylines Lane Lines

### First Frame

### Second Frame onwards


## Radius of Curvature and Vehicle Position

## Unwarp the Final Lane Lines

## Video Processing




![alt text][image1]





