# Lab 3 - Intensity Transformation and Spatial Filtering

This lab focuses on intensity transformation and spatial filtering techniques in MATLAB. We use enhancement methods such as adjusting contrast, performing negative and gamma corrections, and applying histogram-based transformations to improve or modify image intensities. The lab uses built-in MATLAB functions like imadjust and histogram tools to demonstrate how changes to pixel intensity distributions affect visual image quality. 

---

## Contents
- Task 1: Contrast enhancement with 'imadjust'
- Task 2: Contrast-stretching transformation
- Task 3: Contrast enhancement using histograms
- Task 4: Noise reduction with a low-pass filter
- Task 5: Median filtering
- Task 6: Sharpening the image with Laplacian, Sobel and Unsharp filters
- Task 7: Test yourself Challenges

---

### Task 1: Contrast enhancement with 'imadjust'

This task applies different intensity transformations to an image to demonstrate how pixel values can be manipulated. The first transformation inverts the image by reversing the intensity scale. The second enhances contrast by stretching a selected mid-range of intensities while suppressing others. The third applies gamma correction to darken the image, particularly affecting mid-tone values.

```matlab
clear all
imfinfo('breastXray.tif');
f = imread('breastXray.tif');
imshow(f)
```

For the first task, the image we'll be processing is of a breast X-ray. To begin, the code above imports the image and reads the 

<img width="575" height="572" alt="image" src="https://github.com/user-attachments/assets/5b2a2a84-a94e-47e1-8972-897accebc1ef" />

```matlab
f(3,10);              % print the intensity of pixel(3,10)
imshow(f(:,241:482))  % display only top half of the image
[fmin, fmax] = bounds(f(:));
```

This image gets stored as a matrix in MATLAB, and 'f(3,10)' returns the pixel value of what is stored in row 3 & column 10 (28). The next function shows the image between the bounds of 241:482. The image in total spans 571:482, so using the imshow function, we can see only the right side of our image. 'fmin' and 'fmax' give us the maximum and minimum values of the pixel in our image.

<img width="461" height="696" alt="image" src="https://github.com/user-attachments/assets/3f21d9cb-7b39-4028-8c41-118d2d0bf489" />

```matlab
g1 = imadjust(f, [0 1], [1 0]);
figure              % open a new figure window
montage({f, g1})
```
The 'imadjust' function helps change the brightness and contrast of the image. [0 1] tells MATLAB to use the full intensity range of the image, while [1 0] tells MATLAB to flip the intensity. Finally, montage gives us the original image and the version of the image with the intensity flipped side-by-side. 

<img width="575" height="338" alt="image" src="https://github.com/user-attachments/assets/e0520a20-20f6-4a7b-83b2-fa610ea6a670" />

```matlab
g2 = imadjust(f, [0.5 0.75], [0 1]);
g3 = imadjust(f, [ ], [ ], 2);
figure
montage({g2,g3})
```

Similar to the previous section, 'g2' tells MATLAB to only use pixels with intensities between 0.5 and 0.75, and this range is then stretched to [0 1]. The pixels below 0.5 appear black while the ones above 0.75 appear white.

In 'g3', the empty brackets tell MATLAB to use the default intensity settings, but the 2 at the end, which is the gamma value, helps make the image darker (especially the mid-tones), thus making the bright areas look relatively bright.

<img width="575" height="338" alt="image" src="https://github.com/user-attachments/assets/0b9df22a-8f09-446b-b2ba-207eaf0aa107" />

---

### Task 2: Contrast-stretching transformation

This code enhances the contrast of a medical image by using the average brightness of the image to adjust pixel intensities, then shows the original and enhanced images next to each other.

```matlab
clear all       % clear all variables
close all       % close all figure windows
f = imread('bonescan-front.tif');
r = double(f);  % uint8 to double conversion
k = mean2(r);   % find mean intensity of image
E = 0.9;
s = 1 ./ (1.0 + (k ./ (r + eps)) .^ E);
g = uint8(255*s);
imshowpair(f, g, "montage")
```

Setting the variable for imread saves a uint8 image into 'f', which we then convert into a double because mathematical operations work better with doubles. 'k' is the average value of the intensities in the image, while 'E' sets a constant value for how strong the intensity transformation is. 

The equation 's' maps pixel values to a new range, which enhances details in darker regions of the image. 'g' then scales the transformed image back to the range 0-255 and converts it to uint8 so it can be displayed properly.

<img width="575" height="744" alt="image" src="https://github.com/user-attachments/assets/e63b5f7e-97d3-4268-afe4-285c08a79a36" />

---

### Task 3: Contrast enhancement using histograms

This code loads a grayscale image, improves its contrast using intensity adjustment and histogram equalisation, and analyses how pixel intensities change by plotting histograms, PDFs, and CDFs. The original, adjusted, and equalised images are then compared visually and statistically.

```matlab
clear all       % clear all variable in workspace
close all       % close all figure windows
f=imread('pollen.tif');
imshow(f)
figure          % open a new figure window
imhist(f)      % calculate and plot the histogram
```

The code above loads a uint8 image of grains of pollen into a variable 'f', after which we calculate and plot the histograms of the image using the imhist function.


<img width="575" height="501" alt="image" src="https://github.com/user-attachments/assets/e1d8e293-add9-4196-89cc-462d83779490" />
<img width="575" height="432" alt="image" src="https://github.com/user-attachments/assets/5fab7df8-3359-4e38-bf46-b13590023226" />

```matlab
close all
g=imadjust(f,[0.3 0.55]);
montage({f, g})     % display list of images side-by-side
figure
imhist(g)
```

The imadjust function in variable 'g' adjusts the image contrast by stretching pixel values between 0.3 and 0.55 to the full range 0 to 1, after which imhist displays the histogram of the contrast-adjusted image.

```matlab
g_pdf = imhist(g) ./ numel(g);  % compute PDF
g_cdf = cumsum(g_pdf);          % compute CDF
close all                       % close all figure windows
imshow(g);
subplot(1,2,1)                  % plot 1 in a 1x2 subplot
plot(g_pdf)
subplot(1,2,2)                  % plot 2 in a 1x2 subplot
plot(g_cdf)
```

'g_pdf' computes the probability density function (PDF) of the image by normalising the histogram, while 'g_cdf' computes the cumulative distribution function (CDF) from the PDF. We can then view the PDF and CDF side by side. 'g' over here also shows us the contrast-adjusted image.


<img width="575" height="501" alt="image" src="https://github.com/user-attachments/assets/8b3644a7-4d87-4b91-8b1b-478d4224db16" />
<img width="575" height="501" alt="image" src="https://github.com/user-attachments/assets/fc70f13b-9f2a-44a1-988f-697e0cf6db39" />

```matlab
x = linspace(0, 1, 256);    % x has 256 values equally spaced
                            %  .... between 0 and 1
figure
plot(x, g_cdf)
axis([0 1 0 1])             % graph x and y range is 0 to 1
set(gca, 'xtick', 0:0.2:1)  % x tick marks are in steps of 0.2
set(gca, 'ytick', 0:0.2:1)
xlabel('Input intensity values', 'fontsize', 9)
ylabel('Output intensity values', 'fontsize', 9)
title('Transformation function', 'fontsize', 12)

h = histeq(g,256);              % histogram equalize g
close all
montage({f, g, h})
figure;
subplot(1,3,1); imhist(f);
subplot(1,3,2); imhist(g);
subplot(1,3,3); imhist(h);
```

This code plots the cumulative distribution function (CDF) of the contrast-adjusted image to show how input pixel intensities are mapped to output values. The graph is formatted with fixed axis limits, tick marks, and labels so the transformation is easy to understand.

The code then applies histogram equalisation to further enhance the image contrast. The original, adjusted, and equalised images are displayed side by side, along with their histograms, allowing a clear comparison of how each step changes the intensity distribution.

---

### Task 4 + 5: Noise reduction with a low-pass filter & Median filtering

This code compares different noise-reduction techniques by applying box, Gaussian, and median filters to a noisy image. The results are displayed side by side to show how each filter smooths the image and handles noise differently.

```matlab
clear all
close all
f = imread('noisyPCB.jpg');
imshow(f)

w_box = fspecial('average', [9 9]);
w_gauss = fspecial('Gaussian', [7 7], 1.0);

```

Once we load the noisy PCB image into 'f', 'w_box' creates a 9×9 box (average) filter that smooths the image by averaging neighbouring pixels, while 'w_gauss' creates a 7×7 Gaussian filter with a standard deviation of 1.0, which smooths the image while preserving edges better than a box filter.

```matlab

g_box = imfilter(f, w_box, 0);
g_gauss = imfilter(f, w_gauss, 0);
figure
montage({f, g_box, g_gauss})
```

'g_box' applies the box filter to the image to reduce noise by averaging pixel values, and 'g_gauss' applies the Gaussian filter to reduce noise in a smoother, more natural way.

<img width="575" height="497" alt="image" src="https://github.com/user-attachments/assets/ed1386da-c6a2-40da-a4ea-8c5ba378e7ad" />
<img width="575" height="504" alt="image" src="https://github.com/user-attachments/assets/0cca35d6-c6a6-4e41-91ef-26b01f45f0c3" />


```matlab

g_median = medfilt2(f, [7 7], 'zero');
figure; montage({f, g_median})
```
Finally, 'g_median' applies a 7×7 median filter, which replaces each pixel with the median of its neighbourhood and is especially effective at removing salt-and-pepper noise.

<img width="575" height="290" alt="image" src="https://github.com/user-attachments/assets/1ba514cd-8624-4e02-a1ab-826f8e138810" />

---

# Task 6: Sharpening the image with Laplacian, Sobel and Unsharp filters

In this task, we explore various filter kernels to sharpen the moon image stored in the file moon.tif. The goal was to make the moon photo sharper so that the craters can be observed better.

```matlab
clear all
close all

% Read original image
f = imread('moon.tif');
f = double(f);

% Create sharpening filters
w_lap = fspecial('laplacian', 0.5);
w_sobel = fspecial('sobel');
w_unsharp = fspecial('unsharp', 0.2);

% Apply filters
g_lap = imfilter(f, w_lap, 'replicate');
g_sobel = imfilter(f, w_sobel, 'replicate');
g_unsharp = imfilter(f, w_unsharp, 'replicate');

% Laplacian sharpening
alpha = 1.0;
sharp_lap = f - alpha * g_lap;

% Sobel sharpening
beta = 0.7;
sharp_sobel = f + beta * abs(g_sobel);

% Combined Laplacian + Sobel sharpening
sharp_combined = f - 0.8 * g_lap + 0.6 * abs(g_sobel);

% Display results
figure
montage({
    uint8(f),uint8(sharp_combined)})
title('Original vs. Combined ')
```

The image is sharpened by extracting edge information using the Laplacian and Sobel filters and adding this information back to the original image. Laplacian sharpening enhances fine details, while Sobel sharpening strengthens edges. Combining both produces a clearer image where lunar craters are more visible. Unsharp masking is also used as a reference method for comparison.

---

### Task 7: Challenges

For this task, we have 3 challenges to complete:
- Challenge 1: Improve the contrast of a lake and tree image stored in the file lake&tree.png
- Challenge 2: Use the Sobel filter in combination with any other techniques, find the edge of the circles in the image file circles.tif.
- Challenge 3: Improve the lighting and colour of office.jpg, which is a colour photograph taken of an office with bad exposure.

#### Challenge 1: Contrast improvement

For the first challenge, I will use two of the methods from this lab session and compare how they differ when improving the contrast of the 'lake&tree' image.

```matlab
imfinfo('lake&tree.png');
f = imread('lake&tree.png');
g1 = imadjust(f, [0 0.6], [0 1]);
h = histeq(f,256);              % histogram equalize g

montage({g1, h})
```
The first method I applied was using the imadjust function to manually adjust the range of pixels used based on their intensities (from 0 - 0.6), and the second method was the histogram equalisation from the end of task 3. 

<img width="571" height="396" alt="image" src="https://github.com/user-attachments/assets/1bd2bc21-ad5b-4d1d-a31a-bdbc1e6d07fc" />

On comparing the two images, it is clear that the second method (histogram equalisation) produces an image with better contrast. This is because while imadjust only performs a linear mapping by shifting the intensity range, it fails when data is heavily clumped in the mid-tones. histeq is better here because it uses a non-linear transformation to get a uniform distribution of the histogram.

#### Challenge 2: Edge detection using Sobel & other filters

```matlab
clear all
close all

% Read image
f = imread('circles.tif');

% Convert to double for processing
f = double(f);

% Create Sobel filters
w_sobel_x = fspecial('sobel');        % horizontal edges
w_sobel_y = w_sobel_x';               % vertical edges

% Apply Sobel filters
gx = imfilter(f, w_sobel_x, 'replicate');
gy = imfilter(f, w_sobel_y, 'replicate');

% Combine gradients (edge strength)
g_mag = sqrt(gx.^2 + gy.^2);

% Normalise for display
g_mag = g_mag / max(g_mag(:));

% Simple threshold to get edges
T = 0.3;                  % threshold value
edges = g_mag > T;
montage({uint8(f), uint8(255*edges)})
```

The code defines Sobel kernels 'w_sobel_x' for horizontal intensity changes and its transpose 'w_sobel_y' for vertical changes which are applied via imfilter with 'replicate' padding to handle boundary conditions. These directional gradients, gx and gy, are combined to calculate the gradient magnitude, which is then normalised to a [0 1] range. Finally, a global threshold (T = 0.3) is applied to create a binary mask of the strongest edges

<img width="596" height="246" alt="image" src="https://github.com/user-attachments/assets/5b81fe3d-1029-4000-918c-b23ba32e19e1" />

#### Challenge 3: Exposure improvement

For this challenge, in order to increase the exposure of the image, we can use imadjust similar to how we have before. 

```matlab
imfinfo('office.jpg');
f = imread('office.jpg');
g1 = imadjust(f, [0 0.6], [0 1]);

montage({uint8(f), g1})
```

The result of the following code is a much brighter image, as expected.
<img width="596" height="386" alt="image" src="https://github.com/user-attachments/assets/f176397c-7c19-4768-95e8-3782f1a9f65d" />
