---
layout: post
title:  "Using an IPython Notebook as a module"
date:   2012-10-26 18:17:00
categories: introduction
tags: [ipython python module]
---

###{{page.tags}}###


First of all, if you never worked with the ipython notebook, put this post on pause, go to the [ipython home](http://www.ipython.org), install everything you find, play with it and fall in love.
I'll be waiting, don't worry.

Ok, so you love the ipython notebook, work everyday with it, and cry everytime you have to go back to the usual shell. The only problem is writing a module in it is very uncomfortable: each time you made a modification, you have to save it as a python script, and if you work over a network, you have to send it to the same directory, fiddle with the permissions and so on.

The secret is that you can start the ipython notebook server with the option `--script`. This option tell the server to save a copy of the notebook in the script version (the .py file, to be clear) any time you save the notebook.My tipical line to execute the ipython notebook became:

    ipython notebook ./my_notebook_folder --pylab=inline --no-browser --script

So, every time you edit a notebook, you will have the corresponding module to import from other notebook, closing the circle. Using a notebook to write your own module has a lot of advantage, in my opinion, first of all the possibility to explain with a lot of well formatted text how your library works, accompaining it with link to pages ( use `<page to be linked>` ) or multimedia objects ( use `[text](link)`) and even latex formulas, really a killer feature when you have to explain scientific code.

The only limitation is that, as far as I know, the script must be traslate into pure python, so no ipython magic or cell magic. It's not that bad, but would have been a real game changer (anybody thinking "seamless cython integration"?).

So you can declare your classes and functions as usual, and put the test code in a block for execution in main:

{% highlight python %}
if __name__ == '__main__':
    test code in here
{% endhighlight %}

as you would do with a normal script. It will be executed normally when you run the notebook (as it has the `__name__` set to `__main__` by default), but will be skipped in the import phase.
Remember to use the `__all__` parameter to avoid useless name import on

{% highlight python %}
from mylibrary import *
{% endhighlight %}

...no wait, you should never do that. Forget the `__all__`.

Last thing, remember to insert documentation for your code! in a matter of few days you will not remember what each parameter does, so write it down.
If you put a string on the first cell of the notebook, it will be seen as the standard documentation of the whole module.

Last of all, one of the way I prefer to implement documentation and testing in one shot is the use of the DocTest module, which scan all the documentation present in the module, search for documentation lines that looks like shell code, execute it and confront it with the result that you have put in the documentation.
If they are not equal it will complain that a test is failed.

This is obviously not the way to document production code, you should use something like unittest, but is very practical and, honestly, i found that it compels you to write both documentation with examples and useful testing at the same time, both very tedious activity and any help to do them is welcome.

The problem is that doctesting a notebook will fail in a spectacular way, due to its dynamic nature. To help with that, I wrote a function that analyze an object (a class, an instance, a function, a module...whatever) and test each docstring it find in it, reporting a dictionary of the results for each method.

Here it is:

{% highlight python %}
def test(obj,verbose=False, globs = globals()):
    """
    test the docstring of an object, a function or a module
    if verbose is set to True, it will report the result
    of each test, otherwise it will report only 
    the failed ones
    """
    test = doctest.DocTestFinder().find(obj, globs=globs)
    runner = doctest.DocTestRunner(verbose=verbose)
    results = {}
    name = ''
    def out(s):
        results[name].append(s)
    for t in test:
        name = t.name
        results[name]=[]
        runner.run(t,out=out)
    if not verbose:
        rimuovi = [ k for k,v in results.iteritems() if not len(v) ]
        for k in rimuovi:
            del results[k]
    return results
{% endhighlight %}

How does it work? it's actually pretty simple, once you understand how the doctest module works. First of all you take the object and parse it with the class DocTestFinder:

{% highlight python %}
test = doctest.DocTestFinder().find(obj, globs=globs)
{% endhighlight %}

This class will return a list of Test object to be run. To run these tests you use the class  DocTestRunner, then create a dictionary to store the results.

{% highlight python %}
runner = doctest.DocTestRunner(verbose=verbose)
results = {}
{% endhighlight %}

Now the runner will take each Test in our list and execute it, than confront it with the expected result. It would normally print it to terminal, but we can override this behavior giving it a parameter out, which is a function that can manipulate the result. In our case it put the result of the test in the dictionary under the name of the tested object.

{% highlight python %}
def out(s):
    results[name].append(s)
for t in test:
    name = t.name
    results[name]=[]
    runner.run(t,out=out)
{% endhighlight %}

In the end, if you select a non verbose result (the default argument) it will scan the resulting dictionary and remove all the test that didn't return error.
Than the resulting dictionary is returned, and if your documentation is up to date, you will obtain a void dictionary.

To print the results in a friendly version, I use this ancillary function, that simply scan and print keys and values of the dictionary (doing nothing if it is empty):

{% highlight python %}
def verify(obj):
    res = test(obj)
    if not res:
	return
    for k,v in res.iteritems():
	print "--------------"
	print k
	for v1 in v:
	    print v1
{% endhighlight %}

Just another short tip: if you need to test a random function, don't worry. just set the seed to a fixed value at the beginning of the test, and the program will execute the same steps with the same random number each time. In numpy this is done with:

{% highlight python %}
np.random.seed(0)
{% endhighlight %}

That's all folks! Have fun, and see you soon!