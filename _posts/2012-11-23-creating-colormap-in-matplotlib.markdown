---
layout: post
title:  "Creating a colormap in matplotlib"
date:   2012-11-23 02:36:00
categories: introduction
tags: [python colormap matplotlib]
---

Matplotlib, as I said before, is quite an amazing graphics library, and can do some power heavy-lifting in data visualization, as long as you lose some time to understand how it works. Usually it's quite intuitive, but one field where it is capable of giving huge headhace is the generation of personalized colormaps.

This page ([http://matplotlib.org/examples/api/colorbar_only.html](http://matplotlib.org/examples/api/colorbar_only.html)) of the matplotlib manual give some direction, but it's not really useful. What we usually want is to create a new, smooth colormap with our colors of choice.
To do that the only solution is the `matplotlib.colors.LinearSegmentedColormap` class...which is quite a pain to use. Actually there is a very useful function that avoid this pain, but I will tell the secret after we see the basic behavior.

The main idea of the `LinearSegmentedColormap` is that for each color (red, green and blue) we divide the colormap in intervals and explain to the colormap two colors to interpolate in between. This is the code to create the simplest colormap, a grayscale:

{% highlight python %}
mycm = mpl.colors.LinearSegmentedColormap('mycm',
{'red':((0., 0., 0.), (1., 1., 1.)),
 'green':((0., 0., 0.), (1., 1., 1.)),
 'blue':((0., 0., 0.), (1., 1., 1.)),
},256)
{% endhighlight %}

First of all there is the name of the colormap, the last is the number of point of the interpolation and the middle section is the painful one.
The colormap is described for each color by a sequence of three numbers: the first one is the position in the colormap, and can go from 0 to 1, monotolically. The second and the third numbers represents the value of the color before and after the selected position.
This basic example is composed of two point for each color, 0 and 1, and it say that at those position the color is absent (0) or present (1)

To understand better, we can use a colormap that go from red 0 to 0.25 in the first half, then just after the half switch to 0.75 and go to 1 as the colormap go to 1

{% highlight python %}
import matplotlib as mpl
lscm = mpl.colors.LinearSegmentedColormap
mycm = lscm('mygray',
{'red':((0., 0., 0.), (0.5, 0.25, 0.75), (1., 1., 1.)),
 'green':((0., 0., 0.), (1., 0., 0.)),
 'blue':((0., 0., 0.), (1., 0., 0.)),
},256)
{% endhighlight %}

Ok, this is really powerful, but is clearly an overshot in most cases! The matplotlib developers realized this, but for some reason didnt create a whole new class clearly in the module, deciding to create a method of the LinearSegmentedColormap instead, called `from_list`.
This is the magic cure that we need: to make a simple colormap that goes from red to black to blue, we just need this.

{% highlight python %}
mycm = lscm.from_list('mycm',['r','k','b'])
{% endhighlight %}

of course you can mix named colors with tuple of rgb, at your hearth content!

{% highlight python %}
mycm = lscm.from_list('mycm',['pink','k',(0.5,0.5,0.95)])
{% endhighlight %}

Ok, now we have our wonderful colormap...but if we have some nan value in our data, everything is going bad, and value is represented in white, out of our control. Don't worry, as what we need is just to set the color to use for the nan values (actually, for the masked ones) with the function set_bad. in this case we put it to green:

{% highlight python %}
#the colormap
mycm = mpl.colors.LinearSegmentedColormap.from_list('mycm',['r','k','b'])
mycm.set_bad('g')
#the corrupted data
a = rand(10,10)
a[5,5] = np.nan
#the image with a nice green spot
matshow(a,cmap = mycm)
{% endhighlight %}

Note: use matshow when you think that nan values can be present, as pcolor doesn't get along well with them and imshow keep the white color. 