# Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)
[chessboard_marked]: ./output_images/chessboard_marked.png "chessboard marked"
[undistorted]: ./output_images/undistort_calibration.png "Undistorted"
[org_undistorted]: ./output_images/undistort_straight_lines1.png "Undistorted Test Image"
[pre_processings]: ./output_images/pre_processings.png "Show preprozessings"
[processing]: ./output_images/processings.png "Show preprozessings"
[wraped]: ./output_images/warped.png "Wraped image"
[processed_img]: ./output_images/processed_img.png "Processings"
[slidewindow]: ./output_images/slidewindow.png "Fit Visual"
[draw_lines]: ./output_images/draw_lines.png "Lines drawn"
[display_curve_values]: ./output_images/display_curve_values.png "display_curve_values"
[processed_img]: ./output_images/processed_img.png "processed"

[video1]: ./project_video.mp4 "Video"

---
### 0. Files included
Following files and folders are included:

* **README.md:** This README
* **Advanced Lane Finding.ipynb:**  Notebook contains all code to produce the result video and the pictures of this README
* **camera_cal:** This folder contains the pictures of the camera calibration
* **output_images:** This folder contains all pictures used in this README
* **test_images:** This folder contains all pictures to test image preprocessing and calculations.
* **project_video.mp4** The original video to process and find the lines.
* **project_video_output.mp4** The processed video with lines and displaying lines and curve calculations.

---
### 1. Camera Calibration

Every camera produces pictures with its individual [distortion](https://en.wikipedia.org/wiki/Distortion_(optics)). So it is necessary to calibrate the "camara" with chessboard images. 

The code for this step is contained in the first code cell of the IPython notebook located in the notebook in the Chapter "1. Camera Calibration"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `corners` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

Not on all images the chessboard corners were detected. Some images need smaller patterns to be detected.  So first I tried to detect the corners with a (9, 6) pattern. If the pattern  has not been found I tryed a (9, 5) pattern. 

![alt text][chessboard_marked]

---
### 2. Undistorting

I then used the output `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result. See notebook in the Chapter "2. Camera Calibration"

![alt text][undistorted]

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][org_undistorted]

### 3. Preprozessing pipeline
I used a combination of color and gradient thresholds to generate a binary image. You find the code located in the notebook in the Chapter "3. Create a thresholded binary image"

The two in the first line show the
* original image (Original)
* the final binary image (Result)

The three pictures in the bottom line are the result of the three thresholds combined into the final result

* First picture:  Absolute sobel thresholding in x orientation on the L - Layer of an HLS encoded picture. (ksize = 3, hresh=(25, 100)) **ABS_SOBEL(L, x)**

* Second picture: Absolute sobel thresholding in x orientation on the S - Layer of an HLS encoded picture. (ksize = 3, hresh=(10, 100)) **ABS_SOBEL(S, x)**

* Mask to find the light spots on the picture **LIGHT_MASK()**

The Pipeline is defined as  

**(ABS_SOBEL(L, x) OR ABS_SOBEL(S, x)) AND LIGHT_MASK()**

The result is the second Image in the first line (Result)

![alt text][pre_processings]


### 4. Perspective transformation

The code for my perspective transformation includes a function called `warp()`, which appears in the notebook in the Chapter "4. Perspective transform". I chose this hardcoding of the source and destination points in the following manner:

```
src = np.float32 ([
        [285, 671],
        [418, 577],
        [885, 577],
        [1000, 651]
    ])

dst = np.float32 ([
        [220, 651],
        [220, 550],
        [921, 550],
        [921, 651]
    ])
```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 285, 671      | 220, 651        | 
| 418, 577      | 220, 550      |
| 885, 577     | 921, 550      |
| 1000, 651      | 921, 651        |

I verified that my perspective transformation was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][wraped]

### 5.  Image pipline 

The final Image processing pipeline is like the following:

* undistorting the image 
* wrap image
* preprocessing image

This is the result on all test images. The picture in the last column shows the histogramm of the final result . 
You can find the code in the notebook Chapter "5. Image Pipeline"

![alt text][processing]
 

### 6. Identifying lane-line pixels and positions fitting with a polynomial.

In `slidewindow` (notebook Chapter 6 "Sliding Window Polyfit"), I extract lines and fit polynomials to binary image. In order to do that, I find the peak of the left and right halves of the histogram. These will be the starting point for the left and right lines image and use 17 sliding windows to detect the line position. The addition of sliding windows speeds up the search for lane and focuses on only the parts of the image that lane pixels are detected. Numpy method `polyfit()` applies a second-order polynomial to the lane pixels.


The output from `slidewindow` is used as an input for `drawLines` to show the lane polynomial as a red and blue line.
It is also used as an input for `polyfit_on_fit` to narrow down the location of lane further.

![alt text][slidewindow] ![alt text][draw_lines] 


### 5.  Calculation of the radius of the curvature of the lane and the position of the vehicle with respect to the center.

The code for my radius of curvature and vehicle position is done in a function called `calculate_curvature_distance()`, which appears in the notebook Chapter "8. Radius of Curvature and Distance from Lane Center Calculation".

#### a. radius of the curvature

I referred to [this](http://en.wikipedia.org/wiki/Curve_fitting) and [this](http://en.wikipedia.org/wiki/Polynomial_interpolation) to come up with the formula for curvature compution and I use the following code to perform the radius calculation: 
```
left_fit_cr = np.polyfit(left_y*y_meters_per_pixel, left_x*x_meters_per_pixel, 2)

left_curve_radius = ((1 + (2*left_fit_cr[0]*y_bottom*y_meters_per_pixel + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
```
#### b. position of the vehicle with respect to center.

I used the midpoint of dashcam image width as the vehicle position and then used the x position of the lane curves to get the midpoint between the two to find the center of the lane. In the end I subtracted center of the lane from vehicle position and multiplied it by meters per pixel to get an estimate how far off the vehicle is from the center of the lane. I estimated 10 ft was roughly equal to 3.05 meters and 12 ft qual to 3.8 meters. The code for this step can be inspeted in the 28th code cell of the IPython notebook.

### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines in my code in the notebook Chapter "9. Process whole pipeline" in the function `process()`.  Here is an example of my result on a test image:

![alt text][display_curve_values]

### 7. Pipeline (testimages)
Running the pipeline on all testimages the following result is shown:

![alt text][processed_img]

---

### 8. Pipeline (video)

[![E](https://img.youtube.com/vi/CG84tNsAm7Q/0.jpg)](https://youtu.be/CG84tNsAm7Q "Training Track - Track 1")

Here's the [direct download](./project_video_output.mp4)

---

### Discussion

The result of my solution works pretty well on all test images and the most parts of the video.

Looking at the video it seams the process has no problems with dark structures like shadows. It had some problems with light, more horizontal lines on the right side. Likes at timemark 00:24 or 00:40.
 
 An approach to make this process better could be to get pictures where the current solutions have problems and analyse them in the  following way.
 
 * **Adapted perspective transformation** : It will be helpful to choose longer lines at the src and test points, so the histogram is easier to be interpreted via the "Sliding Window Polyfit". The transformation might be used as mask in a better way too.
 
 * **Adapted Image preprocessing**: Watch out for layers of different image encodings like HSV, or look for other threadhold algorithms like Sobel Magnitude Threshold or Sobel Direction Threshold. 
 

