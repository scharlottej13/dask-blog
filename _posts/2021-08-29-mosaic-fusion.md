---
layout: post
title: Mosaic Image Fusion
author: Volker Hilsenstein, Marvin Albert, and Genevieve Buckley
tags: [life science]
theme: twitter
---
{% include JB/setup %}

## Executive Summary

This blogpost shows a case study, where a researcher uses Dask for mosaic image fusion - stitching multiple smaller images together seamlessly into a very large field of view. Full code examples are available on GitHub from the `DaskFusion` repository: https://github.com/VolkerH/DaskFusion

This work was done by Volker Hilsenstein, in conjunction with Marvin Albert. Genevieve Buckley wrote the blogpost.

Volker Hilsenstein is a scientific software developer at EMBL in Theodore Alexandrov's lab.

## The problem

Volker works on [spatial metabolomics](https://www.ebi.ac.uk/training/online/courses/metabolomics-introduction/what-is/) and bio-image analysis.

### What is mosaic image fusion?

In microscopy, we often have very large samples that need to be imaged in high resolution. One way to achieve this is to take many smaller images of each part, and then fuse them together into a single large image.

### Software alternatives

Originally, the researchers intended to use the [BigStitcher plugin](https://imagej.net/plugins/bigstitcher/) in [Fiji](https://fiji.sc/). Big Stitcher is an excellent plugin, designed for exactly this type of mosaic image fusion workflow.

However, the researchers discovered that in practice for large numbers of tiles (several hundred) there was an unfortunate scaling behaviour, estimating roughly `O(n^2)` complexity or worse. (If you want to know more about what that means and why it's a bad thing, see this article on [Learning Big O Notation With O(n) Complexity](https://dzone.com/articles/learning-big-o-notation-with-on-complexity)). This is what prompted them to explore alternative options, and finally settle on using [Dask](https://dask.org/) to manage the image fusion, and the [napari image viewer](https://napari.org/) to display the results.

## The solution

Typically whenever we want to join dask arrays, we use [Stack, Concatenate, and Block](https://docs.dask.org/en/latest/array-stack.html). However, these are not good tools for mosaic image fusion, because:

1. There needs to be some overlap at the edges of each image tile, and
2. While the position of the image tile is known based on the sample stage motor coordinates, there is a small amount of uncertainty in this measurement.


Marvin's lightning talk on multi-view image fusion is up online now: https://www.youtube.com/watch?v=YIblUvonMvo&list=PLJ0vO2F_f6OBAY6hjRHM_mIQ9yh32mWr0&index=10

[MVRegFus](https://github.com/m-albert/MVRegFus)


`block_info` dictionary and creating some
pseudo-code together, I managed to implement mosaic fusion in Dask Image.

From the dask documentaion:
> Your block function gets information about where it is in the array by accepting a special `block_info` or `block_id` keyword argument.


### Results

Using the new method with Dask, a considerable speed-up with this implementation has been observed. Previously, for data with 600 image tiles it was taking several hours to complete the image fusion. Now with Dask, image fusion is complete in minutes instead of hours.

The image below shows mosaic image fusion with 63 image tiles, displayed in the napari image viewer.

![Mosaic fusion images in the napari image viewer](/images/mosaic-fusion/NapariMosaics.png)

And here is an animation of whole slide mosaic fusion.

![Animation of whole slide mosaic fusion images](/images/mosaic-fusion/Lama_whole_slide.gif)

### Code

Code relatiing to this mosaic image fusion project can be found in the `DaskFusion` GitHub repository here: https://github.com/VolkerH/DaskFusion

There is a self-contained example available in [this notebook](https://github.com/VolkerH/DaskFusion/blob/main/Load_Mosaic.ipynb), which downloads reduced-size example data to demonstrate the process.

## What's next?

Volker says, "If anyone is keen this could be amended for 3D and use Big Stitcher Project files as input". If that's something you might be interested in, get in touch!
