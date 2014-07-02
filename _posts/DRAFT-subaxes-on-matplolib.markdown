---
layout: post
title:  "sub axes on matplotlib"
date:   DRAFT
categories: introduction
tags: [python]
--- 

{% highlight python %}
import matplotlib
def axes_subaxes(bounds,ax=None,**kwargs):
    if ax is None:
        ax=pylab.gca()
    fig = ax.figure
    Bbox = matplotlib.transforms.Bbox.from_bounds(*bounds)
    trans = ax.transAxes + fig.transFigure.inverted()
    new_bounds = matplotlib.transforms.TransformedBbox(Bbox, trans).bounds
    axins = fig.add_axes(new_bounds,**kwargs)
    return axins

fig,ax = pylab.subplots(1,figsize=(4,4))
inax = axes_subaxes([0.2, 0.45, .5, .5],ax,sharex=ax,sharey=ax)
ax.plot([1,2,3],[1,4,9])
inax.plot([1,2,3],[1,8,27]) 
{% endhighlight %}