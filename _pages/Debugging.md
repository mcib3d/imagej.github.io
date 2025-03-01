---
autogenerated: true
title: Debugging
layout: page
categories: Development
description: test description
---

{% include develop-menu%}
 {% include info-box content='This page has approaches for ""software developers"" to use for debugging ImageJ.  
If you are a ""user"" looking to troubleshoot issues, see the [Troubleshooting](Troubleshooting) page.' %}

Launching ImageJ in debug mode
==============================

To debug problems with ImageJ, it is often helpful to launch it in debug mode. See the [Troubleshooting](Troubleshooting#Launching_ImageJ_from_the_console) page for instructions.

Debugging plugins in an IDE (Netbeans, IntelliJ, Eclipse, etc)
==============================================================

To debug a plugin in an IDE, you most likely need to provide a *main()* method. To make things easier, we provide helper methods in *fiji-lib* in the class *fiji.Debug* to run plugins, and to load images and run filter plugins:

    import fiji.Debug;

    ...

    public void main(String[] args) {
        Debug.runFilter("/home/clown/my-profile.jpg", "Gaussian Blur", "radius=2");
    }

You need to replace the first argument by a valid path to a sample image and the second argument by the name of your plugin (typically the class name with underscores replaced by spaces).

If your plugin is not a filter plugin, i.e. if it does not require an image to run, simply use the *Debug.run(plugin, parameters)* method.

Attaching to ImageJ instances
-----------------------------

Sometimes, we need to debug things directly in ImageJ, for example because there might be issues with the plugin discovery (ImageJ wants to find the plugins in *<ImageJ>/plugins/*, and often we want to bundle them as *.jar* files, both of which are incompatible with Eclipse debugging). JDWP (*Java Debug Wire Protocol*) to the rescue!

After starting the Java Virtual Machine in a special mode, debuggers (such as Eclipse's built-in one) can attach to it. To start ImageJ in said mode, you need to pass the *--debugger=<port>* option:

    ImageJ.app/ImageJ-linux64 --debugger=8000

In Eclipse (or whatever {% include wikipedia title='JDWP' text='JDWP'%}-enabled debugger) select the correct project so that the source code can be found, mark the break-points where you want execution to halt (e.g. to inspect variables' values), and after clicking on *Run&gt;Debug Configurations...* right-click on the *Remote Java Application* item in the left-hand side list and select *New*. Now you only need to make sure that the port matches the value that you specified (in the example above, *8000*, Eclipse's default port number).

If you require more control over the ImageJ side -- such as picking a semi-random port if port 8000 is already in use -- you can also use the *-agentlib:jdwp=...* Java option directly (*--debugger=<port>* is just a shortcut for convenience):

    ImageJ.app/ImageJ-linux64 -agentlib:jdwp=server=y,suspend=y,transport=dt_socket,address=localhost:8000 --

(the *--* marker separates the Java options -- if any -- from the ImageJ options). Once started that way, ImageJ will wait for the debugger to be attached, after printing a message such as:

> Listening for transport dt\_socket at address: 46317

**Note**: calling *imagej -agentlib:jdwp=help --* will print nice usage information with documentation of other JDWP options.

Attach ImageJ to a waiting Eclipse
----------------------------------

Instead of making ImageJ [the debugging server](#Attaching_to_ImageJ_instances "wikilink"), when debugging startup events and headless operations it is easier to make ImageJ the client and Eclipse (or equivalent) the server.

In this case you start the debugging session first, e.g. in Eclipse debug configurations you specify "Standard (Socket Listen)" as the connection type. Then, simply start ImageJ without the "server=y" flag to connect and debug:

    ImageJ.app/ImageJ-linux64 -agentlib:jdwp=suspend=y,transport=dt_socket,address=localhost:8000 --

Monitoring system calls
=======================

Linux
-----

On Linux, you should call ImageJ using the [strace command](http://www.linuxmanpages.com/man1/strace.1.php):

    strace -Ffo syscall.log ./imagej <args>

MacOSX
------

Use the *dtruss* wrapper around [dtrace](http://developer.apple.com/documentation/Darwin/Reference/ManPages/man1/dtrace.1.html) to monitor system calls:

    dtruss ./imagej <args>

Windows
-------

To monitor all kinds of aspects of processes on Windows, use [Sysinternal's Process Monitor](http://technet.microsoft.com/en-us/sysinternals/bb896645.aspx).

Debugging shared (dynamic) library issues
=========================================

Linux
-----

Set the *LD\_DEBUG* environment variable before launching ImageJ:

    LD_DEBUG=1 ./imagej <args>

MacOSX
------

Set the *DYLD\_PRINT\_APIS* environment variable before launching ImageJ:

    DYLD_PRINT_APIS=1 ./imagej <args>

Windows
-------

Often, dynamic library issues are connected to a dependent .dll file missing. Download [depends.exe](http://www.dependencywalker.com/) and load the .dll file you suspect is missing a dependency.

Debugging JVM hangs
===================

When the Java VM hangs, the reason might be a dead-lock. Try taking a [stack trace](Troubleshooting#If_ImageJ_freezes_or_hangs). If you have trouble, you can try one of the following advanced techniques:

1.  You can use the `jstack` command (you don't need to run ImageJ from the command line in this case). This requires that you first find the PID (process ID) of ImageJ. You can do so by running:
        jps

    from the command line to print a list of running Java processes. If you're not sure which PID is ImageJ's, you can close ImageJ, run `jps`, open ImageJ and run `jps` again. Whichever PID is present in the second run but not the first is ImageJ's. Then, to acquire a stack trace, just run:

        jstack <ImageJ's PID>
2.  For GUI-based debugging, can also attach to the ImageJ PID using the `jvisualvm` program that you can find in `java/`<platform>`/`<jdk>`/bin/`. Here you can simply press a big *Thread Dump* button to view the stack trace.
    MacOSX users, please note that Apple decided that the VisualVM tool should no longer be shipped with the Java Development Kit; you will have to download it [from here](http://visualvm.java.net/download.html).

Regardless of which method you use to acquire the stack trace, to debug you will want to acquire multiple stack traces over time and compare. If all the stack traces are in the same method execution, then that's the source of the deadlock (or slowdown).

Debugging memory leaks
======================

Sometimes, memory is not released properly, leading to OutOfMemoryExceptions.

One way to find out what is happening is to use `jvisualvm` (see [\#Debugging JVM hangs](#Debugging_JVM_hangs "wikilink")) to connect to the ImageJ process, click on *Heap Dump* in the *Monitor* tab, in said tab select the sub-tab *Classes* and sort by size. Double-clicking on the top user should get you to a detailed list of *Instances* where you can expand the tree of references to find out what is holding a reference still.

Debugging hard JVM crashes
==========================

When you have found an issue that crashes the JVM, and you can repeat that crash reliably, there are a number of options to find out what is going on.

Using gdb
---------

Typically when you debug a program that crashes, you start it in a debugger, to inspect the stack trace and the variables at the time of the crash. However, there are substantial problems with gdb when starting the Java VM; either gdb gets confused by segmentation faults (used by the JVM to handle NullPointerExceptions in an efficient manner), or it gets confused by the threading system -- unless you compile gdb yourself.

But there is a very easy method to use gdb to inspect serious errors such as segmentation faults or trap signals nevertheless:

    ./imagej -XX:OnError="gdb - %p" --

Using lldb
----------

On newer OS X versions, gdb has been replaced with lldb. For those familiar with gdb already, there is an [LLDB to GDB Command Map](http://lldb.llvm.org/lldb-gdb.html) cheat sheet which may be useful.

Using the *hs\_err\_pid<pid>.log* files
---------------------------------------

The Java virtual machine (JVM) frequently leaves files of the format *hs\_err\_pid<number>.log* in the current working directory after a crash. Such a file starts with a preamble similar to this:

    #
    # A fatal error has been detected by the Java Runtime Environment:
    #
    #  SIGSEGV (0xb) at pc=0x00007f3dc887dd8b, pid=12116, tid=139899447723792
    #
    # JRE version: 6.0_20-b02
    # Java VM: Java HotSpot(TM) 64-Bit Server VM (16.3-b01 mixed mode linux-amd64 )
    # Problematic frame:
    # C  [libc.so.6+0x86d8b]  memcpy+0x15b
    #
    # If you would like to submit a bug report, please visit:
    #   http://java.sun.com/webapps/bugreport/crash.jsp
    # The crash happened outside the Java Virtual Machine in native code.
    # See problematic frame for where to report the bug.
    #

followed by thread dumps and other useful information including the command-line arguments passed to the JVM.

The most important part is the line after the line *\# Problematic frame:* because it usually gives you an idea in which component the crash was triggered.

Out of memory error
-------------------

If the specific exception you're receiving (or you suspect) is an OutOfMemoryError, there are JVM flags that can be enabled when running ImageJ to help pinpoint the problem:

    ./imagej -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/desired/path/

The first option:

    -XX:+HeapDumpOnOutOfMemoryError

tells the JVM to create a heap dump (.hprof file) if an OutOfMemoryError is thrown. This is basically a snapshot of the JVM state when memory ran out.

The second option:

    -XX:HeapDumpPath=/desired/path/

is not required, but convenient for controlling where the resulting .hprof file is written. Note that these heap dumps are named by PID, and thus are not easily human distinguishable.

After acquiring a heap dump, you can analyze it yourself, e.g. with a [memory analyzer](http://www.eclipse.org/mat/), or post on \[Mailing Lists\|imagej-devel\] with a brief explanation of your problem.

Debugging Java code with jdb
============================

How to attach the Java debugger jdb to a running ImageJ process
---------------------------------------------------------------

This requires two separate processes, ImageJ itself and the debugger. You can do this either in one shell, backgrounding the first process or in two shells, this is recommended. In the two shells do the following:

Shell 1  
In the first shell, start ImageJ with special parameters to open a port (8000 in this case) to which jdb can connect afterwards:

<!-- -->

    ./imagej -Xdebug -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=y --

  
(Tested with Java 1.5.0, ymmv)

Shell 2  
In the second shell, tell jdb to attach to that port:

<!-- -->

    jdb -attach 8000

This is an ultra quick start to jdb, the default Java debugger
--------------------------------------------------------------

Hopefully you are a little familiar with gdb, since jdb resembles it lightly.

Notable differences:

-   a breakpoint is set with "stop in <class>.<method>" or "<class>:<line>". Just remember that the class must be fully specified, i.e. <package>.&lt;subpackages...&gt;.<classname>
-   no tab completion
-   no readline (cursor up/down)
-   no shortcuts; you have to write "run", not "r" to run the program
-   no listing files before the class was loaded
-   much easier method to specify the location of the source: "use
    <dir>

    "
-   "until" is "step", "step" is "stepi"

Okay, so here you go, a little demonstration:

(If you attach jdb to a running ImageJ process, you have to use the line from the previous section instead.)

`$ jdb -classpath ij.jar ij.ImageJ`  
`> stop in ij.ImageJ.main`  
`Deferring breakpoint ij.ImageJ.main.`  
`It will be set after the class is loaded.`  
`> run`  
`run ij.ImageJ`  
`Set uncaught java.lang.Throwable`  
`Set deferred uncaught java.lang.Throwable`  
`>`  
`VM Started: Set deferred breakpoint ij.ImageJ.main`  
  
`Breakpoint hit: "thread=main", ij.ImageJ.main(), line=466 bci=0`  
  
`main[1] use .`  
`main[1] list`  
`462             //prefs.put(IJ_HEIGHT, Integer.toString(size.height));`  
`463     }`  
`464`  
`465     public static void main(String args[]) {`  
`466 =>          if (System.getProperty("java.version").substring(0,3).compareTo("1.4")<0) {`  
`467                     javax.swing.JOptionPane.showMessageDialog(null,"ImageJ "+VERSION+" requires Java 1.4.1 or later.");`  
`468                     System.exit(0);`  
`469             }`  
`470             boolean noGUI = false;`  
`471             arguments = args;`  
`main[1] print args[0]`  
`java.lang.IndexOutOfBoundsException: Invalid array range: 0 to 0`  
` args[0] = null`  
`main[1] print args.length`  
` args.length = 0`  
`main[1] step`  
`>`  
`Step completed: "thread=main", ij.ImageJ.main(), line=470 bci=28`  
`470             boolean noGUI = false;`  
  
`main[1] step`  
`>`  
`Step completed: "thread=main", ij.ImageJ.main(), line=471 bci=30`  
`471             arguments = args;`  
  
`main[1] set noGUI = true`  
` noGUI = true = true`  
`main[1] cont`  
`>`  
`The application exited`

Inspecting serialized objects
=============================

If you have a file with a serialized object, you can use this Beanshell in the [Script Editor](Script_Editor) to open a tree view of the object (double-click to open/close the branches of the view):

    import fiji.debugging.Object_Inspector;

    import ij.io.OpenDialog;

    import java.io.FileInputStream;
    import java.io.ObjectInputStream;

    dialog = new OpenDialog("Classifier", null);
    if (dialog.getDirectory() != null) {
        path = dialog.getDirectory() + "/" + dialog.getFileName();
        in = new FileInputStream(path);
        in = new ObjectInputStream(in);
        object = in.readObject();
        in.close();
        Object_Inspector.openFrame("classifier", object);
    }

Debugging Swing (Event Dispatch Thread) issues
==============================================

Swing does not allow us to call all the methods on all UI objects from wherever we want. Some things, such as *setVisible(true)* or *pack()* need to be called on the Event Dispatch Thread (AKA EDT). See Sun's [detailed explanation](http://java.sun.com/products/jfc/tsc/articles/threads/threads1.html) as to why this is the case.

There are a couple of ways to test for such EDT violations, see [this blog post by Alexander Potochkin](http://weblogs.java.net/blog/alexfromsun/archive/2006/02/debugging_swing.html) (current versions of debug.jar can be found [here](http://java.net/projects/swinghelper/sources/svn/show/trunk/www/bin)).

Debugging Java3D issues
=======================

When Java3D does not work, the first order of business is to use {% include bc content='Plugins | Utilities | Debugging | Test Java3D'%}. If this shows a rotating cube, but the [3D Viewer](3D_Viewer) does not work, please click on {% include bc content='Help | Java3D Properties...'%} in the [3D Viewer](3D_Viewer)'s menu bar.

Command line debugging
----------------------

If this information is not enough to solve the trouble, or if *Test Java3D* did not work, then you need to call ImageJ from the command line to find out more.

From the command line, you have several options to show more or less information about Java3D.

### Windows & Linux

Please find the *ImageJ-<platform>* executable in the ImageJ.app/ directory (on 32-bit Windows, that would be *ImageJ-win32.exe*. Make a copy in the same directory and rename that to *debug* (on Windows: *debug.exe*). Simply double-click that.

On Windows, you will see a console window popping up; to copy information for pasting somewhere else, please right-click the upper-left window icon, select *Properties...*, activate the *Quick Edit* mode. Then mark the text in question by dragging the mouse with the left mouse button pressed, and copy it to the clipboard by right-clicking.

On Linux, the output will be written to the file *.xsession-errors* in the home directory.

### MacOSX

On MacOSX, you need to remember that any application is just a directory with a special layout. So you can call ImageJ like this from the *Terminal* (which you will find in the Finder by clicking on *Go&gt;Utilities*. Example command line:

    cd /Applications/ImageJ.app/Contents/MacOS/
    cp ImageJ-macosx debug
    ./debug

Show Java3D debug messages
--------------------------

    ./imagej -Dj3d.debug=true --

(Of course, you need to substitute the *./imagej* executable name with the appropriate name for your platform.)

**Note:** do not forget the trailing *--*; without them, ImageJ mistakes the first option for an ImageJ option rather than a Java one. **Note, too:** on Windows, you <u>must not</u> forget to pass the *--console* option (this can be anywhere on the command line).

Windows-specific stuff
----------------------

On Windows, you can choose between OpenGL and Direct3D by passing *-Dj3d.rend=ogl* or *-Dj3d.rend=d3d*, respectively.

Further, some setups require enough RAM to be reserved, so you might need to pass an option like *--mem=1200m* (make sure that you have enough RAM free before starting ImageJ that way, though!). If it turns out that memory was the issue, you can make the setting permanent by clicking ImageJ's {% include bc content='Edit | Options | Memory & Threads...'%} menu entry.

More Java 3D properties
-----------------------

You can control quite a few things in Java 3D through setting Java properties. Remember, you can set properties using a command line like this:

    ./imagej -D<property-name>=<property-value> --

where you substitute *<property-name>* and *<property-values>* appropriately. You can have more than one such option, but make sure that they are appearing before the *--* on the command line, otherwise ImageJ will mistake them for ImageJ options.

This list of Java 3D properties was salvaged from the now-defunct j3d.org website:

<table><thead><tr class="header"><th><p>Property</p></th><th><p>Values</p></th><th><p>Java 3D version</p></th><th><p>Explanation</p></th></tr></thead><tbody><tr class="odd"><td><p>j3d.rend</p></td><td><p>"ogl" or "d3d"</p></td><td><p>1.3.2</p></td><td><p>Win32-only. Specifies which underlying rendering API should be used (thus allowing both Direct3D and OpenGL native DLLs to be installed on a singe machine. (default value "ogl")</p></td></tr><tr class="even"><td><p>j3d.deviceSampleTime</p></td><td><p>A time in millseconds</p></td><td><p>1.1</p></td><td><p>The sample time for non-blocking input devices (default value is 5ms).</p></td></tr><tr class="odd"><td><p>j3d.disablecompile</p></td><td><p>N/A</p></td><td><p>1.2</p></td><td><p>If set turns off the ability to internally .compile() the scenegraph.</p></td></tr><tr class="even"><td><p>j3d.docompaction</p></td><td><p>true or false</p></td><td><p>1.3</p></td><td><p>Default true. Controls whether or not objects are removed from the render cache when they haven't been visibile for a while. If it is disabled, they stay in the render cache from when they are first visible until they are removed from the scene graph.</p></td></tr><tr class="odd"><td><p>j3d.forceReleaseView</p></td><td><p>true or false</p></td><td><p>1.3.2</p></td><td><p>Default false. If this flag is set to true, the view is released after the Canvas3D is removed from the view. Can be used if you have memory leaks after disposing Canvas3D. Note: Setting this flag as true disables the bug fix 4267395 in View deactivate()</p></td></tr><tr class="even"><td><p>j3d.implicitAntialiasing</p></td><td><p>true or false</p></td><td><p>1.3</p></td><td><p>Default false. By default, full scene antialiasing is disabled if a multisampling pixel format (or visual) is chosen. To honor a display drivers multisample antialiasing setting (e.g. force scene antialiasing), set the implicitAntialiasing property to true. This causes Java3D to ignore its own scene antialias settings, letting the driver implicitly implement the feature</p></td></tr><tr class="odd"><td><p>j3d.optimizeForSpace</p></td><td><p>true or false</p></td><td><p>1.3</p></td><td><p>Default true Java3d only builds display list for by-copy geometry. Set to false will cause Java3d to build display list for by-ref geometry and infrequently changing geometry using more space, but having greater speed.</p></td></tr><tr class="even"><td><p>j3d.renderLock</p></td><td><p>true or false</p></td><td><p>1.3</p></td><td><p>JDK requires getting the JAWT_DrawingSurfaceInfo and lock the surface before Java3D render on the canvas. (see comment on jdk/include/jawt.h in the SDK) Default false causes Java3D to lock the surface before rendering and unlock it afterwards for onScreen rendering in the Renderer thread. For OffScreen rendering and for front/back buffer swapping the lock will not acquired. Setting the value to true will force Java3D lock the surface using the AWT mechanism before swap() and for offScreen rendering. This may be useful for some driver/OS to workaround problem. But in general the default should work.</p></td></tr><tr class="odd"><td><p>j3d.threadLimit</p></td><td><p>An integer</p></td><td><p>1.2</p></td><td><p>Controls how many threads may run in parallel regardless of how many cpu's the system has. Setting it to "1" will make the system act like a traditional OpenGL render loop. The default value is the number of CPUs in your machine + 1.</p></td></tr><tr class="even"><td><p>j3d.transparentOffScreen</p></td><td><p>true or false</p></td><td><p>1.3.2</p></td><td><p>Default false. If this flag is set to true the background of the off screen canvas is set to transparent.</p></td></tr><tr class="odd"><td><p>j3d.usePbuffer</p></td><td><p>true or false</p></td><td><p>1.3.2</p></td><td><p>Default true. If this flag is set to false pbuffer will not be use for off screen rendering.</p></td></tr><tr class="even"><td><p>j3d.viewFrustumCulling</p></td><td><p>true or false</p></td><td><p>1.3.2</p></td><td><p>Default true. If this flag is set to false, the renderer view frustum culling is turned off. Java 3D uses a 2 pass view culling. The first pass is a loose view culling of the spatial tree, and the second pass is a tight view frustum culling in the renderer before sending the geometry down to the low level graphics API. This property is to control the renderer view frustum culling, and it will not affect the first pass view culling.</p></td></tr><tr class="odd"><td><p>javax.media.j3d.compileStats</p></td><td><p>N/A</p></td><td><p>??</p></td><td><p>Output scenegraph compilation statistics</p></td></tr><tr class="even"><td><p>javax.media.j3d.compileVerbose</p></td><td><p>N/A</p></td><td><p>??</p></td><td><p>Output verbose message when compiling scenegraph</p></td></tr><tr class="odd"><td><p><br />
OpenGL Only</p></td><td></td><td></td><td></td></tr><tr class="even"><td><p>j3d.backgroundtexture</p></td><td><p>true or false</p></td><td><p>1.3</p></td><td><p>Prior to Java3D 1.3 OGL version of Java3D used glDrawPixels() to render background, which is known to be very slow under Windows since most window driver did not accelerate the function. To workaround this performance problem current release uses textures for the backgrond under windows by default (glDrawPixels() is used as default under Solaris). Setting this flag to false will force Java3D fall back to use glDrawPixels() instead of texture when drawing background texture in case it provide better performance under some drivers.</p></td></tr><tr class="odd"><td><p>j3d.disableSeparateSpecular</p></td><td><p>true or false</p></td><td><p>1.2</p></td><td><p>Default true enables the use of specular highlights in textures when using OpenGL 1.2.</p></td></tr><tr class="even"><td><p>j3d.disableXinerama</p></td><td><p>true or false</p></td><td><p>1.3</p></td><td><p>Solaris version only. Allows major performance boost when using dual screen environments with the X11 Xinerama extension enabled. To disable this feature you need JDK1.4. Detailed information in the release notes.</p></td></tr><tr class="odd"><td><p>j3d.displaylist</p></td><td><p>true or false</p></td><td><p>1.2</p></td><td><p>Default true to use display lists (an OpenGL performance enhancing feature). False to disable for debugging.</p></td></tr><tr class="even"><td><p>j3d.g2ddrawpixel</p></td><td><p>true or false</p></td><td><p>1.1</p></td><td><p>If false, this will use texture mapping instead of glDrawPixel to flush the graphics2D to the screen. glDrawPixel is not accelerated on some older video cards (windows).</p></td></tr><tr class="odd"><td><p>j3d.sharedctx</p></td><td><p>true or false</p></td><td><p>1.2</p></td><td><p>Default true for Solaris and false for windows. Shared contexts are used in OpenGL for DisplayLists and Texture Objects to improve performance. However some drivers have bugs causing weird rendering artifacts. This can be used to disable their use to see if this is the problem.</p></td></tr><tr class="even"><td><p>j3d.sharedstereozbuffer</p></td><td><p>true or false</p></td><td><p>1.2</p></td><td><p>Some framebuffers only have one Z buffer and share this between the left and right eyes. This may be the reason why they don't have quad buffer but can still support stereo by setting this flag to true.</p></td></tr><tr class="odd"><td><p>j3d.usecombiners</p></td><td><p>true or false</p></td><td><p>1.3</p></td><td><p>Default false, uses the standard OpenGL all environment options. When set to true, it will make use of the Nvidia register combiner extensions to OpenGL for for Texture combine modes such as COMBINE_INTERPOLATE, COMBINE_DOT3. (ie GL_NV_register_combiners instead of standard OpenGL call glTexEnvi(GL_TEXTURE_ENV, ...)). It can be use in case like Dot3 texture when the driver does not support OpenGL extension GL_ARB_texture_env_dot3/GL_EXT_texture_env_dot3 but it supports the GL_NV_register_combiners extension instead.</p></td></tr><tr class="even"><td><p><br />
DirectX only</p></td><td></td><td></td><td></td></tr><tr class="odd"><td><p>j3d.d3ddevice</p></td><td><p>"emulation" or "hardware" or "tnlhardware" or "reference"</p></td><td><p>1.2</p></td><td><p>Forces the software to use a particular mode for the underlying graphics accelaration. The reference option is only available if you have the Direct3D SDK installed (very unlikely).</p></td></tr><tr class="even"><td><p>j3d.d3ddriver</p></td><td><p>idx</p></td><td><p>1.2</p></td><td><p>For cards like Voodoo that run fullscreen 3D only. idx is the order DirectX enumerates its driver using DirectDrawEnumerateEx(). This number starts at 1. This will force Java3D to use the driver specified by the user (may fail if the driver is not compatible with display). The driver number and details can be found by using the j3d.debug property. For a typical setup of a 3D only card attach to a graphics card in a single monitor system, use idx=2. This will automatically toggle to fullscreen hardware accelerated mode since if the 3D card support 3D only.</p></td></tr><tr class="odd"><td><p>j3d.debug</p></td><td><p>true or false</p></td><td><p>1.1</p></td><td><p>Prints out startup and running information. Useful for finding out information about the underlying hardware setup.</p></td></tr><tr class="even"><td><p>j3d.fullscreen</p></td><td><p>PREFERRED or REQUIRED or UNNECESSARY</p></td><td><p>1.2</p></td><td><p>Option to force Java3D to run in fullscreen mode for video cards that will only use hardware accelaration when using fullscreen (non-windowed) mode, like the older Voodoo series.</p></td></tr><tr class="odd"><td><p>j3d.vertexbuffer</p></td><td><p>true or false</p></td><td><p>1.2</p></td><td><p>false to turn off the use of vertex buffers (a D3D performance enhancing feature equivalent to OpenGL display lists). Some drivers have implementation problems so it might be worth turning this off if you get crashes. Utility Classes</p></td></tr><tr class="even"><td><p>j3d.audiodevice</p></td><td><p>A quote string containing a class name</p></td><td><p>1.3.2</p></td><td><p>SimpleUniverse utility classes. Takes the name of a concrete subclass of com.sun.j3d.audioengines.AudioEngine3DL2 that will be constructed by Viewer.createAudioDevice(). The default value is null, which means that audio is disabled by default for applications that call Viewer.createAudioDevice(). j3d.configURL Unknown 1.3.1 Found in the ConfiguredUniverse class. Functionality unknown currently.</p></td></tr><tr class="odd"><td><p>j3d.io.ImageCompression</p></td><td><p>"None" or "GZIP" or "JPEG"</p></td><td><p>1.3.1</p></td><td><p>Found in the scenegraph I/O package. Functionality unknown currently.</p></td></tr><tr class="even"><td><p>j3d.stereo</p></td><td><p>PREFERRED or REQUIRED</p></td><td><p>1.1</p></td><td><p>Only used by SimpleUniverse. If you roll your own VirtualUniverse, this property is not used. Controls whether you want Java3D to definitely create stereo mode capable canvases or not</p></td></tr><tr class="odd"><td><p>sun.java2d.d3d</p></td><td><p>true or false</p></td><td><p>??</p></td><td><p>Default true. Enable Direct3D in Java 2D (not Java 3D, actually).</p></td></tr><tr class="even"><td><p>sun.java2d.ddoffscreen</p></td><td><p>true or false</p></td><td><p>??</p></td><td><p>Default true. Enable DirectDraw and Direct3D by Java 2D for off screen images, such as the Swing back buffer (not Java 3D, actually).</p></td></tr><tr class="odd"><td><p>sun.java2d.noddraw</p></td><td><p>true or false</p></td><td><p>??</p></td><td><p>Default false. Completely disable DirectDraw and Direct3D by Java 2D (not Java 3D, actually). This avoids any problems associated with use of these APIs and their respective drivers.</p></td></tr></tbody></table>

Interactive debugging using a shared Terminal session
=====================================================

For users running Linux and MacOSX computers (or on Windows, [Cygwin](http://www.cygwin.com/) with an openssh server), one can use an SSH tunnel for a debugging session shared between a user and a developer. All that is needed is a shared account on a public SSH server.

The user should execute this command:

    ssh -R 2222:127.0.0.1:22 -t $ACCOUNT@$SSHSERVER screen

Once connected, the command

    ssh -p 2222 $LOCALACCOUNT@127.0.0.1

will open a connection back to the local machine.

The developer should then execute this command:

    ssh -t $ACCOUNT@$SSHSERVER 'screen -x'

Since this provides a shared [GNU screen](http://savannah.gnu.org/projects/screen/) session, both the user and the developer can execute commands and see the output. It is even quite common to use the terminal window as sort of a private chat room by typing out what you have to say, ending the line with a {% include key content='Ctrl' %}+{% include key content='C' %} (lest it get executed as a command).

After the debugging party is over, the user can log out securely by hitting {% include key content='Ctrl' %}+{% include key content='D' %} to log out from the local machine (since the user typed in their password in the GNU screen session herself, there is no way for the developer to log back in without the user's explicit consent). Another {% include key content='Ctrl' %}+{% include key content='D' %} will terminate the GNU screen session, and yet another {% include key content='Ctrl' %}+{% include key content='D' %} will log out from the shared account on the SSH server.


