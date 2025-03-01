---
autogenerated: true
title: Test Block Matching Parameters
layout: page
categories: 
description: test description
---


{% capture author%}
{% include person content='Saalfeld' %} ([1](mailto:saalfeld@mpi-cbg.de))
{% endcapture %}

{% capture maintainer%}
{% include person content='Saalfeld' %}
{% endcapture %}
{% include info-box name='Test Block Matching Parameters' software='Fiji' author=author maintainer=maintainer source='https://fiji.sc/cgi-bin/gitweb.cgi?p=mpicbg.git;a=tree;f=mpicbg/ij/clahe' released='March 27<sup>th</sup>, 2012' latest-version='March 27<sup>th</sup>, 2012' status='experimental, active' category='[Plugins](Category_Plugins), [Registration](Category_Registration)' %} The plugin **Test Block Matching Parameters** is a helper plugin to explore the parameter space for block matching as used in the plugins for [elastic serial section registration](Elastic_Alignment_and_Montage). It takes as input a stack of pre-aligned RGB images with background rendered in green (rgb 0,255,0). It then asks for block matching parameters using the same dialog as in the alignment plugins. It will try to match all to all slices of the stack and render a matrix that displays all matching results as a color-coded translation field. That field will have blank entries where the matching result did not pass the filters.

What to look for?
-----------------

Non-systematic random colors  
are wrong matches and must be removed. Strengthening the filters and increasing the block-size may help.

<!-- -->

Systematic saturated colors  
indicate that the search radius is too small, local deformation are larger than what you set.

<!-- -->

Systematic blackness  
indicates that the search radius is too large, local deformation is smaller than what you set.

<!-- -->

Consistently much white gaps even for adjacent sections  
indicates that the filters may be too strict. Switch off the smoothness filter and trigger the local filters such that only a few randomly colored spots remain in the otherwise smooth field. Then adjust the smoothnes filter such that only those spots are removed.

<!-- -->

Matches only in the middle of the section with a large white border  
means that your block-size and search radius in combination may be too large. Matching is not performed beyond image borders.

General comments
----------------

You should generate the pre-aligned stack with [TrakEM2](TrakEM2) and export it at the resolution that you want to finally use (e.g. 10% for TEM series where the pixel size is 10× smaller than the section thickness) as RGB with green background (rgb 0,255,0). In the Test plugin you would then set **scale** to 1.0 and use accordingly scaled parameters for all fields that take their input in pixels. In the actual application, you then need to scale those parameters back (e.g. ×10 when you had used a scale of 10% for export).
