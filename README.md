# Analogue-gauge-reader
A model to read multiple Analog gauge reading using computer Vision

# Premise:
Analogue pointer-type dials are widely applied in industry like Oil&Gas sector and many operation industry. Visual inspection is necessary to obtain readings from analog gauges. This means a human operator must travel to the gauge’s location, read its current value, and log that value to enable the data to be used elsewhere. This is a time consuming and expensive process and it is prone to human errors. An alternative to the manual readings is to record a video of the gauge and analyze the pressure variations using computer vision techniques. I had an opportunity to take on one such project.

# Approach:
Placing a camera in close contact to a “Analogue Gauge” and using the feed to analyze frames at desired frequency. I concentrated on building the model on single frame and can be used with “VideoCapture(i)” to access the camera feed, but that’s not the goal for now. 
Lets dive into it.

# Analogy behind:
Before going to the details of process, lets take a look at “Hough Space” concept which will be very helpful in understanding “Shape detection” like line, ellipse or circle detection.

# HOUGH SPACE:
Representation of Lines in the Hough Space Lines can be represented uniquely by two parameters. Often the form in Equation 1 is used with parameters a and b.
Finding lines in an image: Hough Space
y = m0· x + b0 (1)

Connection between Image (x,y) and Hough (m,b) spaces:
A line in the image corresponds to a point in Hough space.
To go from image space to Hough space: given a set of points (x,y), find all (m,b) such that y = mx + b.
Point (xo,yo)

Connection between image (x,y) and Hough (m,b) spaces :
A line in the image corresponds to a point in Hough space
To go from image space to Hough space: given a set of points (x,y), find all (m,b) such that y = mx + b
What does a point (xo , yo ) in the image space map to?
Answer: the solutions of b= -x0m + yo
this is line in Hough space.

Line parameters for the line connecting point(x1,y1) and (xo,yo) can be seen in Hough spaces as the intersection of both the lines represented by points(dots) in image space.

Hough Transform is a voting technique that can be used to answer all of these questions.
Main idea:
Record vote for each possible line on which each edge point lies.
Look for lines that get many votes.
How can we use this to find the most likely parameters (m,b) for the most prominent line in the image space?
Let each edge point in image space vote for a set of possible parameters in Hough space
Accumulate votes in discrete set of bins; parameters with the most votes indicate line in image space.

So the algorithm goes like:
Write each line formed by a pair of these points as yi = axi + b
Find a subset of n points on an image that lie on the same straight line. Then plot them on the parameter space (a, b): b = xi a + yi
All points (xi , yi ) on the same line will pass the same parameter space point (a, b).
Quantize the parameter space and tally # of times each points fall into the same accumulator cell. The cell count = # of points in the same line.
There are some issues with usual (m,b) parameter space so we also have polar representation for lines, which I will go through in details in the next article for “Panel gauge reader”.
Getting back to the topic, I approached the problem in five steps:
# 1. Circles detection (dial): 
OpenCV has a function for detecting circles, called HoughCircles. In the circle case, we need three parameters to define a circle:

Analogue gauge: Dial and angle imposed.
Arguments:
gray: Input image (grayscale)
cv2.hough_gradient: Define the detection method. Currently this is the only one available.
dp = 1: The inverse ratio of resolution
min_dist = 20: Minimum distance between detected centers
param_1 = 100: Upper threshold for the internal Canny edge detector
param_2 = 50: Threshold for center detection.
min_radius : Minimum radio to be detected. If unknown, put zero as default.
max_radius : Maximum radius to be detected. If unknown, put zero as default

# 2. Line detection (needle): 
Probabilistic Hough Transform is an optimization of Hough transform we saw.

It doesn’t take all the points into consideration, instead take only a random subset of points and that is sufficient for line detection. we have to reiterate while decreasing/increasing the threshold.
Arguments:image — 8-bit, single-channel binary source image. The image may be modified by the function.
lines — Output vector of lines. Each line is represented by a 4-element vector(x1,y1,x2,y2) where (x1,y1)&(x2,y2) are ending points of lines.
rho — Distance resolution of the accumulator in pixels.
theta — Angle resolution of the accumulator in radians.
threshold — Accumulator threshold parameter. Only those lines are returned that get enough votes
minLineLength — Minimum line length. Line segments shorter than that are rejected.
maxLineGap — Maximum allowed gap between points on the same line to link them.
3. Contour Detection: Now I have coordinates of center of the circle and the needle. So what I need now is the angle at which the dial reading starts and ends. That will give almost exact value of the needle reading.

I thought initially to proceed with manual calibration but it would be “awesome” if the model has zero user interaction and does the calibration by itself. So I used contour detection to find the contours and set some conditions to find the first contour clockwise and the last one, which will automatically give me the start and the end corresponding angle.

Start and end Angle
4. Angle determination: To determine the final angle of the needle, we already have everything required. We have already defined a “len” function to find the x_len and y_len of point(x,y) determined on the needle.
final_angle= ArcTan(abs(y_len/x_len))

Note: Be careful to put a check on constrain like which quadrant it might lie and make changes accordingly like:
5. Convert angle to readings: This would be the simplest of all steps like we used to do “Unitary” method when we were kid like if 100 unit cost 1000 bucks then how much will 35 units cost.

# “VOILA!!”
We have got a pretty much accurate reading of what we were looking for.
“After all the EYE sees it all!!!!!”

Note: Please look for my github for all the code and I will be updating the second version of analogue gauge reader ie “PANEL gauge reader”. It should be able to detect all the gauges on a panel and detect the reading of each gauge.
References:

1. https://gfycat.com/warmheartedpolitegoldfinch
2. https://objectcomputing.com/resources/publications/sett/june-2019-using-machine-learning-to-read-analog-gauges
3. http://dept.me.umn.edu/courses/me5286/vision/Notes/2015/ME5286-Lecture9.pdf
4. https://github.com/intel-iot-devkit/python-cv-samples/tree/master/examples/analog-gauge-reader
