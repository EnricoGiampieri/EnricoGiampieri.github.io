---
layout: post
title:  "Blogging with the ipython netbook and the github/nbviewer combo"
date:   2013-01-22 23:32:00
categories: introduction
tags: [python ipython git]
--- 

As you can see I've just changed the whole aspect of the blog.
This is a response to a couple of design need. The lesser one is that all the plots that I will create will be with a white background, and against a dark background it result in a pain in the eye. The greater one is that Blogger suck for posting code and images. I spend half of my time to check the font and color of the code, and every image require saving it to disk. I can't even show any more complicated code as the results should be reformatted by hand. And, to be honest, I always dreamed of simply blogging my ipython notebook scripts.

So, thank to [Brian E. Granger](http://brianegranger.com/), who wrote a simple method to [post an ipython notebook as a frame](http://brianegranger.com/?p=215) in the post, I can have the cake and eat it too!
The method is simple as adding an iframe tag (with the correct dimension put by hand, but that's a minor flaw) in the HTML code of the post, and voilà !

`<iframe src="http://nbviewer.ipython.org/3835181/" width="800" height="1500"></iframe>`

Leveraging the magic of the [nbviewer](http://nbviewer.ipython.org/) server (that is an amazing service, by the way) I can now write a wonderfully formatted notebook, with code and formulas and plots and everything, and just hand it to you. What I'm going to do is set a [git repository](https://github.com/), create my notebook in there and link them with nbviewer in here. By the way, the notebook that it's linked it's the wonderful [XKCD plot style](http://jakevdp.github.com/blog/2012/10/07/xkcd-style-plots-in-matplotlib/) created by [Jake Vanderplas](http://jakevdp.github.com/), a great blogger and python developer

I will try to explain how I'm going to set up and manage the repository.
First of all I create a new repository called blogger_notebook
Having set up a github account (that is really easy), the next step is to create the directory that will host my material.

{% highlight bash %}
mkdir blogger_code
cd blogger_code/
{% endhighlight %}

GitHub give some useful information on how to create a new repository. First of all, we create it by writing

{% highlight bash %}
git init
{% endhighlight %}

this tell to git that in this directory we have a git repository and that it should keep the version control backup of the data. Now I tell him the online repository location:

{% highlight bash %}
git remote add origin https://github.com/EnricoGiampieri/blogger_notebook.git 
{% endhighlight %}

Ok, we are close to the goal. I copy the notebook that will be my next post, basemap.ipynb. Now I need to tell git to follow it

{% highlight bash %}
git add basemap.ipynb
{% endhighlight %}

now everytime I make a modification to this file that I want to remember I can save it with the commit command. I also need to add a description of the modification done

{% highlight bash %}
git commit basemap.ipynb -m "creation of an example of basemap usage"
{% endhighlight %}

lastly, to keep up the repository online, I should put it into the GitHub repository. This will ask for my username and password and will upload all the modification to the online repository.

{% highlight bash %}
git push -u origin master
{% endhighlight %}

you can see the results here:

{% highlight bash %}
https://github.com/EnricoGiampieri/blogger_notebook
{% endhighlight %}

Now, the last step is to create a nbviewer link to the notebook. You should take the link to the raw file (you can obtain it going into the file and search for the RAW button) and give it to the nbviewer main page. it will give you a nice link to the notebook with all it's content:

[http://nbviewer.ipython.org/urls/raw.github.com/EnricoGiampieri/blogger_notebook/master/basemap.ipynb](http://nbviewer.ipython.org/urls/raw.github.com/EnricoGiampieri/blogger_notebook/master/basemap.ipynb)

Obviously this is barely scratching the surface of the (super)power of git, but there are tons of manuals online that explain it better than what I could ever do. This was just a step by step guide to how to setup this "delayed blogging" method.