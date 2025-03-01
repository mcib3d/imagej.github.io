---
autogenerated: true
title: SciView
layout: page
categories: 
description: test description
---


{% capture author%}
{% include person content='Kharrington' %}, {% include person content='skalarproduktraum' %}, {% include person content='Rueden' %}
{% endcapture %}
{% include info-box logo='<img src="/media/SciView-icon.png" width="150"/>' name='SciView' software='ImageJ' author=author filename='' source=' [SciView](https://github.com/kephale/SciView)' released='in development' latest-version='in development' status='alpha' category='[Visualization](Category_Visualization)' website='https://github.com/scenerygraphics/SciView' %}== Purpose ==

This plugin provides 3D visualization and virtual reality capabilities for images and meshes using the [Scenery](https://github.com/scenerygraphics/scenery). SciView integrates [ImageJ2](ImageJ2) functionality, including [ImageJ Ops](ImageJ_Ops) and [ImageJ Mesh](ImageJ_Mesh), to provide the ability to interact with image and mesh data in 3D and interface with the popular [Fiji](Fiji) software ecosystem.

An update site is available: http://sites.imagej.net/SciView/

There have been a number of contributors to the project: {% include person content='Saalfeld' %}, {% include person content='Pietzsch' %}, {% include person content='royerloic' %}, and {% include person content='Haesleinhuepf' %}. Development has taken place at hackathons such as the first [SciView hackathon](2018-04-04_22-_SciView_hackathon).

![align=right](/media/Sciview-gameoflife.gif "align=right")

### Shortcuts

The full list of SciView's shortcuts can be accessed through the {% include bc content='Help| '%}menu. Basic navigation is accomplished using the following controls:

-   Drag - Move around
-   {% include key content='Shift\|' %}+Drag - Rotate around selected node
-   Single-click - Select node
-   Double-click - Centers clicked node
-   {% include key content='Shift\|' %}+Scroll - Zoom
-   {% include key content='W' %} {% include key content='A' %} {% include key content='S' %} {% include key content='D' %} - Move around (hold {% include key content='Shift' %} for slow movement)
