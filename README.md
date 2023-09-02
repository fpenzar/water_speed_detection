# Water flow speed detection

Determine the water (river) flow speed from input video. The camera should be static and perpendicular to the water flow.
Algorithm was developed for Geolux (https://www.geolux-radars.com/).

## Algorithm

### Image preprocessing

The image preprocessing follows these steps:
* extract a frame from the video
* frame is converted to grayscale image
* averaging/gaussian filter on the image
* absolute difference between the previous frame and the current frame
* Canny edge detection

After the preprocessing, a binary image is obtained. Background is black, waves and moving edges are white (`blob_features`).

### Extracting blob features

From the binary image, only white pixels are considered. A `blob_feature` is a structure representing all connected white pixels (a single wave, a single edge). Two white pixels are connected if they are neighbours (distance of 1), on the diagonals or to the left, right, up or down.

### Pairing features

A `feature` is a collection of similiar `blob_features` in time. When a new `blob_feature` is spotted it is either connected to an already existing `feature` or a new `feature` is created from it. If `blob_feature` is not found for `feature` in the current frame, its new position is extrapolated from the previous data, adding a new point.

### Scoring

Each `feature` has a score associated with it. `score` is increased for every paired `blob_feature` per frame. Score is decreased for every frame in which `blob_feature` for it is not found (even though the new position is extrapolated). If `score` drops to 0, the `feature` is deleted.

### Calculating speed

After every frame, speed for all active (`score` > `score_threshold`) `features` is calculated. It is stored inside a list, along with the current `score` of the corresponding `feature`. After one second of video, the overall speed is calculated. It is calculated as the weighted sum of the `speed` list based on the `score` associated with each `speed`. After each second, the data from the list is emptied.  

### Determining the flow direction

If the flow detection setting is set to `autodetect`, the algorithm automatically determines in which direction the water is flowing: `up` or `down`. It keeps track of both `features` going up (will only pair `blob_feature` to `feature` if it is higher up) and `features` going down. The algorithm determines the direciton based on the number of paired `features` going up vs paired `features` going down. A scoring system is implemented to avoid sudden direction changes.

### Determining the search radius

The search radius greatly affects the results. If it is too big, wrong `blob_features` are paired with `features` making the calculated speed too big. `mean` and `median` speed is calculated after every second for the `speed` list. It was observed that when search radius is not correct, `mean` and `median` are very different. When the search radius is correct, `mean` and `median` are roughly the same. The search radius is adjusted proportionally to `mean` - `median` relation. 

### Results

![ezgif com-optimize](https://github.com/fpenzar/water_speed_detection/assets/49754912/a4fc9d14-72e6-4869-8ba2-2c1f9e8a9d3a)

### Behind the curtains

![ezgif com-video-to-gif](https://github.com/fpenzar/water_speed_detection/assets/49754912/b8f2a034-7e73-4d1f-8c58-28a304510109)

### Benchmarking

In % of execution time:

* `image processing` -> 88.4%
* `reading frames` -> 10.1%
* `extracting blobs` -> 1.0%
* `pairing features` -> 0.5% 
