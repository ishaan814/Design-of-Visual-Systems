# Lab 1 - Image Rotation & Shearing

This lab implements image rotation and shearing in MATLAB without using pre-existing image transformation functions. All transformations are performed using explicit mathematical operations and inverse mapping.

---

## Contents
- Task 1: `rotate.m` - Image rotation using inverse mapping
- Task 2: `shear.m` - Image shearing transformation
- Final Script: `run_lab1.m` - Script to demonstrate and test both transformations

---

### Task 1: Rotate Function (`rotate.m`)

The `rotate` function rotates an image by a given angle about its centre.  
It performs the rotation using mathematical calculations instead of MATLAB’s built-in image rotation functions.

```matlab
function Out = rotate(In, Theta)
% ROTATE Rotates an image by Theta degrees about its centre
% without using built-in rotation functions.

% Convert degrees to radians
Theta = deg2rad(Theta);

% Get image size
rows = size(In,1);
cols = size(In,2);

% Preallocate output image
Out = zeros(size(In), 'like', In);

% Image centre
cp = [round(rows/2); round(cols/2)];

% Rotation matrix
tm = [ cos(Theta)  -sin(Theta);
       sin(Theta)   cos(Theta) ];

% Inverse rotation matrix
rtm = inv(tm);
```
This section prepares everything needed for rotation. The angle is converted to radians, the image size and centre are calculated, and an empty output image is created. The rotation matrix and its inverse define how pixel coordinates will be rotated mathematically.

```matlab
% Loop through output image pixels
for y = 1:cols
    for x = 1:rows
        
        % Destination pixel
        p = [x; y];
        
        % Map back to source image
        tp = round(rtm * (p - cp) + cp);
```
Here, the code loops through every pixel in the output image. For each output pixel, inverse rotation is applied to find the corresponding location in the original image. Using inverse mapping ensures every output pixel gets a value without leaving holes.

```matlab
        % Boundary check
        if tp(1) < 1 || tp(2) < 1 || tp(1) > rows || tp(2) > cols
            Out(x,y) = 0;   % Black pixel
        else
            Out(x,y) = In(tp(1), tp(2));
        end
    end
end
end

```
<img width="361" height="218" alt="image" src="https://github.com/user-attachments/assets/385c7b93-09aa-4421-98a0-1d1ac8df8a03" />


This final part checks whether the mapped source pixel lies inside the image boundaries. If it falls outside, the output pixel is set to black; otherwise, the corresponding pixel value is copied from the input image. This prevents indexing errors and defines the background of the rotated image.

---

### Task 2: Shear Function (`shear.m`)

The `shear` function applies a shear transformation to an image about its centre.  
This changes the shape of the image by shifting pixels horizontally and/or vertically using a shear matrix.

```matlab
function Out = shear(In, xshear, yshear)
% SHEAR Applies a shear transformation to an image about its centre
% without using built-in image transformation functions.

% Get image size
rows = size(In,1);
cols = size(In,2);

% Preallocate output image
Out = zeros(size(In), 'like', In);

% Image centre
cp = [round(rows/2); round(cols/2)];

% Shear transformation matrix
tm = [ 1       xshear;
       yshear  1      ];

% Inverse transformation matrix
rtm = inv(tm);
```
This section sets up the image dimensions, output image, and the image centre used as the reference point for shearing. The shear matrix defines how pixels are shifted horizontally and vertically, and its inverse is calculated for inverse mapping. Using the image centre ensures the shear happens about the centre rather than a corner.


```matlab
% Loop through output image pixels
for y = 1:cols
    for x = 1:rows
        
        % Destination pixel
        p = [x; y];
        
        % Map back to source image (inverse mapping)
        tp = round(rtm * (p - cp) + cp);
```
The nested loops go through every pixel in the output image. For each output pixel, inverse shearing is applied to find where that pixel originated in the input image. Inverse mapping ensures all output pixels are filled without gaps.


```matlab
        % Boundary check
        if tp(1) < 1 || tp(2) < 1 || tp(1) > rows || tp(2) > cols
            Out(x,y) = 0;   % Set outside pixels to black
        else
            Out(x,y) = In(tp(1), tp(2));
        end
    end
end
end

```
<img width="361" height="218" alt="image" src="https://github.com/user-attachments/assets/cb111b7d-a316-4cb7-82d2-4978ab03d556" />


This final section checks whether the mapped source pixel lies within the input image boundaries. Pixels that map outside the image are set to black, while valid pixels copy their intensity from the input image. This prevents indexing errors and defines the background of the sheared image.


---

### Final Demonstration Script (`Lab_1.m`)

The `Lab_1.m` script is used to test the rotation and shear functions.  
It loads an image, applies both transformations, and displays the original and transformed images for comparison.

```matlab
img = imread('clown.jpg');

if size(img,3) == 3
    img = rgb2gray(img);
end

img = double(img);
theta = -30;
rotated_img = rotate(img, theta);

xshear = 0.1;
yshear = 0.5;
sheared_img = shear(img, xshear, yshear);

imshow(uint8(img));
title('Original Image');

imshow(uint8(rotated_img));
title('Rotated Image (45°)');

imshow(uint8(sheared_img));
title('Sheared Image');

```
---
