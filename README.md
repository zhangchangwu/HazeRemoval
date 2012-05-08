HazeRemoval
===========

Overview
====

Haze removal (or dehazing) is the process in computer vision that attempts to remove haze, or fog, from images. The process implemented for this project utilizing the dark channel of images.  

![dark channel](https://github.com/KnownSubset/HazeRemoval/raw/master/DtSx.jpg "dark channel") 

The dark channel is the color channel of the image that has the lowest intensity.  The dark channel prior is based on the following observation on haze-free outdoor images: in most of the non-sky patches, at least one color channel has very low intensity at some pixels. In other words, the minimum intensity in such a patch should has a very low value. 

#Haze Removal

---

### Dark Channel

I used 15x15 patches as I iterated over the entire image. For each patch we find the color channel that has the minimum value and use that patch in the final dark channel.

######Pseudo-Code
    
    ```matlab
    %image is already available
    cols = size(image, 2);
    rows = size(image, 1);
    dark_channel = zeros(rows, cols);
    for ix = 8:rows-8
        for iy = 8:cols-8
            dark_channel(ix-7:ix+7, iy-7:iy+7) = find_minimum(image(ix-7:ix+7, iy-7:iy+7));
        end
    end
    ```


### Atmospheric Light

The atmospheric light is calculated using the dark channel and the original image.  To ensure that we select the brightest pixels that are not objects within the image, you must look at the pixels in the dark channel.  The simple approach used in the paper finds the .1% brightest pixels of the dark channel, then selects the maximum value from those pixels in the original image.

######Pseudo-Code


    ```matlab
    %image & dark_channel are already available
    [cols rows] = size(dark_channel);
    [vector index] = sort(reshape(dark_channel, cols * rows, []), 1, 'descend');
    % take the brightest .1% of the dark channel
    limit = round(cols * rows /1000);
    for ix = 1:limit
        vector(ix) = max(image(floor(index(ix)/rows)+1, mod(index(ix),rows)+1,:));
    end
    atmospheric_light= max(vector(1:limit));
    ```

### Transmission

######Pseudo-Code
    ```matlab
    cols = size(image, 2);
    rows = size(image, 1);
    t = zeros(rows, cols);
    w = .95;
    for ix = 8:rows-8
        for iy = 8:cols-8
            t(ix-7:ix+7, iy-7:iy+7) = calculate_transmission(image(ix-7:ix+7, iy-7:iy+7), atmospheric_light, w);
        end
    end
    ```

### Scene Radiance

This is the final image that will have the hazyiness removed.  

######Pseudo-Code
    ```matlab
    cols = size(image, 2);
    rows = size(image, 1);
    r = zeros(rows, cols, 3);
    t(:,:,1) = transmission;
    for ic = 1:3
        for ix = 1:rows
            for iy = 1:cols
                r(ix, iy, ic) = (image(ix, iy, ic) - atmospheric_light) / (max(t(ix, iy), .1)) + atmospheric_light;
            end
        end
    end
    ```


since ω is parameterizable and this process is able to generate a depth map.  After generating the first of the depth map, you could reprocess the image utilizing the depth map info for ω.