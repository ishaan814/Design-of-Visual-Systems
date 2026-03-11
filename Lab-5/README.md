# Lab 5 - Segmentation and Feature Detection

This lab explores techniques to identify features and regions in an image. We apply point detection, edge detection, the Hough Transform, thresholding, k-means clustering, and watershed segmentation, building on the theory from Lectures 8 and 9.


## Contents
- Task 1: Point Detection
- Task 2: Edge Detection
- Task 3: Hough Transform for Line Detection
- Task 4: Segmentation by Thresholding
- Task 5: Segmentation by k-means clustering
- Task 6: Watershed Segmentation with Distance Transform
- Task 7: Challenges

---

### Task 1: Point Detection

The goal is to isolate the individual stars in an image of the Crab Nebula (`crabpulsar.tif`) by filtering out the diffuse nebular background and detecting only isolated bright points.

```matlab
clear all
close all

f = imread('crabpulsar.tif');

% 8-connected Laplacian kernel - highlights pixels that differ from all neighbours
w = [-1 -1 -1;
     -1  8 -1;
     -1 -1 -1];

g1 = abs(imfilter(f, w));       % filter image, take absolute value
se = strel("disk", 1);          % small disk structuring element
g2 = imerode(g1, se);           % erode to sharpen point responses
threshold = 100;
g3 = uint8((g2 >= threshold) * 255);  % threshold to binary
montage({f, g1, g2, g3});
```

<p align="center">
<img width="593" height="535" alt="image" src="https://github.com/user-attachments/assets/a1a20701-453c-4340-a81a-1db62db70a31" />
</p> 

The kernel `w` is the 8-connected Laplacian (Lecture 8, Slide 5). It computes a second-order derivative at each pixel, a pixel that differs significantly from all 8 of its neighbours produces a large response, while flat or gradually varying regions produce a response near zero. `abs()` ensures both bright and dark isolated points are captured. `imerode` with a disk-shaped structuring element then sharpens the responses by removing weaker broad blobs, and the final threshold produces a binary image `g3` where white pixels mark the detected point features, the individual stars in the nebula.

---

### Task 2: Edge Detection

Three edge detection algorithms, Sobel, LoG, and Canny, are applied to a chip micrograph (`circuit.tif`) and a brain tumour MRI scan (`brain_tumor.jpg`).

```matlab
clear all
close all

f_circuit = imread('circuit.tif');

% Apply Sobel, LoG and Canny edge detectors to circuit image
[g_sobel_c, t_sobel_c] = edge(f_circuit, 'Sobel');
[g_log_c,   t_log_c]   = edge(f_circuit, 'log');
[g_canny_c, t_canny_c] = edge(f_circuit, 'Canny');

figure
montage({f_circuit, uint8(g_sobel_c*255), uint8(g_log_c*255), uint8(g_canny_c*255)});
title('Circuits: Original | Sobel | LoG | Canny');
```
<p align="center">
<img width="593" height="395" alt="image" src="https://github.com/user-attachments/assets/e89830bf-4f1e-4225-947a-307a911829ca" />
</p>

```matlab
f_brain = imread('brain_tumor.jpg');
if size(f_brain, 3) == 3
    f_brain = rgb2gray(f_brain);    % edge() requires single-channel input
end

% Apply same three detectors to brain tumour image
[g_sobel_b] = edge(f_brain, 'Sobel');
[g_log_b]   = edge(f_brain, 'log');
[g_canny_b] = edge(f_brain, 'Canny');

figure
montage({f_brain, uint8(g_sobel_b*255), uint8(g_log_b*255), uint8(g_canny_b*255)});
title('Brain Tumour: Original | Sobel | LoG | Canny');
```
<p align="center">
<img width="593" height="535" alt="image" src="https://github.com/user-attachments/assets/f2fe58c1-5a79-4484-9079-a20d431b4388" />
</p>

Sobel computes the first-order gradient using two 3×3 kernels (Lecture 8, Slide 8). It is fast and simple, but sensitive to noise and produces thick edges. LoG smooths the image with a Gaussian first to suppress noise, then applies the Laplacian and finds zero-crossings as edges (Lecture 8, Slides 9–12). Canny is the most robust: it applies Gaussian smoothing, computes gradient magnitude and direction, performs non-maximum suppression to thin edges to single-pixel width, and uses double thresholding with hysteresis to link edge segments (Lecture 8, Slides 13–17). The brain image is converted to greyscale before processing since `edge()` requires a single-channel input.

```matlab
% Tune thresholds for better results
[g_canny_c_tuned] = edge(f_circuit, 'Canny', [0.04 0.10]);
[g_log_c_tuned]   = edge(f_circuit, 'log', 0.003);

figure
montage({uint8(g_canny_c*255), uint8(g_canny_c_tuned*255)});
title('Circuits: Default Canny | Tuned Canny [0.04 0.10]');
```

<p align="center">
<img width="593" height="314" alt="image" src="https://github.com/user-attachments/assets/1efca3b9-73cf-4a47-8897-418f1339a102" />
</p>

```matlab
[g_canny_b_tuned] = edge(f_brain, 'Canny', [0.05 0.15]);

figure
montage({uint8(g_canny_b*255), uint8(g_canny_b_tuned*255)});
title('Brain: Default Canny | Tuned Canny [0.05 0.15]');
```
<p align="center">
<img width="593" height="307" alt="image" src="https://github.com/user-attachments/assets/f43a0fc8-57ce-44b7-8654-15683a654d12" />
</p>

For Canny, the two-element threshold vector sets the low and high hysteresis thresholds. Lowering the low threshold allows more weak edge segments to be accepted, recovering finer detail in gradual boundaries like those in the brain scan. For LoG, the scalar threshold controls how significant a zero-crossing must be before it is accepted as an edge; a lower value retains more edge detail at the cost of more noise.

---
### Task 3: Hough Transform for Line Detection

The Hough Transform is used to detect straight lines in a rotated circuit image (`circuit_rotated.tif`), following the theory from Lecture 8 Slides 20–26.

Steps 1 & 2: Edge detection and Hough Transform
```matlab
clear all; close all;

% Step 1: edge detection
f = imread('circuit_rotated.tif');
fEdge = edge(f, 'Canny');       % binary edge map feeds into Hough Transform
figure(1)
montage({f, fEdge})
```
<p align="center">
<img width="593" height="314" alt="image" src="https://github.com/user-attachments/assets/ad7a3c11-2124-47de-8a82-3cfb6891850f" />
</p>

```matlab
% Step 2: Hough Transform - maps edge pixels to (theta, rho) parameter space
[H, theta, rho] = hough(fEdge);
figure(2)
imshow(H, [], 'XData', theta, 'YData', rho, 'InitialMagnification', 'fit');
xlabel('theta'), ylabel('rho');
axis on, axis normal, hold on;
```
<p align="center">
<img width="563" height="422" alt="image" src="https://github.com/user-attachments/assets/18dce566-81fd-4df8-bd1e-fa58bbbddb19" />
</p>

Canny edge detection first produces a binary edge map which feeds into the Hough Transform. `hough()` maps each edge pixel to a sinusoidal curve in `(theta, rho)` parameter space, where `theta` is the line's angle and `rho` is its perpendicular distance from the origin (Lecture 8, Slide 24). The matrix `H` counts how many edge pixels vote for each `(theta, rho)` bin. Bright regions in the Hough Image correspond to parameter values shared by many collinear edge pixels, i.e. a straight line in the original image.

Steps 3 & 4: Finding peaks and 3D visualisation
```matlab
% Step 3: find the 5 tallest peaks in the Hough Image
peaks = houghpeaks(H, 5);
x = theta(peaks(:,2)); y = rho(peaks(:,1));
plot(x, y, 'o', 'color', 'red', 'MarkerSize', 10, 'LineWidth', 1);

% Step 4: 3D surface plot to better visualise peak distribution
figure(3)
surf(theta, rho, H);
xlabel('theta', 'FontSize', 16);
ylabel('rho', 'FontSize', 16)
zlabel('Hough Transform counts', 'FontSize', 16)
```
<p align="center">
<img width="563" height="422" alt="image" src="https://github.com/user-attachments/assets/de74ecd4-1ae4-4391-8298-83d8e5c8e4a5" />
</p>

`houghpeaks` finds the 5 highest bins in `H` and returns their `(theta, rho)` coordinates, overlaid as red circles on the Hough Image. Each peak corresponds to the parameters of one dominant line in the original image. The `surf` plot visualises the same data as a 3D landscape, where peaks appear as sharp spikes, making it intuitive to see where the dominant lines sit in parameter space and how clearly they stand above the background.

Step 5: Fitting lines to the image
```matlab
% Step 5: convert peaks back to line segments and overlay on image
lines = houghlines(fEdge, theta, rho, peaks, 'FillGap', 5, 'MinLength', 7);
figure(4); imshow(f);
hold on
for k = 1:length(lines)
    xy = [lines(k).point1; lines(k).point2];
    plot(xy(:,1), xy(:,2), 'LineWidth', 2, 'Color', 'green');
end
```
<p align="center">
<img width="329" height="271" alt="image" src="https://github.com/user-attachments/assets/d5c45330-0df6-465d-8899-f8053830bcc0" />
</p>

`houghlines` converts the peak parameters back into actual line segment coordinates in the image. `FillGap` merges collinear segments within 5 pixels of each other, and `MinLength` discards segments shorter than 7 pixels. The number of detected segments is typically greater than the number of peaks (5) because a single Hough peak can correspond to multiple disconnected collinear segments, for example, where an object partially occludes a line. To detect different lines such as those orthogonal to the current ones, the number of peaks searched can be increased or the `theta` range can be restricted.

---

### Task 4: Segmentation by Thresholding

Otsu's method is used to segment a yeast cell image (`yeast-cells.tif`), and its limitations with touching cells are explored.

```matlab
clear all
close all

f = imread('yeast-cells.tif');

% Otsu's method: find threshold that maximises between-class variance
T = graythresh(f);
g_otsu = im2bw(f, T);

figure(1)
montage({f, uint8(g_otsu * 255)});
title(sprintf("Original | Otsu's Threshold T = %.3f", T));
```
<p align="center">
<img width="566" height="325" alt="image" src="https://github.com/user-attachments/assets/912a95b0-1c4f-45a8-8552-3a0329d0fa5f" />
</p>

`graythresh` implements Otsu's method, which selects the threshold that maximises the between-class variance of the foreground and background pixel groups (Lecture 9, Slide 8). For images with a bimodal histogram where the two groups form clearly separated peaks, this works well. However, in the yeast cell image, cells that are touching share a boundary with no intensity gap between them, so a single global threshold cannot separate them.

```matlab
% Adaptive local threshold to handle touching cells
T_local = adaptthresh(f, 0.4, 'NeighborhoodSize', 51, 'Statistic', 'mean');
g_local = imbinarize(f, T_local);

figure(2)
montage({f, uint8(g_otsu*255), uint8(g_local*255)});
title("Original | Otsu Global Threshold | Adaptive Local Threshold");
```
<p align="center">
<img width="566" height="566" alt="image" src="https://github.com/user-attachments/assets/db65331c-e6b9-473b-821a-bb11718bca5a" />
</p>


`adaptthresh` computes a spatially varying threshold at each pixel based on its local neighbourhood mean (Lecture 9, Slide 9). By adapting to local intensity statistics rather than using a single global value, it can detect the subtle transitions between touching cells that Otsu's method misses, producing cleaner separation between adjacent cells.

---

### Task 5: Segmentation by k-means Clustering

k-means clustering is used to segment images by colour, treating each pixel's `[R G B]` values as a point in 3D colour space and grouping similar colours together.

```matlab
clear all; close all;

f = imread('baboon.png');
[M, N, S] = size(f);
F = reshape(f, [M*N S]);        % reshape to 1D array of [R G B] pixels
R = F(:,1); G = F(:,2); B = F(:,3);
C = double(F) / 255;            % normalise colours for scatter plot

% 3D scatter plot: each pixel as a coloured dot in RGB colour space
figure(1)
scatter3(R, G, B, 1, C);
xlabel('RED', 'FontSize', 14);
ylabel('GREEN', 'FontSize', 14);
zlabel('BLUE', 'FontSize', 14);
title('Baboon: Pixel RGB Scatter Plot');
```
<p align="center">
<img width="590" height="442" alt="image" src="https://github.com/user-attachments/assets/004ff69e-672b-44c6-bdfc-0c36eaadf5a3" />
</p>


`reshape` converts the 2D image into a 1D list of `[R G B]` triplets, one per pixel. `scatter3` plots each pixel as a dot in 3D colour space, coloured by its actual pixel colour (Lecture 9, Slide 12). Natural clusters visible in this plot correspond to the distinct colour regions in the image.

```matlab
% k-means clustering with k=10
k = 10;
[L, centers] = imsegkmeans(f, k);

% Overlay cluster centroids on scatter plot
hold on
scatter3(centers(:,1), centers(:,2), centers(:,3), 100, 'black', 'fill');

% Reconstruct segmented image using cluster mean colours
J = label2rgb(L, im2double(centers));
figure(2)
montage({f, J});
title(sprintf('Baboon: Original | k-means Segmented (k=%d)', k));
```
<p align="center">
<img width="590" height="305" alt="image" src="https://github.com/user-attachments/assets/3f6576b2-293e-4f70-852a-44dfb7581c24" />
</p>


`imsegkmeans` runs k-means: it places `k` initial centroids in colour space, assigns each pixel to its nearest centroid, recomputes centroids as cluster means, and iterates until convergence (Lecture 9, Slides 15–16). `L` is a label matrix where each pixel holds its cluster index, and `centers` holds the mean `[R G B]` of each cluster. `label2rgb` replaces each label with its corresponding cluster colour to reconstruct the segmented image. The black dots on the scatter plot show where the final centroids sit in colour space.

```matlab
% Compare effect of different k values
figure(3)
for ki = 1:4
    k_val = [3, 5, 10, 20];
    [Li, ci] = imsegkmeans(f, k_val(ki));
    Ji = label2rgb(Li, im2double(ci));
    subplot(2,2,ki);
    imshow(Ji);
    title(sprintf('k = %d', k_val(ki)));
end
sgtitle('k-means Segmentation: Effect of k Value');
```
<p align="center">
<img width="590" height="442" alt="image" src="https://github.com/user-attachments/assets/3a3e43dd-7658-4b47-adec-d9e852c08fce" />
</p>


```matlab
% Repeat on peppers image
f2 = imread('peppers.png');
k2 = 10;
[L2, centers2] = imsegkmeans(f2, k2);
J2 = label2rgb(L2, im2double(centers2));
figure(4)
montage({f2, J2});
title(sprintf('Peppers: Original | k-means Segmented (k=%d)', k2));
```
<p align="center">
<img width="590" height="243" alt="image" src="https://github.com/user-attachments/assets/0c0b878e-d09f-4128-973b-4aa8498cd0fa" />
</p>

A small `k` over-simplifies, merging distinct regions into just a few colours. As `k` increases, finer colour distinctions are preserved and the result more closely resembles the original, but computation time increases. The colourful peppers image is well-suited to colour-based clustering; with k=10 the different coloured peppers are cleanly separated into distinct regions.

---

## Task 6: Watershed Segmentation with Distance Transform

The watershed algorithm combined with a distance transform is used to segment touching cylindrical dowels in `dowels.tif` a case where simple thresholding fails because touching objects merge into one.

```matlab
clear all; close all;

% Read, binarize and clean the dowels image
I = imread('dowels.tif');
f = im2bw(I, graythresh(I));    % Otsu threshold to binarize
g = bwmorph(f, "close", 1);     % close: fill small holes
g = bwmorph(g, "open", 1);      % open: remove noise pixels

figure(1)
montage({I, g});
title('Original & Binarized Cleaned Image');
```
<p align="center">
<img width="590" height="252" alt="image" src="https://github.com/user-attachments/assets/80a022a2-b1d7-4b92-81b2-76bf8770ae33" />
</p>

After binarizing with Otsu's method, the wood grain texture of the dowels creates speckled noise within the binary image. `bwmorph` with `"close"` (dilation then erosion) fills small internal holes, and `"open"` (erosion then dilation) removes isolated noise pixels, leaving a clean binary blob for each dowel.

```matlab
% Distance transform: intensity = distance from background edge
gc = imcomplement(g);           % complement so background is foreground for bwdist
D = bwdist(gc);

figure(2)
imshow(D, [min(D(:)) max(D(:))]);
title('Distance Transform');
```
<p align="center">
<img width="590" height="433" alt="image" src="https://github.com/user-attachments/assets/7a28eb29-8b2b-4cc5-8700-644eae5354e7" />
</p>


`bwdist` measures the distance from each pixel to the nearest foreground (white) pixel. It is applied to `gc`, the complement of `g`, so that the background is white, meaning distances are measured from the background edge inward. The result is an image that peaks in intensity at the centre of each dowel and falls to zero at the boundaries, creating a landscape of hills, one per dowel.

```matlab
% Watershed on complement of D: distance peaks become catchment basins
L = watershed(imcomplement(D));

figure(3)
imshow(L, [0 max(L(:))]);
title('Watershed Segmented Label');
```
<p align="center">
<img width="590" height="433" alt="image" src="https://github.com/user-attachments/assets/c39e77f9-20c6-4445-85ff-f8c553e25ce6" />
</p>

The watershed algorithm is applied to the complement of `D` so that the distance peaks become local minima, watershed floods from local minima, and each resulting catchment basin corresponds to one dowel (Lecture 9, Slides 10–11). The label image `L` appears as a gradient from dark to light because MATLAB assigns integer labels sequentially across the image.

```matlab
% Overlay watershed boundary lines onto binary image
W = (L == 0);                   % ridge lines where L == 0
g2 = g | W;                     % merge boundaries onto binary image

figure(4)
montage({I, g, W, g2}, 'size', [2 2]);
title('Original | Binarized | Watershed Boundaries | Merged Result');
```
<p align="center">
<img width="590" height="439" alt="image" src="https://github.com/user-attachments/assets/99011c2a-58e3-4716-bc69-3a7f5b0784c1" />
</p>

Pixels where `L == 0` are the watershed ridge lines, the dividing boundaries between adjacent basins. `W` is a binary image marking exactly these boundaries. OR-ing `W` with the cleaned binary image `g` overlays the boundaries onto the dowels, producing `g2`: the final result where every dowel is clearly separated from its neighbours by a thin boundary line, even when touching. The four-panel montage shows the full pipeline from raw input to final segmented output.

---

### Challenges

Challenge 2: F14 Fighter Jet Segmentation
The goal is to produce a binary image of `f14.png` where only the F14 fighter jet is shown as white and the rest of the image is black.
Looking at the image, the jet is substantially darker than the surrounding sky background, which is a fairly uniform light grey. This makes intensity-based thresholding a natural starting point. However, several complications arise: the jet has lighter internal regions (cockpit glass, markings) that produce holes after thresholding, and the long nose probe is thin enough to become disconnected from the main body. The pipeline below addresses each of these in turn.

```matlab
clear all; close all;

f = imread('f14.png');

% Convert to grayscale - im2gray handles both RGB and already-grayscale images
f_gray = im2gray(f);

% Step 1: Otsu's threshold to separate jet from sky background
% The jet is darker than the sky so jet pixels fall below the threshold -> invert
T = graythresh(f_gray);
g = ~im2bw(f_gray, T);          % invert: jet = white, sky = black

figure(1)
montage({f_gray, uint8(g*255)});
title(sprintf('Grayscale | Otsu Threshold T = %.3f (inverted)', T));
```
<p align="center">
<img width="590" height="243" alt="image" src="https://github.com/user-attachments/assets/6eecde56-f582-4853-b641-cd5c003a91c6" />
</p>

`graythresh` applies Otsu's method to find the threshold that maximises between-class variance (Lecture 9, Slide 8). `im2gray` is used instead of `rgb2gray` since the image is already greyscale `im2gray` handles both cases without error. The image histogram is roughly bimodal, with a large bright peak from the sky and a smaller darker cluster from the jet, making Otsu a good fit here. Because the jet is the darker class, it falls below the threshold, so the result is inverted with `~` to make the jet white and the sky black.

```matlab
% Step 2: Fill holes inside the jet body
% Thresholding leaves hollow regions where cockpit glass and markings are lighter
g = imfill(g, 'holes');
```

The cockpit canopy and various light-coloured markings on the fuselage are closer in intensity to the sky than to the darker jet body, so they remain black after thresholding, punching holes in the jet mask. `imfill` with the `'holes'` option floods any enclosed black regions surrounded by white, filling these internal cavities so the entire jet body becomes solid white.

```matlab
% Step 3: Close with a large disk to bridge disconnected jet parts
% The nose probe is thin and disconnected from the main body after thresholding
se = strel('disk', 15);
g = imclose(g, se);
```

The long nose probe of the F14 is very thin and slightly lighter in intensity, causing it to become disconnected from the main fuselage after thresholding. `imclose` (dilation followed by erosion with a disk of radius 15) expands all white regions temporarily, bridging any narrow gaps, then contracts them back, reconnecting the probe to the body without permanently thickening the overall shape.

```matlab
% Step 4: Keep only the largest connected component (the jet itself)
% Removes any remaining small noise blobs from the ground/cloud region below
cc = bwconncomp(g);
stats = regionprops(cc, 'Area');
[~, idx] = max([stats.Area]);   % index of largest component
g_jet = false(size(g));
g_jet(cc.PixelIdxList{idx}) = true;  % retain only the jet blob

figure(2)
montage({f, uint8(g_jet*255)});
title('Original | Final Binary Jet Mask');
```
<p align="center">
<img width="590" height="243" alt="image" src="https://github.com/user-attachments/assets/145b9cf9-6f77-4249-ae2b-f2cc616505f1" />
</p>

Despite the earlier cleaning steps, small residual blobs may remain from the ground terrain visible in the lower portion of the image. `bwconncomp` finds all connected components in the binary image, `regionprops` measures the area of each, and only the largest, the jet, is retained in the final mask `g_jet`. The result is a clean binary silhouette of the F14 with all background pixels set to black.

---
