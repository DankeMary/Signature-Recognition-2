# Signature Recognition
*Recognizing handwritten signatures of individuals accurately considering that they might be forged*

I have used Open CV implementation of different algorithms for feature extraction and comparison

### Following methods are used to compute feature point descriptors:

1. **Binary Robust Independent Elementary Features (BRIEF)**
	* We need image descriptors that are fast to compute, match and are memory efficient
	* It provides a shortcut to find the binary strings directly without finding descriptors
	* It takes smoothened image patch and selects a set of n (x,y) location pairs in a unique way
	* Then on those location pairs (p and q), it calculates intensity comparisons
	* If I(p) < I(q), then its result is 1, else it is 0
	* This is applied for all the n location pairs to get a n-dimensional bitstring
	* This n can be 128, 256 or 512 bits (OpenCV has default value of 32 bytes = 256 bits)
	* Comparing strings can be done using the Hamming distance, which is very efficient to compute (instead of the L2 norm as is usually done)

<p align="center">
<img src="https://i.imgur.com/fo3ZxPJ.png">
</p>
<p align="center"><sup><sub>Image taken from original research paper</sub></sup></p>
<p align="center">Fig: Different approaches on choosing the test locations (n = 128 bits)</p>

* More details can be found in this Research paper [Binary Robust Independent Elementary Features (BRIEF)](https://www.cs.ubc.ca/~lowe/525/papers/calonder_eccv10.pdf)

2. **Oriented FAST and Rotated BRIEF (ORB)**
	* SIRF and SURF are patented and you need to pay for their usage
	* ORB is an efficient alternative to SIFT or SURF (as the title of the paper suggests)
	* ORB uses FAST keypoint detectors
		* Keypoints are the interest points, which decscribe what is interesting or stands out in the image. 
		* Keypoints are important because no matter how the image changes (rotates, expands/shrinks, distorted etc), the keypoints in the original and modified image should be the same.
		* When finding the keypoints in the modified image, the orientation of the keypoints might be changed.
		* To find the orientation of keypoints, ORB computes the intensity weighted centroid of the patch with located corner at center 
		* The direction of the vector from this corner point to centroid gives the orientation
		* The equations and more detailed information can be found in the heading **3.2. Orientation by Intensity Centroid** of the paper
	* ORB uses BRIEF descriptors
		* Descriptors are how you describe these keypoints
		* but we know that BRIEF works poorly with rotated images
		* so what ORB does is to “steer” BRIEF according to the orientation of keypoints
		* More information about steering the BRIEF can be found in the heading **4.1. Efficient Rotation of the BRIEF Operator** of the paper
	* For descriptors matching, multi probe LSH (Locality Sensitive Hashing) is used instead of traditional LSH
		* LSH is used for approximate similarity search
		* LSH hashes input items so that similar items map to the same “buckets” with high probability (the number of buckets being much smaller than the universe of possible input items)
		* The problem is the requirement for a large number of hash tables in order to achieve good search quality
		* Thus it intelligently probes multiple buckets that are likely to contain query results in a hash table
		* Multi-probe LSH uses less query time and 5 to 8 times fewer number of hash tables
		* Thus it is both space and time efficient compared to traditional LSH
		
<p align="center">
<img src="https://i.imgur.com/Ol73NDg.jpg">
</p>
<p align="center"><sup><sub>Image taken from <a href="http://www.vlfeat.org/overview/sift.html">VLFeat Toolbox Tutorial</a></sub></sup></p>
<p align="center">Fig: Scale and Orientation of Keypoints</p>

* More details can be found in this Research paper [ORB: An efficient alternative to SIFT or SURF](http://www.willowgarage.com/sites/default/files/orb_final.pdf)

2. **Scale Invariant Feature Transform (SIFT)**
	* Why SIFT
		* Some keypoint detectors are rotation invariant
		* But they are not scale invariant (example is Harris Corner & Edge detector)
		* Thus SIFT aims to provide scale invariant keypoint detection
	* Goals
		* Extracting distinct invariant features
			* to correctly match against a large database of features from many images
		* Invariance to image scaling and rotation
		* Robustness to
			* distortion
			* orientation
			* noise
	* Advantages
		* Provides local features (computation on different patches of the image instead of Global features which generalizes the whole image)
		* We can get many features even for smaller objects
		* Efficient (can have real-time implementation)
	* Extracting Keypoints
		* Scale space peak selection
			* Potential locations for finding features
		* Key point localization
			* Accurately locating the feature key points
		* Orientation Assignment
			* Assigning orientation to the key points
		* Key point descriptor
			* Describing the key point as a high dimensional vector
