---
autogenerated: true
title: Wavefront .OBJ ...
layout: page
categories: 
description: test description
---


{% capture source%}
{% include github org='fiji' repo='VIB' source='marchingcubes/ExportMesh\_.java' %}
{% endcapture %}
{% include info-box software='ImageJ' name='ExportMesh\_' author='Benjamin Schmid, Mark Longair' maintainer='Mark Longair' filename='VIB\_.jar' source=source status='stable' %}

This plugin will threshold a stack and create a surface using marchingcubes, and then save that as a Wavefront .OBJ file.

There is an option to export surfaces in this format from the [3D Viewer](3D_Viewer) but this option may still be useful for you in the following situations:

-   If the 3D Viewer doesn't work for you (or is very slow)
-   If you are using Fiji's --headless option
