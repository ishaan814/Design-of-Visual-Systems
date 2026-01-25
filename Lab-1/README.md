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
The `rotate` function rotates a grayscale image by a specified angle about its centre without using MATLAB’s built-in `imrotate` function.

### Full Implementation
```matlab
function Out = rotate(In, Theta)

Theta = deg2rad(Theta);

rows = size(In,1);
cols = size(In,2);

Out = zeros(size(In), 'like', In);

cp = [round(rows/2); round(cols/2)];

tm = [ cos(Theta) -sin(Theta);
       sin(Theta)  cos(Theta) ];

rtm = inv(tm);

for y = 1:cols
    for x = 1:rows
        p = [x; y];
        tp = round(rtm * (p - cp) + cp);

        if tp(1) < 1 || tp(2) < 1 || tp(1) > rows || tp(2) > cols
            Out(x,y) = 0;
        else
            Out(x,y) = In(tp(1), tp(2));
        end
    end
end
end

