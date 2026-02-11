# Lab 2 - Colour Perception

The first part of this lab shows us various physiological and psychological phenomena related to our vision, colour perception, and how the brain makes up missing visual information, along with why they occur. In the second part of the lab we see how images are separated into different colour spaces.

---

## Contents
Part 1 - Seeing Colours and Shapes
- Task 1: Finding your blindspot
- Task 2: Ishihara colour test
- Task 3: Reverse colour
- Task 4: Troxler's fading
- Task 5: Brain interpretation according to expectation
- Task 6: Grid illusion
- Task 7: Cafe wall illusion
- Task 8: The silhouette illusion
- Task 9: The incomplete triangles


Part 2 - Exploring Colours in MATLAB
- Task 10: Converting RGB images to greyscale
- Task 11: Splitting an RGB image into separate channels
- Task 12: Map RGB image to HSV space and into separate channels
- Task 13: Map RGB image into XYZ space

---

### Task 1: Finding your blindspot
Watch this video to find your blind spot: http://www.ee.ic.ac.uk/pcheung/teaching/DE4_DVS/assets/blind_spot_test.mp4

**Reasoning:** The optic disc, the point where the optic nerve leaves the retina, lacks light-detecting photoreceptor cells (rods and cones). Because there are no light sensors at this spot, it cannot send visual information to the brain, thus creating a gap in our vision.

---

### Task 2: Ishihara colour test
The Ishihara test is a colour vision test designed to detect deficiencies in the long and medium cones. It consists of one set of pictures containing colour dots with a number embedded within. Here's the link to perform the test: https://github.com/DE4-DVS-Labs/Lab2-Colour-Perception/blob/main/Ishihara_test.md

**Reasoning:** The Ishihara colour test is used to check whether someone has a red-green colour vision deficiency (a common type of colour blindness). It helps determine if a person’s colour vision is normal or impaired, and is often used in medical exams and job screenings where accurate colour perception is important.

It works by showing plates made of many coloured dots that form numbers or shapes. The colours of the dots are chosen so that people with normal colour vision see a clear figure, but people with red-green deficiencies see a different figure or no figure, because their cone cells cannot distinguish certain red-green mixtures, causing those dots to “blend” together instead of standing out.

---

### Task 3: Reverse colour
Look at the image of the flag below for 10 seconds and then look at a clear white surface. You should see the American flag with its original colours.
<p align="center">
<img width="931" height="550" alt="image" src="https://github.com/user-attachments/assets/bac7ed34-1276-420f-99a7-03963460bdea" />
</p>

**Reasoning:** When you stare at the oddly coloured flag, the cone cells in your eyes that respond to those specific colours become fatigued and reduce their response. When you then look at the white paper (which contains all wavelengths of visible light), the tired cones signal less strongly than the others, so your brain “subtracts” those colours and you perceive the complementary ones instead.

---

### Task 4: Troxler's fading

Watch this, which demonstrates Troxler's fading effect: http://www.ee.ic.ac.uk/pcheung/teaching/DE4_DVS/assets/purple_dots.mp4

**Reasoning:** In the video, you see a ring of purple dots with a central cross or marker. When you stare at the centre, the moving “gap” appears as a single greenish spot, and after a few seconds, the purple dots disappear so that only the moving greenish spot remains visible. This impression of the purple dots vanishing, even though they are still physically present, is Trozler's fading effect.

When you stare at the centre and keep your eyes very still, your brain slowly “switches off” the dots that don’t change, because it treats them as unimportant background. Normally, your eyes make tiny jumps that keep everything refreshed, but in this task, the purple dots stay almost the same on your retina, so their signal fades, and they seem to disappear, while the moving gap keeps changing and remains visible.

---

### Task 5: Brain interpretation according to expectation
The image below shows two tables with blue and red tops. Which is the longer table?
<p align="center">
<img width="917" height="503" alt="image" src="https://github.com/user-attachments/assets/4c5c4e7b-a912-4420-a081-b5d5d6f8b775" />
</p>
**Reasoning:** Most people say the blue table looks much longer and thinner than the red one, but if you measure them, the tabletops are actually the same size. This illusion happens because your brain automatically interprets the drawing using 3D depth cues: the slanted blue table is seen as receding into the distance like a foreshortened object, so your brain “stretches” its length to make sense of it in 3D, while the red table is seen more from above, so it looks shorter and wider even though, on the flat page, both shapes are identical.

Another example of this is the image below. Our brain sees what it expects instead of what hits the retina. Which square is darker, the one labelled A or B? Why?
<p align="center">
<img width="761" height="534" alt="image" src="https://github.com/user-attachments/assets/286c7618-ff59-4e93-8882-710f8145c4ca" />
</p>
**Reasoning:** Square A and square B look different in brightness, but if you isolate them you find they are actually the same shade of grey. Your brain assumes there is a light source and a shadow from the green cylinder, and it “corrects” for that shadow. It expects a square in shadow (B) to be lighter than it appears, so it boosts its perceived brightness, while it expects a square outside the shadow (A) to be relatively darker, so it keeps it looking dark, even though the light reaching your eyes from both squares is identical.

---

### Task 6: Grid Illusion
When you stare at the centre of the grid below, you should see black dots at the intersection appearing and disappearing.
<p align="center">
<img width="789" height="723" alt="image" src="https://github.com/user-attachments/assets/072d7c2d-26cc-4377-adf2-43c2cc044795" />
</p>
**Reasoning:** This happens because of how your visual system processes contrast and light. When you look directly at an intersection, the cones in your retina see it clearly, so the white dot looks normal. But when the intersections are in your peripheral vision, neurons that detect light suppress the activity of nearby neurons more strongly at high-contrast edges like the grey lines on a black background. This extra inhibition makes the intersections in your peripheral vision appear darker, so your brain interprets them as black dots. This is also known as lateral inhibition.

---

### Task 7: Cafe wall illusion
Look at the image below. Do you see the following brick wall layers are parallel? If so, then measure the boundaries of each layer with a ruler.
<p align="center">
<img width="578" height="287" alt="image" src="https://github.com/user-attachments/assets/12a27f78-7655-44bc-8a82-9994fe6ad9ef" />
</p>
**Reasoning:** Even though the horizontal brick layers look slanted, if you measure them with a ruler, you will find they are perfectly parallel. This happens because of the strong contrast between the black and white tiles and the thin grey lines between them. Your visual system uses edge detection to understand shapes, but the alternating light and dark blocks shift how edges are perceived. This causes small positional errors in your peripheral vision, so your brain incorrectly connects the edges at a slight angle. If you were to do the same with colours of lower contrast, you wouldn't see the same effect.

---

### Task 8: The silhouette illusion
Watch this video of a spinning dancer. Play the video and look at it for some time, you may find that the dance suddenly spins in the opposite direction: http://www.ee.ic.ac.uk/pcheung/teaching/DE4_DVS/assets/dancer.m4v

**Reasoning:** This happens because your brain interprets ambiguous motion in the 2D image. The dancer silhouette doesn’t have clear depth cues, so your visual system has to guess whether parts are in front or behind. Once your brain chooses a particular 3D orientation, it assumes the dancer is spinning in one direction. After a moment, your brain can switch to the alternate depth interpretation, making the dancer suddenly appear to spin the other way.

---

### Task 9: The incomplete triangles
Consider the picture below. How many triangles are in the picture? What conclusions can you draw from this observation?
<p align="center">
<img width="458" height="434" alt="image" src="https://github.com/user-attachments/assets/b2e3da82-17c6-4421-8308-21a53e7b5810" />
</p>
**Reasoning:** Most people say there is one large triangle in the centre, even though no triangle is actually drawn with complete lines. If you look carefully, the triangle’s edges are missing, but your brain still “sees” a clear white triangle pointing upward. You might also notice a few smaller incomplete triangle shapes around it.

This happens because of illusory contours and Gestalt perception. Your brain likes to organise visual information into simple, familiar shapes, so it automatically fills in missing edges to create a triangle. The black shapes and angled lines suggest where the triangle boundaries should be, and your visual system completes them for you.

---

### Task 10 + 11 + 12: Exploring colours in MATLAB

```matlab
imfinfo("peppers.png");
RGB = imread("peppers.png");
imshow(RGB);
```

---
<p align="center">
<img width="575" height="410" alt="image" src="https://github.com/user-attachments/assets/63c58a88-1069-475d-96c6-fc5d78804bb2" />
</p>

```matlab
I = rgb2gray(RGB);
figure              % start a new figure window
imshow(I);
imshowpair(RGB, I, 'montage')
```
<p align="center">
<img width="575" height="230" alt="image" src="https://github.com/user-attachments/assets/9cd6eb16-35d3-4669-bf2b-4e1d69ba5a72" />
</p>

```matlab
[R,G,B] = imsplit(RGB);
montage({R, G, B},'Size',[1 3])
```
<p align="center">
<img width="575" height="171" alt="image" src="https://github.com/user-attachments/assets/aa886a59-5ebc-44ed-b050-8a31fc49732d" />
</p>

```matlab
HSV = rgb2hsv(RGB);
[H,S,V] = imsplit(HSV);
montage({H,S,V}, 'Size', [1 3])
```
<p align="center">
<img width="575" height="171" alt="image" src="https://github.com/user-attachments/assets/cbc1412c-9cf1-4c60-8761-261ed670685f" />
</p>

This code loads an image (peppers.png) and shows it in different ways so you can understand its colour information. First, it reads the image and displays the original colour picture. Then it converts the image to grayscale, which means it removes colour and keeps only brightness, and shows the colour and grayscale images side by side for comparison. 

After that, the code splits the image into its red, green, and blue colour channels and displays them next to each other so you can see how much of each colour is present. 

Finally, it converts the image into HSV colour space and shows the hue, saturation, and value channels separately, which helps explain colour in terms of type of colour, colour strength, and brightness.
