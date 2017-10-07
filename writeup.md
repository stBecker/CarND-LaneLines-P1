# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[image2]: ./test_images_output/lanes_solidWhiteCurve.png
[image3]: ./test_images_output/lanes_avg_solidWhiteCurve.png


---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps. 

1. Convert image to grayscale.

![alt text][image1]

2. Apply gaussian blur.
3. Apply canny edge detection.
4. Mask region of interest. The region is defined as the area between the bottom left and right 
corners of the image and two points approximately in the middle of the image.

        vertices = np.array([[(0,imshape[0]),(imshape[1]/2-1, imshape[0]/2+50), (imshape[1]/2+1, imshape[0]/2+50), (imshape[1],imshape[0])]], dtype=np.int32)
        img = region_of_interest(img, vertices)
        
5. Find the hough lines and draw them in the image.
6. Overlay the original image with the discovered lane lines.

The result of the naive approach can be seen below.
![alt text][image2]


To improve the robustness of the lane detection, it is necessary to modify the draw_lines() function.

First I try to determine if a hough line belongs to the left lane or to the right lane or 
no lane at all (e.g. a nearly horizontal line is probably not part of a lane).
I compute the slope of a hough line and determine if it falls within a certain threshold, i.e. 
between 20° and 60° for the right lane and between -20° and -60° for the left lane.

    # angles in degrees
    angle_max = 60
    angle_min = 20
    slope_min = np.tan(np.deg2rad(angle_min))
    slope_max = np.tan(np.deg2rad(angle_max))
    
    imshape = img.shape
    
    left_segments = []
    right_segments = []
    left_slopes = []
    right_slopes = []
    for line in lines:
        for x1,y1,x2,y2 in line:
            slope = (y2-y1)/(x2-x1)
            if slope_min < slope < slope_max:
                right_segments.append((x1,y1,x2,y2))
                right_slopes.append(slope)
            elif -slope_min > slope > -slope_max:
                left_segments.append((x1,y1,x2,y2))
                left_slopes.append(slope)
                    
                    
Then for each lane I calculate the mean of the slopes 'm', rounding the value to increase robustness.
I also calculate the mean x and y coordinates of the hough lines belonging to the lane.
With these variables I can determine the y-intercept 'b', according to the formula y = m*x + b.

Plugging in the above variables and the min and max y-values for the region of interest, we can solve for x.

    m = np.mean(left_slopes)
    m = round(m, 2)
    
    x = np.mean([(x1 + x2)/2 for x1,y1,x2,y2 in left_segments])
    y = np.mean([(y1 + y2)/2 for x1,y1,x2,y2 in left_segments])
    b = y - m*x

    y = imshape[0]/2+50
    x = (y-b)/m
    lane_top = (int(x), int(y))

    y = imshape[0]
    x = (y-b)/m
    lane_bottom = (int(x), int(y))

    cv2.line(img, lane_top, lane_bottom, color, thickness)

![alt text][image3]



### 2. Identify potential shortcomings with your current pipeline


1. Currently only straight lanes can be predicted accurately, when the lane is curved it is not tracked 
correctly (as can be seen in the challenge video).
2. The hough lines algorithm sometimes fails to detect the broken lane line.


### 3. Suggest possible improvements to your pipeline

1. Track curved lanes by fitting a 2nd order polynomial, instead of the current 1st order polynomial.
2. In a video stream record a time series of detected lane lines and interpolate the lane 
lines between several frames of the video, to 
- make the lane lines output smoother between frames,
- predict the lane line for frames in which no (broken) lane line could be detected because of 
insufficient data.


