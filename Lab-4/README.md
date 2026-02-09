# Lab 4 - Morphological Image Processing

In this lab, we will explore the use of various morphological operators and get a feel of how they modify visual information.

---

## Contents
- Task 1: Dilation & Erosion
- Task 2: Morphological Filtering with Open & Close
- Task 3: Boundary Detection
- Task 4: Function bwmorph - Thinning & Thickening
- Task 5: Connected Components & Labels
- Task 6: Morphological Reconstruction
- Task 7: Morphological Operations on Grayscale Images
- Task 8: Challenges

---

### Task 1: Dilation & Erosion

For the first part of this task, we'll use the dilation function on an image with its corresponding structural element.

```matlab
A = imread('text-broken.tif');
B1 = [0 1 0;
     1 1 1;
     0 1 0];    % create structuring element
A1 = imdilate(A, B1);
montage({A,A1})

B2 = ones(3,3);     % generate a 3x3 matrix of 1's

Bx = [1 0 1;
      0 1 0;
      1 0 1];

SE = strel('disk',4);
SE.Neighborhood         % print the SE neighborhood contents
```
<img width="571" height="266" alt="image" src="https://github.com/user-attachments/assets/48cc0440-21dc-4820-ae9d-d822f6ca4e2b" />

This code reads a grayscale image and applies dilation using a small cross-shaped structuring element to make the bright regions thicker. The original and processed images are displayed side by side to show the effect of dilation. It also defines other example structuring elements and displays the pixel layout of a disk-shaped structuring element.

The second morphological operation we will be testing is erosion.

```matlab
clear all
close all
A = imread('wirebond-mask.tif');
SE2 = strel('disk',2);
SE10 = strel('disk',10);
SE20 = strel('disk',20);
E2 = imerode(A,SE2);
E10 = imerode(A,SE10);
E20 = imerode(A,SE20);
montage({A, E2, E10, E20}, "size", [2 2])
```
<img width="571" height="515" alt="image" src="https://github.com/user-attachments/assets/2c308dfe-2741-4ad7-83dc-f522e1eb3752" />


This code loads an image and applies erosion using disk-shaped structuring elements of increasing size. As the disk size increases, more of the bright areas in the image are removed. The original image and the eroded results are displayed together to compare the effect of different erosion sizes.


---

### Task 2: Morphological Filtering with Open & Close

In this task, we explore the effect of using Open and Close on a binary noisy fingerprint image.

```matlab
f = imread('fingerprint-noisy.tif');
SE = ones(3,3);

% Erosion
fe = imerode(f, SE);

% Dilation of eroded image
fed = imdilate(fe, SE);

% Opening (erosion followed by dilation)
fo = imopen(f, SE);

% Display results
montage({f, fe, fed, fo}, 'Size', [2 2])
```
<img width="571" height="411" alt="image" src="https://github.com/user-attachments/assets/c1fb69fb-4ade-4480-b1ed-1d836e9b3d2f" />

This code reads a noisy fingerprint image and applies erosion and dilation using a 3×3 structuring element. It then performs opening and displays the original image alongside the processed results to compare their effects.

```matlab
% Try a different structuring element (disk)
SE_disk = strel('disk', 2);
fo_disk = imopen(f, SE_disk);

% Improve image using closing
foc = imclose(fo_disk, SE_disk);

figure
montage({f, fo_disk, foc}, 'Size', [1 3])

% Gaussian filtering for comparison
f_gauss = imgaussfilt(uint8(f), 2);

figure
montage({f, foc, f_gauss}, 'Size', [1 3])
```
<img width="571" height="170" alt="image" src="https://github.com/user-attachments/assets/a2b307da-d109-4fcf-ad60-8ee9690d6905" />
<img width="571" height="170" alt="image" src="https://github.com/user-attachments/assets/fa3b92b3-b4de-4554-8c0d-8719963542db" />


This code applies opening and closing using a disk-shaped structuring element to further reduce noise while preserving the fingerprint pattern. It then compares the result of morphological filtering with Gaussian smoothing to show the difference between structure-preserving and blur-based noise removal.

---

### Task 3: Boundary Detection

```matlab
clear all
close all

% Read and invert the grayscale image
I = imread('assets/blobs.tif');
I = imcomplement(I);

% Convert to binary using Otsu's method
level = graythresh(I);
BW = imbinarize(I, level);

% Create 3x3 structuring element
SE = ones(3,3);

% Erode the binary image
BW_eroded = imerode(BW, SE);

% Boundary detection (original minus eroded)
BW_boundary = BW - BW_eroded;

% Display results
montage({I, BW, BW_eroded, BW_boundary}, 'Size', [2 2])
```
<img width="571" height="515" alt="image" src="https://github.com/user-attachments/assets/a8a22d08-49c8-4ff3-83ab-8cd582070f89" />

The boundary operation successfully highlights the edges of the blobs by subtracting the eroded image from the original binary image. Some noise and broken boundaries are still visible due to thresholding and background noise. The result can be improved by applying morphological opening or closing before boundary extraction to reduce noise and smooth blob shapes.

---

### Task 4: Function bwmorph - Thinning and Thickening

In this task, we will be using Matlab's morphological function bwmorph, which implements a variety of morphological operations based on combinations of dilations and erosions.

```matlab
clear all
close all

% Read fingerprint image
f = imread('fingerprint.tif');

% Convert to binary using Otsu's method
level = graythresh(f);
BW = imbinarize(f, level);

% Make sure fingerprint is white on black
BW = imcomplement(BW);

% Thinning 1 to 5 iterations
g1 = bwmorph(BW, 'thin', 1);
g2 = bwmorph(BW, 'thin', 2);
g3 = bwmorph(BW, 'thin', 3);
g4 = bwmorph(BW, 'thin', 4);
g5 = bwmorph(BW, 'thin', 5);

% Thinning until convergence
g_inf = bwmorph(BW, 'thin', inf);

% Display results
montage({BW, g1, g2, g3, g4, g5, g_inf}, 'Size', [2 4])

% Display black lines on white background
BW_black = imcomplement(BW);
g_inf_black = bwmorph(BW_black, 'thin', inf);

figure
montage({BW_black, g_inf_black}, 'Size', [1 2])
```
<img width="571" height="342" alt="image" src="https://github.com/user-attachments/assets/d19a7e36-0eba-4880-8aa9-a867fc6a40c2" />
<img width="571" height="340" alt="image" src="https://github.com/user-attachments/assets/ae2e10dd-8fc4-4950-ad6e-543322f63b99" />


Repeated thinning gradually reduces the fingerprint ridges to single-pixel-wide skeletons. When thinning is applied with n = inf, the image converges to its skeleton and no longer changes. Thinning and thickening are complementary operations, depending on whether the foreground is defined as white or black.

---

### Task 5: Connected Components & Labels

The goal of this task is to find the largest connected component in the image and then erase it.

<img width="521" height="419" alt="image" src="https://github.com/user-attachments/assets/8380493d-0cdb-4f62-ae4e-3fee532f5fdd" />

```matlab
t = imread('text.png');
imshow(t)
CC = bwconncomp(t)

numPixels = cellfun(@numel, CC.PixelIdxList);
[biggest, idx] = max(numPixels);
t(CC.PixelIdxList{idx}) = 0;
figure
imshow(t)
```
<img width="521" height="419" alt="image" src="https://github.com/user-attachments/assets/86263201-c655-4c6e-a60b-bdd37560e193" />

This code finds all connected components in a binary text image and measures their sizes. It identifies the largest connected component and removes it by setting its pixels to zero. The result is displayed to show the image with the largest object removed.

---

### Task 6: Morphological Reconstruction

In morphological opening, erosion removes small objects, and subsequent dilation tends to restore the shape of the objects that remain. However, the accuracy of this restoration relies on the similarity between the shapes to be restored and the structuring element.

Morphological reconstruction (MR) is a better method that restores the original shapes of the objects that remain after erosion, which is what we will be trying in this task.

```matlab
clear all
close all
f = imread('text_bw.tif');
se = ones(17,1);
g = imerode(f, se);
fo = imopen(f, se);     % perform open to compare
fr = imreconstruct(g, f);
montage({f, g, fo, fr}, "size", [2 2])
```
<img width="571" height="569" alt="image" src="https://github.com/user-attachments/assets/e20f3f91-bb16-4869-afd8-a660e021a86c" />

This code reads a black-and-white image and applies basic morphological operations. It first erodes the image (imerode) using a vertical line structuring element, then performs an opening (imopen) for comparison, and finally reconstructs the image from the eroded version (imreconstruct). All four images—the original, eroded, opened, and reconstructed—are displayed together in a 2×2 grid using montage.

The original image shows all text intact. The eroded image thins the text and breaks small features, while the opened image removes some noise but also shrinks thinner parts of the text. The reconstructed image restores much of the original text, demonstrating that morphological reconstruction can recover structures that were partially removed during erosion.

```matlab
ff = imfill(f);
figure
montage({f, ff})
```
<img width="571" height="327" alt="image" src="https://github.com/user-attachments/assets/6b903499-3cc6-46fb-8921-433762a3950f" />

The function imfill fills the hole in the original image, allowing us to focus on the text.

--- 

### Task 7: Morphological Operations on Grayscale images

So far, we have only been using binary images because they vividly show the effect of morphological operations, turning black pixels to white pixels instead of just changing the shades of grey.

In this task, we will explore the effect of erosion and dilation on grayscale images.

```matlab
clear all; 
close all;
f = imread('headCT.tif');
se = strel('square',3);
gd = imdilate(f, se);
ge = imerode(f, se);
gg = gd - ge;
montage({f, gd, ge, gg}, 'size', [2 2])
```
<img width="580" height="544" alt="image" src="https://github.com/user-attachments/assets/6f5b4bc1-9127-4531-9922-1b04d7cced9e" />

The original CT image shows the natural grayscale of the head. Dilation brightens and expands the lighter regions, making edges more pronounced, while erosion darkens and shrinks them, reducing small bright details. The difference image highlights edges clearly, showing how morphological operations can reveal structure boundaries in grayscale images.

---

### Task 8: Challenges

The challenge I've chosen to do for this lab is:
The file 'assets/normal-blood.png' is a microscope image of red blood cells. Using various techniques you have learned, write a Matlab .m script to count the number of red blood cells.

In this code, we use `bwareaopen`, `imfill`, `imerode`, and `imreconstruct`, which are all morphological operations covered in Lab 4 to clean noise, fill holes, and separate connected objects. We also use `bwconncomp` and `bwboundaries` to identify and count individual red blood cells, just like we did when analysing connected components in Lab 4.

```matlab
I = imread('normal-blood.png');

% Convert to grayscale if image is RGB
if size(I,3) == 3
    I = rgb2gray(I);
end

% Normalize to [0,1] and improve contrast
I = mat2gray(I);
I = imadjust(I);

% Binarize
level = graythresh(I);
BW = imbinarize(I, level);

% Invert to accurately count cells without background hinderance
BW = ~BW;
```
This section reads the image, converts it to grayscale if necessary, and normalizes the intensity values to a standard 0–1 range. We then enhance contrast with imadjust to make the cells stand out, binarize the image using Otsu’s method (graythresh + imbinarize), and finally invert it so the cells are foreground (white) and the background is black, which is easier for morphological operations.

```matlab
% Remove small noise blobs
BW_clean = bwareaopen(BW, 50);

% Fill small holes in cells to avoid inner-counting
BW_clean = imfill(BW_clean, 'holes');

% Separate touching cells using erosion then reconstruction
se = strel('disk',3);
BW_eroded = imerode(BW_clean, se);
BW_recon = imreconstruct(BW_eroded, BW_clean);
```
Here, we use morphological operations from Lab 4 to prepare the binary image for counting. bwareaopen removes small noise pixels, imfill fills holes inside cells, and the combination of imerode + imreconstruct separates touching or overlapping cells while preserving their shape. This ensures each RBC becomes an individual connected component.

```matlab
% Count cells using connected components
CC = bwconncomp(BW_recon);
numCells = CC.NumObjects;

figure;
imshow(I); title('Red Blood Cells');
hold on;

% Draw boundaries
B = bwboundaries(BW_recon);
for k = 1:length(B)
    boundary = B{k};
    plot(boundary(:,2), boundary(:,1), 'r', 'LineWidth', 1);
end

% Output number of cells
disp('Number of red blood cells:');
disp(numCells);
```

This part identifies individual cells using bwconncomp, which finds connected foreground regions, and counts them. bwboundaries is used to extract the boundaries of each cell for visualisation. Finally, the code displays the original image with red outlines over the RBCs and prints the total number of red blood cells.

The final outlined image, which is used to count the cells, is as follows:
<img width="734" height="582" alt="image" src="https://github.com/user-attachments/assets/30d22fdd-c1b0-4105-aea3-50dec64faefa" />

---


