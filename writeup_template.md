**Advanced Lane Finding Project**

The steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images and store them as a pickle file.
* Break the video feed into separate images .
* Apply undistortion methods in opencv to correct the input image .
* Convert Color space from RGB to HLS .
* From HLS remove the saturation channel and apply threshold .
* Find the gradient in x direction using sobel and apply threshold to it .
* Combine both these threshold images into a single one .
* Define a region of interest mas and apply to the threshold image to remove all the unwanted borders .
* Apply perspective transformation to the thresholded image to convert it into birds eye view
* Use techniques like erode , dilate to clean the image from noise
* Apply histogram to find the center of the lane line
* If we had already found the lane line in the previous frame , we can use its x value to find the starting point (Don't use Histogram)
* Using sliding window method provided at udacity , find the lane pixels and fit an second order polynomial
* From the polynomial function , calculate the radius of curvature by applying this formula

    Image of formula
* Convert it into meters if needed
* Find the difference between absolute center and the car center by computing the difference between midpoint between the lane and the pic center

* plot the needed things onto the image (center , binary image )and save them as a video


[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for calculating cam matrix and distortion coeff is at the 4th block . the workflow are,
* load in the calibration image
* define object point (X,Y,Z) and image point(X,Y) list
* using the opencv function (cv2.findChessboardCorners) find the X & Y coordinates of corners in the chessboard image and store these points into image point list. in the object point list , create and store the index of these points ((1,2,0),(3,4,0)...). 
* using these points and the size of the image , we can compute the matrix and distortion coeff by using opencv function (cv2.calibrateCamera)

Note : in the 3rd code block , specify the path for the chessboard calibration images

 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.

from the previous step , we can obtain the cam matrix and the distortion coeff of out camera . using these values , we can use these values to correct the image distortion by sending the image through the cv function cv2.undistort along with the cam matrix and distortion coeff



![alt text][image2]



####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

At code blocks ( 8 and 10 ) i have used HLS thresholding and sobelx thresholding methods to find lines in the image

###### HLS 
* convert the image colorspace from RGB to HLS
* separate S channel from the HLS
* apply thresholding to the output S channel

###### sobelX
* convert the RGB image to grayscale
* apply sobel gradient along the x axis using cv function (cv2.Sobel(gray,cv2.CV_64F,1,0))
* apply threshold to separate the lane lines from the gradient image

finally add both of these images into a single binary image by combining both thresholded images

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

At codeblock 16 , i have defined a function to perform perspective transformation of images . i defined a region (a polygon) and used opencv function (cv2.getPerspectiveTransform(src,des)) . here src = source points and des = destination points . this function will give me a  3x3 matrix of perspective transform . using this matrix , we can transform the binary image into a birds eye view image

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 760 , 486      | 1080 ,    0        | 
| 1000 , 700      | 1080 ,   720      |
| 220 , 700     |  200  , 720     |
| 560 , 486      | 200   ,  0       |

img_size = (gray.shape[1],gray.shape[0])
adjx = 200
adjy = 0    
src = np.float32([[760 , 486],[1000 , 700],[220 , 700] , [560 , 486]])        
des = np.float32([[img_size[0]-adjx,adjy],[img_size[0]-adjx,img_size[1]-adjy],[adjx,img_size[1]-adjy] ,[adjx,adjy]])        
M = cv2.getPerspectiveTransform(src,des)
Minv = cv2.getPerspectiveTransform(des,src)


![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

At the main code function (AdvLaneFinder) , i have done both  window search to find the white pixels representing lane lines and also fitted a polynomial to it . i reused the same code provided by udacity to do my pixel searching and fitting a polynomial to the points 

window search consist of finding the center point either from a histogram or using the center of the previous lane line found (if we had a good fit in the past) . from that point , initialize a window and start searching form pixels with non zero value (that is 1 ) . if we found pixels which exceeds an arbitrary count , say 100 , we then recenter the window based on the pixel mid point . carryout this same process till you reach the top of the image 

using the point found by the previous method , fit a 2nd order polynomial by using numpy function (np.polyfit)

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

for the radius of curvature , i reused the udacity code . for position of vehicle with respected to center , i developed a function called centerFind which is at 15 th code block , calculates the center between the two lanes and compares it with the image center along x axis . the difference in the value is the position of vehicle respective to the center of lane

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Problems faced 

* the process of choosing src and des points for perspective transformation is tedious because slight change in points can affect the orientation of the transformed image
* presence of shadow and sudden jerk in the video can affect the lane finding process
* this model works well for flat roads . roads with up and down are a bit challenging 
* threshold values need to be modified for various road and video conditions . using a fixed value give rise to a problem of not being generalized (finds lot of noise )
* implement polynomial check for more robust model

