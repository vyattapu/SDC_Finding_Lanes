# **Finding Lane Lines on the Road** 

<img src="examples/laneLines_thirdPass.jpg" width="480" alt="Combined Image" />

# **Reflection**

The finding lanes pipeline of this project consists of 6 steps:
1. Convert the images to grayscale using the ***grayscale(image)*** function
2. Apply gaussian blur using ***gaussian_blur(img, kernel_size)*** function to smooth the image
3. Detect the edges using canny edge detection function ***canny(img, low_threshold, high_threshold)***
4. Find the region of interest using ***region_of_interest(img, vertices)*** function to identify edges containing only the lane lines
5. Draw hough lines using ***hough_lines(img, rho, theta, threshold, min_line_len, max_line_gap)*** function
6. Finally, overlay the image with hough lines over the original image using ***weighted_img(img, initial_img)*** function

In order to draw a single solid line on both the left and right lanes, I modified the draw_lines() by splitting the lane lines into left and right lines by slope using ***find_slope(x1, y1, x2, y2)*** function

```
def find_slope(x1, y1, x2, y2):
    return ((y2 - y1) / (x2 - x1))
```

Then, I eliminated the vertical lines

```
for line in lines:
        for x1,y1,x2,y2 in line:
            if x1 == x2:
                continue
```
Then, I discarded lines with impossible slopes

```
if abs(slope) < .5 or abs(slope) > .9: continue
```
Next, I split the lines based on slope

```
if(slope > 0):
                x1_left.append(x1)
                x2_left.append(x2)
                y1_left.append(y1)
                y2_left.append(y2)
            else: 
                x1_right.append(x1)
                x2_right.append(x2)
                y1_right.append(y1)
                y2_right.append(y2)
```

Then, I calculated the slope and intercept of the averaged lines using numpy's polyfit function

```
x1_right_avg = int(np.mean(x1_right))
        x2_right_avg = int(np.mean(x2_right))
        y1_right_avg = int(np.mean(y1_right))
        y2_right_avg = int(np.mean(y2_right))
        [slope_right, intercept_right] = np.polyfit((x1_right_avg, x2_right_avg), (y1_right_avg, y2_right_avg), 1)
```
```
x1_left_avg = int(np.mean(x1_left))
        x2_left_avg = int(np.mean(x2_left))
        y1_left_avg = int(np.mean(y1_left))
        y2_left_avg = int(np.mean(y2_left))
        [slope_left, intercept_left] = np.polyfit((x1_left_avg, x2_left_avg), (y1_left_avg, y2_left_avg), 1)
```
Based on the slope and intercept of the left and right lines identified above, I have calculated the respective line points

```
right_y1 = image.shape[0]
        right_y2 = int(right_y1 * 0.6)
        right_x1 = int((right_y1 - intercept_right) / slope_right)
        right_x2 = int((right_y2 - intercept_right) / slope_right)
```
```
left_y1 = image.shape[0]
        left_y2 = int(left_y1 * 0.6)
        left_x1 = int((left_y1 - intercept_left) / slope_left)
        left_x2 = int((left_y2 - intercept_left) / slope_left)
```
Finally, I plotted the lines on the image using opencv

```
cv2.line(img, (right_x1, right_y1), (right_x2, right_y2), color, thickness)
cv2.line(img, (left_x1, left_y1), (left_x2, left_y2), color, thickness)
```
**Potential Short Comings**

One potential shortcoming would be what would happen when the lines are curved. As seen in the challenge video, the lane lines are all over the place when they are curved. The current pipeline fits only the straight lines

Another shortcoming would be the current pipeline might have trouble identifying lanes on steep roads as the region of interest is masked to around the center of the image

**Possible Improvements**

To detect curves effectively, fit the lines to polynomials of higher degree

Modify the pipeline for smoother lines across video frames i.e. to avoid bouncing lines in the video


