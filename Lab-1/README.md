# Lab 1 - Image Rotation & Shearing

This lab implements image rotation and shearing in MATLAB without using pre-existing image transformation functions. All transformations are performed using explicit mathematical operations and inverse mapping.

---

## Contents
- `rotate.m` - Image rotation using inverse mapping
- `shear.m` - Image shearing transformation
- `run_lab1.m` - Script to demonstrate and test both transformations

---

## Rotate Function (`rotate.m`)

### Purpose
The `rotate` function rotates an image by a given angle about its centre.  
It performs the rotation using mathematical calculations instead of MATLAB’s built-in image rotation functions.

### Full Implementation
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

% Loop through output image pixels
for y = 1:cols
    for x = 1:rows
        
        % Destination pixel
        p = [x; y];
        
        % Map back to source image
        tp = round(rtm * (p - cp) + cp);
        
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
---

## Shear Function (`shear.m`)

### Purpose
The `shear` function applies a shear transformation to an image about its centre.  
This changes the shape of the image by shifting pixels horizontally and/or vertically using a shear matrix.

### Full Implementation
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

% Loop through output image pixels
for y = 1:cols
    for x = 1:rows
        
        % Destination pixel
        p = [x; y];
        
        % Map back to source image (inverse mapping)
        tp = round(rtm * (p - cp) + cp);
        
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

---

## Demonstration Script (`Lab_1.m`)

### Purpose
The `Lab_1.m` script is used to test the rotation and shear functions.  
It loads an image, applies both transformations, and displays the original and transformed images for comparison.

### Script Implementation
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

figure;

subplot(1,3,1);
imshow(uint8(img));
title('Original Image');

subplot(1,3,2);
imshow(uint8(rotated_img));
title('Rotated Image (45°)');

subplot(1,3,3);
imshow(uint8(sheared_img));
title('Sheared Image');

```
