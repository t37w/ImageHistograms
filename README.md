
# Package ImageHistograms.jl
Implement 2d and 3d histograms of images. Support gray and colored. Include support for plotting and some predefined layouts.

## Goal and purpose
Main purpose is to have a view to the color sitribution to check the quality of photos.
And to have a reasonable sized window and not a little stamp.

* 2d histograms
    + gray, red, green, blue
* 3d histogram
    + x-, y-, z-axis : derive from red, green, blue
    + plot using colored dots in RGB24 to get an impression of used color distribution within the image
* identify which Plt package is best for 3d plotting
    + current decison for purpose :  plot ImageHistograms
        - Plots : very good for 2D plotting and customitzed layouts
        - Gnuplot : very good for 3D plotting; in 2D it needs too much typing
        - GR : out of the game. Found no syntax for RGB colored dots

## Used Packages
* julia as programming language
* Images.jl
* TestImages.jl
* Plots.jl : as default for 2D plots; easy to use, very good customizable. Currently not the first choice when doing 3D stuff. Too slow to work with several array having ~140000 elements.
* Gnuplot.jl : Currently too complex for this purpose in 2D.  But best for 3D stuff. Works fine and within seconds for test picture "lena_color_512" which has ~148279 different colors.
* GR.jl : can show 3D. But I have no idea how to plot each marker dot with its own RGB color.

## Package tree layout
* directory src:
    + module "ImageHistogram" : contains the stable source. No cooked plotting
    + module "ImageHistogramTest" : contains the unstable source; use to try something, add new stuff, change existing, ...
        - cooked plotting in 2D using Plots
        - cooked plotting in 3D using Gnuplot
This way, source and runtime can be compared easily.

## Background Information and related URLs [ToDo]
* Color to Gray conversions
* Color schemes
* RGB and friends

## Usage
Copy ImageHistogram.jl and/or ImageHistogramTest.jl into the load path of julia.
Alternatively extend julia's load path to contain the directory the module files are located.

ImageHistogramTest contains functions trying to plot 3D using Plots.

### use of ImageHistogram.jl

```
using Images, TestImages ; reload("ImageHistogram") ; img_col256 = testimage("lena_color_256");

ihR,ihG,ihB = ImageHistogramTest.imhistogramRGB(img_col256);

ihgray = ImageHistogramTest.imhistogramGray(img_col256);

plot(ihgray, color=:lightgray, w=3, line=:sticks)

plot(ihR, line=:red, w=2)

plot_red = plot(ihR, line=:red, w=2); plot_green = plot(ihG, line=:green, w=2); plot_blue = plot(ihB, line=:blue, w=2); plot_Gray = plot(ihgray, line=:lightgray, w=2);
```

### use of ImageHistogramTest.jl

```
using Images, TestImages ; reload("ImageHistogramTest") ; img_col256 = testimage("mandril_color");

ihR,ihG,ihB = ImageHistogramTest.imhistogramRGB(img_col256);

ihgray = ImageHistogramTest.imhistogramGray(img_col256);

ImageHistogramTest.plot_imhi(ihR_cooked=ihR, ihG_cooked=ihG, ihB_cooked=ihB,how=1,bg=1)

ImageHistogramTest.plot_imhi(ihR_cooked=ihR, ihG_cooked=ihG, ihB_cooked=ihB,how=4,bg=1)

ImageHistogramTest.plot_imhi(ihGray_cooked=ihgray,ihR_cooked=ihR, ihG_cooked=ihG, ihB_cooked=ihB,how=3,bg=1)


ImageHistogramTest.plot_imhi_GrayRGB(img_col256)

ImageHistogramTest.plot_imhi_GrayRGB(img_col256, how=3, bg=0)
```

## Plotting 3D: a color cube
### ... with package Gnuplot

With the new Gnuplot.jl verion 0.2.0, doing splot based stuff is easier, and works well.

```
using Gnuplot;

using Images, TestImages ; reload("ImageHistogram") ; img_col256 = testimage("lena_color_512");

redv, greenv,bluev, colv = ImageHistogramTest.imhistogramRGB3d_new2(img_col256);

redv=redv*255.0; greenv=greenv*255.0; bluev=bluev*255.0;

gen_pcv(cv24_a)=(pcv24=zeros(length(cv24_a));for i = 1:endof(cv24_a); pcv24[i]=cv24_a[i].color; end;return pcv24)

@gsp(redv[1:10:end],greenv[1:10:end],bluev[1:10:end],gen_pcv(colv[1:10:end]),"with points pt 13 ps 0.7 lc rgb variable", xrange=(0,255), yrange=(0,255), zrange=(0,255), xlabel="red", ylabel="green", zlabel="blue", "set border -1", "set tics in mirror", "set grid", "set zticks out mirror", "set grid ztics", "set xyplane at 0.0")
```

After hitting the return-key, be patient for a few seconds. Especially if you use the full range of the color arrays. Each has a size of 148279.

Doing the same with "mandril_color" has much less colors and color spots than smooth distribution.

### ... with package Plots

With GR as backend.
Ensure to feed scatter3d() with less than 3000 - 4000 points per color.  It is too slow and it may die.

```
using Plots;

using Images, TestImages ; reload("ImageHistogram") ; img_col256 = testimage("lena_color_512");

redv, greenv,bluev, colv = ImageHistogramTest.imhistogramRGB3d_new2(img_col256);

redv=redv*255.0; greenv=greenv*255.0; bluev=bluev*255.0;

scatter3d(redv[1:50:end], greenv[1:50:end], bluev[1:50:end],color=colv[1:50:end], markerstrokewidth=0, markersize=2,marker=:circle)  # set border of marker symbol to zero with 'markerstrokewidth=0'

scatter3d(redv[1:11:end], greenv[1:11:end], bluev[1:11:end],color=colv[1:11:end], ms=0.1, msw=0, xlims=(0,255),ylims=(0,255),bg=:black, cam=(85,80))  #  set the camera angle to see the plot in this case from above
```
or just call

```
ImageHistogramTest.plot_imhi_3D(img_col256); # with default range_step = 50, i.e. just take each 50s point

ImageHistogramTest.plot_imhi_3D(img_col256, range_step = 30); # 
ImageHistogramTest.plot_imhi_3D(img_col256, range_step = 1); # take each color value; requires much mem; on Windows best after a re-boot
```

to create a draft 3D histogram.

Hints for improvements are welcome via tickets, pull requests, ...

PS:
My coding style uses ';' at line ends by intention. I use copy-n-paste from/to REPL and want quiet exec.



