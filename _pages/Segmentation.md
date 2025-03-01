---
autogenerated: true
title: Segmentation
layout: page
categories: Tutorials,Segmentation
description: test description
---

{% include biginfo-box content='See [:Category:Segmentation](Category_Segmentation) for pages about image segmentation.' %} {% include learn content='techniques' %}

{% include tip tip='See [this helpful workshop on Image Segmentation](https://imagej.net/_images/8/87/Arganda-Carreras-Segmentation-Bioimage-course-MDC-Berlin-2016.pdf) for another great overview of Segmentation!' %}

Introduction
============

Image segmentation is "the process of partitioning a digital image into multiple segments." ({% include wikipedia title='Image segmentation' text='Wikipedia'%})

![](/media/Segmentation-overlay.jpg "fig:Segmentation-overlay.jpg") ![](/media/Segmentation-boundaries.jpg "fig:Segmentation-boundaries.jpg")

It is typically used to locate *objects* and *boundaries*.

More precisely, image segmentation is the process of *assigning a label* to every pixel in an image such that pixels with the same label share certain visual characteristics.

Easy workflow
=============

One plugin which is designed to be very powerful, yet easy to use for non-experts in image processing:

<table><tbody><tr class="odd"><td><p><strong>Plugin Name</strong></p></td><td><p><strong>Short Description</strong></p></td><td><p><strong>Highlights</strong></p></td><td><p><strong>Plugin Snapshot</strong></p></td></tr><tr class="even"><td><p><a href="https://imagej.net/Trainable_Weka_Segmentation">Trainable Weka Segmentation</a></p></td><td><p>A tool that combines a collection of machine learning algorithms with a set of selected image features to produce pixel-based segmentations.</p></td><td><ul><li>Can be trained to learn from the user input and perform later the same task in unknown (test) data</li><li>Makes use of all the powerful tools and classifiers from the latest version of <a href="http://www.cs.waikato.ac.nz/ml/weka/">Weka</a></li><li>Provides a labeled result based on the training of a chosen classifier</li><li>Ease of use due to its graphical user interfaces</li></ul></td><td><p><img src="/media/TWS-GUI-after-training.png" width="500"/></p></td></tr><tr class="odd"><td></td><td></td><td></td><td></td></tr></tbody></table>

Give it a try—you might like it!

Flexible workflow
=================

One good workflow for segmentation in ImageJ is as follows:

1.  [ Preprocess](Segmentation#Preprocessing) the given images
2.  Apply an [ Auto Threshold](Segmentation#Adjusting_Threshold)
3.  Create and manipulate a [ mask](Segmentation#Creating_Masks)
4.  [ Create and transfer](Segmentation#Selections#Creating_Selections) a selection from a mask to your original image
5.  [ Analyze](Segmentation#Analysis) the resulting data

Preprocessing
-------------

Preprocess the image using filters, to make later thresholding more effective. Which filter(s) to use is highly dependent on your data, but some commonly useful filters include:

-   [Deconvolution](Category_Deconvolution)
-   \[https://imagej.net/docs/guide/146-29.html#sub:Subtract-Background... Subtract Background\]
-   \[https://imagej.net/docs/guide/146-29.html#sub:Gaussian-Blur... Gaussian Blur\]
-   [Find Edges](https://imagej.net/docs/guide/146-29.html#sub:Find-Edges)

Adjusting Threshold
-------------------

<figure><img src="/media/ Threshold tree.png" title="Tree ring sample image with a threshold applied for a B&amp;W image" width="300" alt="Tree ring sample image with a threshold applied for a B&amp;W image" /><figcaption aria-hidden="true">Tree ring sample image with a threshold applied for a B&amp;W image</figcaption></figure>

Ideally you want to use one of the auto-threshold methods, rather than manually tweaking, so that your result is reproducible later on the same data, and on multiple other datasets.

-   Open your image
-   Select {% include bc content='Image | Adjust | Threshold...'%}
-   Specify whether or not the background should be dark or light
-   Adjust the minimum and maximum sliders until you are satisfied with the saturation level of your image
    -   \[https://imagej.net/docs/guide/146-28.html#sub:Threshold...%5BT%5D More information\]

Creating Masks
--------------

<figure><img src="/media/Eroded tree.png" title="Over-saturated mask is eroded around the center tree ring" width="300" alt="Over-saturated mask is eroded around the center tree ring" /><figcaption aria-hidden="true">Over-saturated mask is eroded around the center tree ring</figcaption></figure>

-   Select {% include bc content='Edit | Selection | Create Mask'%}
-   Based on the image and set threshold, some portions of the image may be over/under saturated
    -   Select the portion of the image that needs to be adjusted
    -   Select [Dilate](https://imagej.net/docs/guide/146-29.html#sub:Dilate) to grow the included pixels to further saturate this portion of the image or [Erode](https://imagej.net/docs/guide/146-29.html#sub:Erode) to remove saturation
        -   [More information](https://imagej.net/docs/guide/146-29.html#infobox:InvertedLutMask).
-   One quick way to split overlapping objects is the [Watershed](https://imagej.net/docs/guide/146-29.html#sub:Watershed) command.

Selections
----------

<figure><img src="/media/ Selection tree.png" title="Selections on the mask" width="300" alt="Selections on the mask" /><figcaption aria-hidden="true">Selections on the mask</figcaption></figure>

### Creating Selections

-   Select {% include bc content='Edit | Selection | Create Selection'%} to select the objects within the mask
-   To deselect a portion of the image, select {% include key content='Shift' %}+{% include key content='click' %}
    -   [More information](https://imagej.net/docs/guide/146-27.html#sub:Create-Selection)

<figure><img src="/media/ Reverted tree.png" title="Selections on the reverted image" width="300" alt="Selections on the reverted image" /><figcaption aria-hidden="true">Selections on the reverted image</figcaption></figure>

### Transferring Selections

-   Before transferring the mask's selections, revert the image to its original form by selecting {% include key content='Shift' %}+{% include key content='E' %}
-   Select first the mask, then the original image, and select {% include key content='Shift' %}+{% include key content='E' %} to transfer the mask's selections
    -   [More information](https://imagej.net/docs/guide/146-27.html#infobox:TransferSelections)

Analysis
--------

Do some numerical analysis on the selected data:

-   \[https://imagej.net/docs/guide/146-30.html#sub:Measure...%5Bm%5D Measure\] the entire selection directly.
    -   Control which measurements are done using \[https://imagej.net/docs/guide/146-30.html#sub:Set-Measurements... Set Measurements\].
-   Use \[https://imagej.net/docs/guide/146-30.html#sub:Analyze-Particles... Analyze Particles\] to extract desirable objects from your selection and report individual statistics on them.
-   Use the [ROI Manager](https://imagej.net/docs/guide/146-30.html#fig:The-ROI-Manager) to **Add** the selection and then **Split** it (under the **More** button), then use **Multi Measure** (also under **More**) to report statistics on the objects.
-   [Write a macro](Macros_Intro) to automate this sort of analysis, loop over objects in the ROI manager, measure and manipulate them, etc.

See also
========

-   The [Introduction to Image Segmentation using ImageJ/Fiji](https://imagej.net/_images/8/87/Arganda-Carreras-Segmentation-Bioimage-course-MDC-Berlin-2016.pdf) workshop.
-   The [Segmentation with Fiji workshop slides](http://imagej.github.io/presentations/fiji-segmentation/).
-   [:Category:Segmentation](Category_Segmentation), a list of pages about image segmentation.

 
