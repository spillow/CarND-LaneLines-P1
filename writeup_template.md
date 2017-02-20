#**Finding Lane Lines on the Road**

##Writeup Template

---

[//]: # (Image References)

[image1]: ./examples/gray.png "Grayscale"
[image2]: ./examples/blur.png "Blur"
[image3]: ./examples/canny.png "Canny"
[image4]: ./examples/region.png "Region"
[image5]: ./examples/hough.png "Hough"
[image6]: ./test_images/lightroad.jpg "LightRoadColor"
[image7]: ./examples/lightroadgray.png "LightRoadGray"
[image8]: ./examples/annotated.png "Annotated"

---

### Reflection

###1. Pipeline description

The lane finding pipeline consists of 5 steps:

* Convert to grayscale
  * ![alt text][image1]
* Gaussian blur
  * preprocessing step to smooth out noise
  * ![alt text][image2]
* Canny edge detection
  * find edges via gradient calculation
  * ![alt text][image3]
* Region masking
  * static region of interest to disregard edges outside of lane (e.g., sky, other cars)
  * ![alt text][image4]
* Hough lines
  * find line segments from canny
  * ![alt text][image5]
* Smoother
  * clustering of segments, centroid calculation, outlier rejection

Ultimately resulting in lane annotation:
  * ![alt text][image8]


In order to draw a single line on the left and right lanes, I implemented a function (smoother()) that
takes a collection of lines out of the hough transform and emits two lines to represent the lanes.

It first transforms lines into hough space and then does k-means clustering (k=2) to identify groups
associated with both lane lines.  Slopes are thresholded to remove artifacts (such as the hood of the car and horizontal shadows from the challenge video) and some outliers are removed to eliminate nearly parallel
to the road lines from, for example, the shoulder of the road.


###2. Potential shortcomings with current pipeline

* The challenge video demonstrates that the pipeline currently has difficulty with contrast issues such as lighting
that can create shadows and low contrast between lightly colored road and lane lines.  Adverse weather
conditions would also be difficult (e.g., raindrops on camera, reflections from the road).

* The static region mask will have a hard time with road that is not flat.  For example, when going down
a hill the first road seen will be further off in the distance.  what do we do when no road is visible
at all?

* It works well with plenty of space in front of it, but a car that merges in front will create edges
that can throw off the lane detection.

###3. Suggest possible improvements to your pipeline

* I experimented with RGB normalization, histogram equalization and adaptive thresholding (which doesn't interact well with canny) to detect the challenge video lane on the light road but it still washed out.  For example:
  * ![alt text][image6]
Is fairly visible with color but in grayscale:
  * ![alt text][image7]
takes some effort to discern even with human vision.  Utilizing color information may be useful here.

* Even though the lanes track well, there is still some jitter I'd like to remove from frame to frame.
Perhaps use of interframe motion estimation would help correlate lanes from frame to frame.

* It may be useful to model curves (even if by a set of line segments) for more bendy roads where the
current single line would diverge from the true path too much in the distance.

