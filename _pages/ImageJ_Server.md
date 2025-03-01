---
autogenerated: true
title: ImageJ Server
layout: page
categories: SciJava
description: test description
---


{% capture source%}
{% include github org='imagej' repo='imagej-server' %}
{% endcapture %}
{% include info-box content='Plugin' software='ImageJ' name='ImageJ Server' author='ImageJ developers' maintainer='ImageJ developers' status='Experimental' source=source %}The ImageJ Server is an extension and [update site](Update_site) for ImageJ that enables ImageJ to act as a {% include wikipedia title='Representational state transfer' text='RESTful'%} image server.

Client software can:

-   Send images to, and receive images from, the server.
-   Discover and execute ImageJ modules.

Installation
------------

-   [Enable](Following_an_update_site) the Server [update site](Update_site).
-   Start the server via the {% include bc content='Plugins | Utilities | Start Server'%} command.
-   Or launch ImageJ in server mode using the `--server` flag.

Documentation
-------------

See the {% include github org='imagej' repo='imagej-server' label='GitHub site' %}.


