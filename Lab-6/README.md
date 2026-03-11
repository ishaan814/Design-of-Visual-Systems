# Lab 6 - Feature Matching and Classification

This lab covers image resizing and pyramid construction, template matching using normalised cross-correlation, SIFT feature detection and matching, a comparison of SIFT and SURF for object tracking, and finally live image classification using a pre-trained neural network. It supports the content of Lectures 10 and 11.

---

## Contents
- Task 1: Image Resizing
- Task 2: Pattern Matching with Normalised Cross Correlation
- Task 3: SIFT Feature Detection
- Task 4: SIFT Matching - Scale and Rotation Invariance
- Task 5: SIFT vs SURF
- Task 6: Image recognition using neural networks

---

### Task 1: Image Resizing - Image Pyramid

An image pyramid is a multi-scale representation of an image produced by repeatedly downsampling the image. Here we compare two approaches: naive subsampling by dropping pixels, and proper resizing using `imresize`.

```matlab
clear all; close all;

f = imread('cafe_van_gogh.jpg');

% --- Method 1: Naive subsampling by dropping every other row and column ---
% No filtering before subsampling - can introduce aliasing artefacts
f2   = f(1:2:end, 1:2:end, :);     % 1/2 scale
f4   = f(1:4:end, 1:4:end, :);     % 1/4 scale
f8   = f(1:8:end, 1:8:end, :);     % 1/8 scale
f16  = f(1:16:end, 1:16:end, :);   % 1/16 scale
f32  = f(1:32:end, 1:32:end, :);   % 1/32 scale

figure(1)
montage({f, f2, f4, f8, f16, f32}, 'Size', [2 3]);
title('Image Pyramid: Naive Subsampling (no filtering)');
```
<p align="center">
<img width="590" height="451" alt="image" src="https://github.com/user-attachments/assets/e1c8ff32-bb87-4bfb-9e04-f0caaf051846" />
</p>

MATLAB's colon syntax `start:increment: end` is used to select every nth row and column. For a factor of 2, `1:2:end` selects rows 1, 3, 5, …, effectively keeping every other row and discarding the rest. Applying this to both dimensions simultaneously reduces the image to 1/4 of its original pixel count (half in each dimension). The same idea is applied with increments of 4, 8, 16 and 32 to produce the remaining pyramid levels. Because no filtering is applied beforehand, high-frequency content that cannot be represented at the lower resolution creates aliasing, visible as jagged edges and moiré patterns at smaller scales.

```matlab
% --- Method 2: Proper resizing using imresize ---
% imresize applies a Gaussian lowpass filter before subsampling to prevent aliasing
g2  = imresize(f, 0.5);
g4  = imresize(f, 0.25);
g8  = imresize(f, 0.125);
g16 = imresize(f, 1/16);
g32 = imresize(f, 1/32);

figure(2)
montage({f, g2, g4, g8, g16, g32}, 'Size', [2 3]);
title('Image Pyramid: Proper Resizing with imresize (anti-aliased)');
```
<p align="center">
<img width="590" height="451" alt="image" src="https://github.com/user-attachments/assets/199f9fc2-ccbf-4c76-a68a-2a572cad9e56" />
</p>

```matlab
% Compare the two methods at 1/8 scale side by side
figure(3)
montage({f8, g8});
title('1/8 Scale Comparison: Naive Subsampling | imresize');
```
<p align="center">
<img width="590" height="353" alt="image" src="https://github.com/user-attachments/assets/0aafd7ff-5a02-4304-9021-c583643c6452" />
</p>

`imresize` first applies a Gaussian lowpass filter to the image before subsampling. The filter removes frequency components above the Nyquist limit of the new lower resolution, preventing them from folding back into the image as aliasing artefacts. The side-by-side comparison at 1/8 scale makes this difference clear: the `imresize` result is smoother and retains the painted brushstroke texture faithfully, while the naive result shows ringing and structural distortion, particularly at fine edges.

---

### Task 2: Pattern Matching with Normalised Cross Correlation

Normalised cross-correlation (NCC) is used to locate a small template patch within a larger image by measuring how closely the template matches every possible position in the image.

```matlab
clear all; close all;

f = imread('salvador_grayscale.tif');
w = imread('template1.tif');        % template to search for

% Compute normalised cross correlation between template and image
c = normxcorr2(w, f);

% 3D surface plot of the NCC response
figure(1)
surf(c)
shading interp
title('NCC Surface Plot - template1');
```
<p align="center">
<img width="590" height="442" alt="image" src="https://github.com/user-attachments/assets/99e47c86-a27b-41ee-8d5f-e1741e5e9a4b" />
</p>

`normxcorr2(w, f)` slides the template `w` across every position in the image `f` and computes the normalised cross-correlation at each location. The output `c` has values in the range [−1, 1], where 1 indicates a perfect match, and 0 indicates no correlation. The `surf` plot visualises `c` as a 3D landscape; the location of the sharp peak corresponds to where the template best matches the image. Normalisation means the result is independent of absolute brightness, making it robust to uniform illumination changes.

```matlab
% Automatically locate the peak (exact match location)
[ypeak, xpeak] = find(c == max(c(:)));
yoffSet = ypeak - size(w, 1);      % correct for template size offset
xoffSet = xpeak - size(w, 2);

% Overlay a rectangle on the image at the detected location
figure(2)
imshow(f)
drawrectangle(gca, 'Position', ...
    [xoffSet, yoffSet, size(w,2), size(w,1)], 'FaceAlpha', 0);
title('Template 1 Match Location');
```
<p align="center">
<img width="590" height="378" alt="image" src="https://github.com/user-attachments/assets/9569fb36-d3a5-46b3-a08d-9757838e7615" />
</p>

`find(c == max(c(:)))` returns the row and column index of the maximum value in `c`. Because `normxcorr2` pads the output to account for the template size, the true top-left corner of the match in the original image is obtained by subtracting the template dimensions from the peak coordinates. `drawrectangle` then overlays a bounding box of the same size as the template at the detected location, visually confirming the match.

```matlab
% Repeat with template 2
w2 = imread('template2.tif');
c2 = normxcorr2(w2, f);

figure(3)
surf(c2)
shading interp
title('NCC Surface Plot - template2');
```
<p align="center">
<img width="590" height="442" alt="image" src="https://github.com/user-attachments/assets/c1f81195-3b65-4e24-adfe-e2b7d6e1b636" />
</p>

```matlab
[ypeak2, xpeak2] = find(c2 == max(c2(:)));
yoffSet2 = ypeak2 - size(w2, 1);
xoffSet2 = xpeak2 - size(w2, 2);

figure(4)
imshow(f)
drawrectangle(gca, 'Position', ...
    [xoffSet2, yoffSet2, size(w2,2), size(w2,1)], 'FaceAlpha', 0);
title('Template 2 Match Location');
```
<p align="center">
<img width="590" height="378" alt="image" src="https://github.com/user-attachments/assets/813cb736-fcab-483e-a776-6b0ff9feb479" />
</p>

The same procedure is repeated for `template2.tif`. NCC works well when the template is an exact or near-exact crop of the target image, same scale, same orientation, same lighting. If the object appears at a different scale or has been rotated, the NCC value at that location drops significantly, and the match may fail. This is the fundamental limitation of NCC that motivates the use of scale- and rotation-invariant feature descriptors such as SIFT in the following tasks.

---

Task 3: SIFT Feature Detection

SIFT (Scale-Invariant Feature Transform) detects interest points that are stable across changes in scale, rotation, and illumination, making it far more powerful than NCC for general matching tasks.

```matlab
clear all; close all;

% --- SIFT on the Dali painting ---
I = imread('salvador.jpg');
f = im2gray(I);

% Detect SIFT interest points
points = detectSIFTFeatures(f);

figure(1); imshow(I);
hold on;
plot(points.selectStrongest(100));  % overlay 100 strongest keypoints
title('SIFT: 100 Strongest Features - Salvador Dali');

% Explore the points structure
fprintf('Total SIFT points detected: %d\n', points.Count);
fprintf('Fields in points: Location, Scale, Orientation, Metric\n');
```
<p align="center">
<img width="590" height="378" alt="image" src="https://github.com/user-attachments/assets/08566f25-3542-4784-93f1-ef24faf7a50d" />
</p>

`detectSIFTFeatures` builds a scale-space representation of the image using a Gaussian pyramid and identifies keypoints at locations where the Difference-of-Gaussians (DoG) response is a local extremum across both space and scale. Each keypoint in `points` stores its pixel `Location`, `Scale` (the size of the surrounding region), `Orientation` (dominant gradient direction), and `Metric` (response strength). `selectStrongest(100)` returns the 100 keypoints with the highest metric values, these are overlaid as oriented circles on the image, where the circle radius indicates scale and the line inside indicates orientation.

```matlab
% --- SIFT on the Van Gogh painting ---
I2 = imread('cafe_van_gogh.jpg');
f2 = im2gray(I2);
points2 = detectSIFTFeatures(f2);

figure(2); imshow(I2);
hold on;
plot(points2.selectStrongest(100));
title('SIFT: 100 Strongest Features - Cafe Van Gogh');

fprintf('Total SIFT points (Van Gogh): %d\n', points2.Count);
```
<p align="center">
<img width="590" height="618" alt="image" src="https://github.com/user-attachments/assets/e2eb9ed6-fc64-4b56-95a9-f29c59671180" />
</p>

The Van Gogh painting has highly textured brushwork with strong local contrast in many regions, so SIFT detects a large number of interest points spread across the canvas. The strongest 100 tend to cluster around high-contrast areas such as the lanterns, window frames and the edge of the terrace canopy, regions that are geometrically distinctive enough to be reliably re-detected under different conditions.

---

Task 4: SIFT Matching - Scale and Rotation Invariance

This task demonstrates that SIFT can match features between two versions of the same image at different scales, and even after rotation.

```matlab
clear all; close all;

I1 = imread('cafe_van_gogh.jpg');
I2 = imresize(I1, 0.5);            % half-size version of same image
f1 = im2gray(I1);
f2 = im2gray(I2);

% Detect SIFT features in both images
points1 = detectSIFTFeatures(f1);
points2 = detectSIFTFeatures(f2);

Nbest = 100;
bestFeatures1 = points1.selectStrongest(Nbest);
bestFeatures2 = points2.selectStrongest(Nbest);

% Display the strongest features on each image
figure(1); imshow(I1);
hold on; plot(bestFeatures1); hold off;
title('SIFT Features: Full Resolution');
```
<p align="center">
<img width="590" height="618" alt="image" src="https://github.com/user-attachments/assets/7a803960-09cd-4773-9d01-00d010c25c90" />
</p>

```matlab
figure(2); imshow(I2);
hold on; plot(bestFeatures2); hold off;
title('SIFT Features: Half Resolution (0.5x)');
```
<p align="center">
<img width="504" height="509" alt="image" src="https://github.com/user-attachments/assets/6063c57b-d93d-4e01-9c1d-68072da139b9" />
</p>

The two images are the same scene at different resolutions, one full size and one at half size. SIFT detects interest points at multiple scales internally, so the keypoints found in the half-size image correspond to the same physical locations as those found in the full-size image, just at a different scale level. The circle radii in the plot should roughly double between the half-size and full-size images for corresponding points.

```matlab
% --- Extract descriptors and match features using all points ---
[features1, valid_points1] = extractFeatures(f1, points1);
[features2, valid_points2] = extractFeatures(f2, points2);

indexPairs = matchFeatures(features1, features2, 'Unique', true);

matchedPoints1 = valid_points1(indexPairs(:,1),:);
matchedPoints2 = valid_points2(indexPairs(:,2),:);

figure(3);
showMatchedFeatures(f1, f2, matchedPoints1, matchedPoints2);
title('Matched Features: All Points');
```
<p align="center">
<img width="590" height="618" alt="image" src="https://github.com/user-attachments/assets/eb91a136-7760-4f3b-a52c-11222908fbdd" />
</p>

`extractFeatures` computes a 128-dimensional descriptor vector for each keypoint by building a histogram of gradient orientations in a 4×4 grid of cells around the keypoint, normalised by the keypoint's dominant orientation. `matchFeatures` then compares descriptor vectors between the two images using Euclidean distance and finds the best unique pairing for each descriptor. `showMatchedFeatures` draws lines connecting the matched keypoints side by side. When using all detected points (thousands per image), the result is an explosion of lines in every direction; the sheer volume of weak, non-distinctive keypoints produces enormous numbers of false matches, making the plot unreadable. This motivates restricting to only the strongest points.

```matlab
% --- Now use only the Nbest strongest points ---
[features1b, valid_points1b] = extractFeatures(f1, bestFeatures1);
[features2b, valid_points2b] = extractFeatures(f2, bestFeatures2);

indexPairs2 = matchFeatures(features1b, features2b, 'Unique', true);

matchedPoints1b = valid_points1b(indexPairs2(:,1),:);
matchedPoints2b = valid_points2b(indexPairs2(:,2),:);

figure(4);
showMatchedFeatures(f1, f2, matchedPoints1b, matchedPoints2b);
title('Matched Features: Strongest 100 Points Only');
```
<p align="center">
<img width="590" height="618" alt="image" src="https://github.com/user-attachments/assets/06949a10-717f-46f3-8da5-6dd38e0e4b1a" />
</p>

Restricting matching to the strongest 100 points reduces the number of matches but improves their reliability; weaker keypoints are more likely to produce false matches because their descriptors are less distinctive. Using only the strongest features, therefore, produces a cleaner set of matches with fewer outliers, at the cost of covering fewer regions of the image.

```matlab
% --- Rotation invariance: rotate f2 by 20 degrees ---
f2_rot = imrotate(f2, 20);
I2_rot = imrotate(I2, 20);

points2_rot = detectSIFTFeatures(f2_rot);
[features2_rot, valid_points2_rot] = extractFeatures(f2_rot, points2_rot);

indexPairs_rot = matchFeatures(features1, features2_rot, 'Unique', true);

matchedPoints1_rot = valid_points1(indexPairs_rot(:,1),:);
matchedPoints2_rot = valid_points2_rot(indexPairs_rot(:,2),:);

figure(5);
showMatchedFeatures(f1, f2_rot, matchedPoints1_rot, matchedPoints2_rot);
title('SIFT Rotation Invariance: Full vs 20-degree Rotated Half-size');
```
<p align="center">
<img width="590" height="618" alt="image" src="https://github.com/user-attachments/assets/409f8dfd-6647-4a97-8c9e-372301c4ac0b" />
</p>

`imrotate(f2, 20)` rotates the half-size image by 20 degrees. Because each SIFT descriptor is computed relative to the keypoint's dominant gradient orientation, the descriptor is inherently rotation-invariant; the same physical location in the scene produces the same descriptor regardless of how the image is oriented. The matching result shows that SIFT can still find correct correspondences between the full-size upright image and the rotated half-size image, demonstrating both scale and rotation invariance simultaneously.

---

Task 5: SIFT vs SURF

SIFT and SURF are compared on two successive frames of motorway traffic footage to evaluate their suitability for object tracking between video frames.

```matlab
clear all; close all;

I1 = imread('traffic_1.jpg');
I2 = imread('traffic_2.jpg');
f1 = im2gray(I1);
f2 = im2gray(I2);

% --- SIFT matching between the two traffic frames ---
points1_sift = detectSIFTFeatures(f1);
points2_sift = detectSIFTFeatures(f2);

[feat1_sift, vp1_sift] = extractFeatures(f1, points1_sift);
[feat2_sift, vp2_sift] = extractFeatures(f2, points2_sift);

pairs_sift = matchFeatures(feat1_sift, feat2_sift, 'Unique', true);

mp1_sift = vp1_sift(pairs_sift(:,1),:);
mp2_sift = vp2_sift(pairs_sift(:,2),:);

figure(1);
showMatchedFeatures(f1, f2, mp1_sift, mp2_sift);
title(sprintf('SIFT: %d matched pairs between traffic frames', size(pairs_sift,1)));
```
<p align="center">
<img width="1498" height="845" alt="image" src="https://github.com/user-attachments/assets/f0f5b7fd-3125-473c-8b4c-291e89e2681c" />
</p>

Between two successive video frames, cars move only a small distance, so corresponding features should appear at nearby positions in each frame. SIFT detects interest points on the vehicles' licence plates, door edges, windows and matches them across frames. The matched lines connecting the two images should be roughly parallel and short, reflecting the small inter-frame motion. Any long or crossing lines indicate false matches.

```matlab
% --- SURF matching between the same two frames ---
points1_surf = detectSURFFeatures(f1);
points2_surf = detectSURFFeatures(f2);

[feat1_surf, vp1_surf] = extractFeatures(f1, points1_surf);
[feat2_surf, vp2_surf] = extractFeatures(f2, points2_surf);

pairs_surf = matchFeatures(feat1_surf, feat2_surf, 'Unique', true);

mp1_surf = vp1_surf(pairs_surf(:,1),:);
mp2_surf = vp2_surf(pairs_surf(:,2),:);

figure(2);
showMatchedFeatures(f1, f2, mp1_surf, mp2_surf);
title(sprintf('SURF: %d matched pairs between traffic frames', size(pairs_surf,1)));

% Print summary comparison
fprintf('SIFT matched pairs: %d\n', size(pairs_sift,1));
fprintf('SURF matched pairs: %d\n', size(pairs_surf,1));
```
<p align="center">
<img width="1498" height="845" alt="image" src="https://github.com/user-attachments/assets/be688520-6241-419c-a36a-8dc0a7a3ea2b" />
</p>

SURF (Speeded-Up Robust Features) approximates the SIFT approach using box filters and integral images, making it significantly faster to compute while remaining scale and rotation-invariant. However, in practice on traffic footage, SIFT substantially outperformed SURF, finding 655 matched pairs versus only 68 for SURF. This is because SIFT builds a richer 128-dimensional descriptor from gradient orientation histograms, making its descriptors more distinctive and less likely to produce false rejections during matching. SURF uses a 64-dimensional descriptor based on Haar wavelet responses, which is faster to compute but less discriminative in scenes with repetitive texture, such as road markings and car bodywork. Both methods are performing what is essentially object tracking: finding the same physical points in successive video frames, which is the foundation of visual odometry, optical flow, and video stabilisation.

---

Task 6: Image Recognition Using Neural Networks

A pre-trained deep neural network is used to classify objects captured live from a webcam.

```matlab
clear all; close all;

camera = webcam;                            % create webcam object

% Load a pre-trained network - try googlenet, fall back to alexnet if not installed
try
    net = googlenet;                        % requires Deep Learning Toolbox Model for GoogLeNet
catch
    warning('googlenet not installed, falling back to alexnet.');
    net = alexnet;                          % alexnet is more commonly available
end

inputSize = net.Layers(1).InputSize(1:2);   % get required input image size

figure
I = snapshot(camera);                       % capture a frame from webcam
image(I);
f = imresize(I, inputSize);                 % resize to match network input size
tic;                                        % start timer
[label, score] = classify(net, f);          % classify image with network
toc                                         % report elapsed time
title({char(label), num2str(max(score), 2)});
```
<p align="center">
<img width="840" height="630" alt="image" src="https://github.com/user-attachments/assets/ce674c42-4521-4e50-a392-f54aa85bfd0a" />
</p>

The `try/catch` block attempts to load GoogLeNet first. This requires the Deep Learning Toolbox Model for GoogLeNet support package to be installed separately via the Add-On Explorer. If it is not available, it falls back to AlexNet, which is more commonly pre-installed with the Deep Learning Toolbox. Both networks are trained on the ImageNet dataset with 1000 object categories. The first layer's `InputSize` specifies the spatial resolution the network expects, the captured image must be resized to this before being passed in. `classify` feeds the resized image through all network layers and returns the predicted class label and a confidence score between 0 and 1. `tic`/`toc` measure the inference time, which varies by network complexity and hardware.

```matlab
% --- Continuous recognition loop ---
figure
while true
    I = snapshot(camera);                   % capture frame
    image(I);
    f = imresize(I, inputSize);
    tic;
    [label, score] = classify(net, f);
    toc
    title({char(label), num2str(max(score), 2)});
    drawnow;                                % update figure without blocking loop
end
```

The continuous loop captures a new frame, classifies it, and updates the figure title with the predicted label and confidence on every iteration. `drawnow` forces MATLAB to refresh the figure display before the next loop iteration; without it, the figure would not update while the loop is running. Different networks (AlexNet, ResNet, GoogLeNet) trade off between speed and accuracy: shallower networks like AlexNet run faster but with lower accuracy, while deeper networks like GoogLeNet are slower but more reliable. Pointing the camera at everyday objects and observing the label and confidence score gives an intuitive sense of the network's capabilities and failure modes.

---
