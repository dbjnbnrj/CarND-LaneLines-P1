**Finding Lane Lines on the Road**

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline includes the following steps

* Step 1: Converting the image to grayscale
In the lessons we were taught color selection. However I observed that converting the image to grayscale was sufficient enough for me to identify the yellow and white lines on the road
gray = grayscale(image)

[image1]: ./test_examples/grayscale.jpg "Grayscale"

* Step 2: Apply Gaussian Blur
I've added a minimum amount of Gaussian blur to get rid of noise. If too much Gaussian filtering is applied it will also smooth the edge, which is considered as the high frequency feature.

[image2]: ./test_examples/gaussian_blur.jpg "Gaussian Blur"

* Step 3: Apply Canny edge detection
We perform canny edge detection on the image to intensify the edges of the image and reduce the threshold. I've optimized the parameters for thresholding which is required to decide which edges are relevant.

[image3]: ./test_examples/canny_edge.jpg "Canny Edge"

* Step 4: Getting region of interest
I only care about the region of interest so I've marked out a polygon which helps me isolate only the lane lines that I'm considering.

[image4]: ./test_examples/roi.jpg "Region of Interest"

* Step 5: Run Hough transform
Hough transform is a feature extraction technique that maps the lines from coordinate space to parameter space and back again to isolate primary points that the each lane is composed of.

[image5]: ./test_examples/hough.jpg "Hough Transform"



* Modifying the draw lines function
Hough transform returned a collection of points, each tuple representing a line in coordinate space of our image.

Original draw_lines function took result of Hough transform and recreated those points on the image. This resulted in multiple small lines on the image which was sufficient for the first iteration.


[image5]: ./test_examples/draw_lines_1.jpg "Draw Lines 1"


My next task involved averaging and extrapolating out these lines. I proceeded with a simple brute force approach which involved

1. Looping through all the lines
2. For each pair of points constituting the lines
  - Calculate slope, intercept and length of the line
  - If the slope is negative this is the left lane. Make sure you multiply the slope+intercept with the length of the line. This serves as a weight for the line.
  - Else the line is assumed to be for the right lane
3. Average the slope and intercept you have calculated by dividing it by the overall weight for the left and right lanes
4. We know from the set of images the Y-coordinates for the line.
5. Using these known Y-coordinates we calculate X coordinates for the line by using the formula $X = (Y - Intercept) / Slope$.
6. We draw the line onto the image using the cv2 function

Below is the resultant image -



[image6]: ./test_examples/draw_lines_2.jpg "Draw Lines 2"


Finally we superimpose the result of Hough Transform on the original image to display our frames.

[image5]: ./test_examples/final.jpg "Final Image"


### 2. Identify potential shortcomings with your current pipeline
I make a number of assumptions in my pipeline
- My dataset is limited and contains images from a specific highway at a specific angle. This makes setting values for region of interest etc easy. I've hardcoded these values for now. These can be removed in the future
- I have not used color selection as I did not feel it was required for the given dataset. In future datasets it may be a crucial step in eliminating noise. I feel canny edge, gaussian blur and region of interest extraction already did this for me
- My current solution will be unable to handle a number of cases for the lanes
  - It will not be able to handle the optional case as lane lines. The line appears to be hopping from the leftmost lane onto the divider
  - It will not handle curved lines
  - It will not be able to extrapolate the lane very well if there are too many unknown objects (car etc) is in front of it. This can be dealt with by adding greater Gaussian Blur but that might interfere with the lane lines as well


### 3. Suggest possible improvements to your pipeline

I have researched and come across some functions which may greatly improve my current pipeline

- Contour Detection to isolate lane lines
  Contours are curve joining all the continuous points (along the boundary), having same color or intensity. Determining contour lines will be helpful for our lane detection task.

- Better Draw Lines Function
  My new draw lines function may not work efficiently all the time and is error prone. However my intuition is that the technique I have used is similar to Linear Regression so I can consider using scipy's linear regression library to improve the resultant line. For curved lines there is a function in the scipy library called scipy.optimize.curve_fit which will I think will work with more complex liens
