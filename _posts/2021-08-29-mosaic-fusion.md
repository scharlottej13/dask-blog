---
layout: post
title: Mosaic Image Fusion
author: Volker Hilsenstein, Marvin Albert, and Genevieve Buckley
tags: [life science]
theme: twitter
---
{% include JB/setup %}

## Executive Summary

This blogpost shows a case study, where a researcher uses Dask for mosaic image fusion.
Mosaic image fusion is when you combine multiple smaller images taken at known locations and stitch them together into a single image with a very large field of view. Full code examples are available on GitHub from the `DaskFusion` repository: https://github.com/VolkerH/DaskFusion

## The problem

### Image mosaicing in microscopy

In optical microscopy, a single field of view captured with a 20x objective typically
has a diagonal on the order of a few 100 um (exact dimensions depend on other
parts of the optical system, including the size of the camera chip). A typical 
sample slide has a size of 25mm * 70mm. 
So when imaging a whole slide, one has to acquire hundreds of images, typically
with some overlap between individual tiles.  With increasing magnifcation,
the required number of images increases accordingly. 

To obtain an overview one has to fuse this large number of individual
image tiles into a large mosaic image. Herer, we assume that the inforamtion requiring for 
positioning and alignment of the individual image tiles is already available. In our case,
this information is available based on the the metadata recorded by the microscope, however this
information could also be from  previous registration step that matches image features 
in the overlapping areas.

## The solution

The array that can hold resulting mosaic image will often have a size that is too large 
to fit in RAM, therefore we will use Dask arrays and the `map_block` function to enable 
out-of-core processing. As an added benefit, we will get parallel processing for free
which speeds up the fusion process.

Typically whenever we want to join Dask arrays, we use [Stack, Concatenate, and Block](https://docs.dask.org/en/latest/array-stack.html). However, these are not good tools for mosaic image fusion, because:

1. The image tiles will be be overlapping,
2. In the general case, individual image tiles may not be positioned on an exact grid and they can also have small rotations.

Using the `block_info` dictionary and creating some pseudo-code together, Volker managed to implement mosaic fusion with `dask-image`. From the [Dask documentaion](https://docs.dask.org/en/latest/generated/dask.array.map_blocks.html?highlight=block_info#dask.array.map_blocks):
> Your block function gets information about where it is in the array by accepting a special `block_info` or `block_id` keyword argument.

The basic outline of the image mosiac workflow is as follows:
1. Read in image tiles acquired from the microscope.
2. Calculate the location of the image tiles, using information from the sample stage position at the time each image tile was acquired.
3. Use an affine transformation to place the image tiles in their proper location.
4. Fuse the combined images into a single result.

### Results

Using the Dask-based image fusion method, we could achieve a speed up from several hours to tens of minutes compared to 
a previous method based on an ImageJ plugin for datasets with many image tiles (~500-1000 tiles) on similar workstation hardware.
Due to Dask's ability to handle data out-of-core and chunked array storage using zarr it is also possible to run the 
fusion on hardware with limited RAM.

The image below shows mosaic image tile placement for a small example with 63 image tiles, displayed in the napari image viewer.

![Mosaic fusion images in the napari image viewer](/images/mosaic-fusion/NapariMosaics.png)

And here is an animation of placing the individual tiles.

![Animation of whole slide mosaic fusion images](/images/mosaic-fusion/Lama_whole_slide.gif)

Finally, we have the final mosaic fusion result.

![Final mosaic fusion result](/images/mosaic-fusion/final-mosaic-fusion-result.png)

### Code

Code relatiing to this mosaic image fusion project can be found in the `DaskFusion` GitHub repository here: https://github.com/VolkerH/DaskFusion

There is a self-contained example available in [this notebook](https://github.com/VolkerH/DaskFusion/blob/main/DaskFusion_Example.ipynb), which downloads reduced-size example data to demonstrate the process.

## What's next?

Currently, the DaskFusion code is a proof of concept for single-channel 2D images and simple maximum projection for blending the tiles in overlapping areas, it is not production code.
However, the same principle can be used for fusing multi-channel image volumes,
such as from Light-Sheet data if the tile chunk intersection calculation is extended to higher-dimensional arrays.
Such even larger datasets will benefit even more from leveraging dask,
as the processing can be distributed across multiple nodes of a HPC cluster using [dask jobqueue](http://jobqueue.dask.org/en/latest/).

### Also see

Marvin's lightning talk on multi-view image fusion is up online now: https://www.youtube.com/watch?v=YIblUvonMvo&list=PLJ0vO2F_f6OBAY6hjRHM_mIQ9yh32mWr0&index=10

The GitHub repository [MVRegFus](https://github.com/m-albert/MVRegFus) that Marvin talks about in the video is available here: https://github.com/m-albert/MVRegFus

## Acknowledgements

This computational work was done by Volker Hilsenstein, in conjunction with Marvin Albert.
Volker Hilsenstein is a scientific software developer at [EMBL in Theodore Alexandrov's lab](https://www.embl.org/groups/alexandrov/) with a focus on spatial metabolomics and bio-image analysis.

The sample images were prepared and imaged by Mohammed Shahraz from the Alexandrov lab at EMBL Heidelberg.

Genevieve Buckley and Volker Hilsenstein wrote this blogpost.
