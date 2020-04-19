# **Finding Lane Lines on the Road** 

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 7 steps. First, I created a copy of my image the applied the filters below:
- `grayscale` - as canny edge detection works correctly only on grayscale images
- `gaussian_blur` - as additional tweaking for canny edge detection
- `canny` - to detect edges in the image
- `region_of_interest` - to reduce amount of edges which Hough's transform needs to process
- `hough_lines` - to detect lines
  - it also included extrapolating of the line fragments to cover the whole FoV
- `region_of_interest` - to remove excessive line fragments after extrapolation
- `weighted_img` - to overlay detected lanes on the original image

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by
1. Calculating the slopes of all the lines
2. Grouping them in right and left lane according to the slope values (0.4 to 0.8 belong to the right lane, -0.8 to -0.4 belong to the left lane; the others are just noise)
3. Then, for right and left lane separately replace all the lines with the average one:
   1. Calculate the averages of all the coordinates (x1, y1, x2, y2)
   2. Using `np.polyfit` and `np.poly1d` get the function based on the average points
   3. Calculate the line parameters for the whole X range which is relevant (i.e. from 0 to image width)
4. In the end draw the two average lines


### 2. Identify potential shortcomings with your current pipeline

1. The lines sometimes disappear completely, i.e. probably the Hough's transform has problems detecting any line on some frames
2. The lines are shaking. I included 'amortization' by doing the average of the current and previous line positions, but it still does not look right
3. It goes crazy on the challenge video. Sun, different position of the camera and sharp edges of the barriers make the algorithm completely confused
4. It calculates the RoI twice, which would be not necessary if the `draw_lines()` was implemented better, but it was just the fastest way to code it 


### 3. Suggest possible improvements to your pipeline

1. Smoothing out the shaking - currently it is average from current and previous reading. If the size of 'history' was increased, the smoothing would surely work better
2. `draw_lines()` function can be improved not to draw lines over the whole image, but only up to the horizon. Then the second RoI would be unnecessary. Also it would look much better on the challenge video.
3. The detection of the current frame line should use more information from the previous frame. It would enable to mitigate that confusion in the challenge video. Ex. under normal circumstances the line on one frame cannot be too far away from  the line on the preceeding frame, so some kind of 'validity zone' can be used. 
4. Probably RoI may be slightly adjusted to be more generic. Currently the slightest change in the camera position makes it useless (as in the challenge video)
