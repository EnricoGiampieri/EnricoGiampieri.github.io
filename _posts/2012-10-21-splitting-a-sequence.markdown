---
layout: post
title:  "Splitting a sequence"
date:   2012-10-21 04:05:17
categories: introduction
tags: [sequence python]
---

###{{page.tags}}###

 
I would like to start with one common exercise, that a lot of people wrote about, each one proposing its own version. 

Let say that we have a sequence of objects, and we want to split this sequence in equally sized chunks of given size. The idea is the following:

{% highlight python %}
>>> seq = range(12)
>>> split(seq, 3)
[[0, 1, 2], [3, 4, 5], [6, 7, 8], [9, 10, 11]]
{% endhighlight %}

It looks simple enough, doesn't it? There are probably one hundred different solution to this problem. Anyone has its own preferred version. I will try to show the most common, curious or instructive ones. 

Coming from languages like C or Java would be spontaneous to write code like this:

{% highlight python %}
L = len(seq)
size = 3 
solution = list()for i in range(L):
     if i%size:  
         solution[-1].append(seq[i])
     else:
         solution.append([seq[i]])
{% endhighlight %}

Ok, this gives me the correct answer, but this isn't python code. This is C written in python.

a better solution is the following:

{% highlight python %}
[ seq[size*i:size*i+size] for i in range(len(seq)//size+1) ]
{% endhighlight %}

It does the same work, but is only one line of code, and, after you are used to the list comprehension magic, it looks a lot more readable. But it still suck, if I should be honest.
It calculate the number of pieces the list will be divided into (len(seq)//size), add one to keep the last fragment, the iterate on the obtained indices. Not so elegant.

A more pythonic code can be written using the list comprehension, and is simply one line of code:

{% highlight python %}
[ seq[i:i+size] for i in range(0,len(seq),size) ]
{% endhighlight %}

What this piece of code does is to iterate over the indices of the list, starting from 0 and increasing of "size" step at the time. for each step it takes the element of the list from the given index to the following "size" elements

This will do the trick in the exact same way as the initial code, but is way more simple to write and to read.

But this is not the end of the story. I would like to show you a slightly more esoteric way to slit the sequence:

{% highlight python %}
zip(*([iter(seq)] * 3))
{% endhighlight %}

This is a trick that use several effect at the same time. First an iterator is generated from the list. than this iterator is placed inside a list, and the list is triplicated. Each element of the list is now a reference to the same iterator, as you can see writing

{% highlight python %}
>>> print [iter(seq)]*3
[<listiterator object at 0x3900610>, <listiterator object at 0x3900610>, <listiterator object at 0x3900610>]
{% endhighlight %}

Where the address of the iterator will change each time you lanch the program. This triplicated access to the iterator means that every time the zip function call the next function on one o them to obtain an element, each one will be increased. The result is a splitted sequence. Due to the behavior of the zip function, it will trim the sequence to the shorter one, so if the sequence is not a multiple of the given size, it will be trimmed.

{% highlight python %}
>>> seq = range(11)
>>> zip(*([iter(seq)] * 3))
[(0, 1, 2), (3, 4, 5), (6, 7, 8)]
{% endhighlight %}

This can be avoided using the function izip_longest of the package itertools, which extend the shortest sequence with a serie of None. It does return a generator that can be easily converted into a list:

{% highlight python %}
>>> from itertools import izip_longest as lzip
>>> list(lzip(*([iter(seq)] * 3)))  
[(0, 1, 2), (3, 4, 5), (6, 7, 8), (9, 10, None)]
{% endhighlight %}

You can specify a filling value for the izip_longest, if the None could lead to problems:

{% highlight python %}
>>> from itertools import izip_longest as lzip
>>> list(lzip(*([iter(seq)] * 3), fillvalue = -1))  
[(0, 1, 2), (3, 4, 5), (6, 7, 8), (9, 10, -1)]
{% endhighlight %}

A similar effect can be obtained with the function map,  which by default extend the sequence to the longest element.

{% highlight python %}
>>> map( lambda *s:s, [iter(seq)]*3 )
[(0, 1, 2), (3, 4, 5), (6, 7, 8), (9, 10, None)]
{% endhighlight %}

 with python 2.7 you can also use None instead of the identity lambda *s:s, but it is always a good habit to think of the compatibility with python 3, where it's possible.
To obtain the initial  result of having a shorter last sequence, one can explicitly trim the None:
 
{% highlight python %}
>>> map( lambda *s: [ r for r in s if not r is None ], [iter(seq)]*3 )
[(0, 1, 2), (3, 4, 5), (6, 7, 8), (9, 10)]
{% endhighlight %}

Going back to the itertools module, one can think also to the groupby function. This function split an iterator into chuncks that respect the same condition, and when they change it start another chunk. So if we use an integer division, we can split the array into equally sized parts:

{% highlight python %}
>>> from itertools import groupby
>>> for key_value, split_generator in groupby(seq, lambda s: s//3):
....    print  key_value, list(split_generator)
0 [0, 1, 2] 
1 [3, 4, 5] 
2 [6, 7, 8] 
3 [9, 10]
{% endhighlight %}

This version has just one problem: it works only for the sequence of number given by range. to adapt it to a more general case, we need to use the enumerate function to obtain the golden sequence, filter it and then keep only the interesting data:

{% highlight python %}
from itertools import groupby
seq = 'abcdefghilm'
for key, split_gen in groupby(enumerate(seq), lambda s: s[0]//3):
    print  key, list(i[1] for i in split_gen) 
0 ['a', 'b', 'c'] 
1 ['d', 'e', 'f'] 
2 ['g', 'h', 'i'] 
3 ['l', 'm']
{% endhighlight %}

We will meet again this function when we will talk about natural ordering.

Last, but not least, using the numpy module, one obtain the function array_split, that allow to divide the sequence in n given partition of variable size. In this case, to obtain the same splitting, you should use 4 division. It also return not a list of lists, but a list of numpy arrays.

{% highlight python %}
>>> from numpy import array_split as split
>>> split(range(11), 4)
[array([0, 1, 2]), array([3, 4, 5]), array([6, 7, 8]), array([ 9, 10])]
{% endhighlight %}

This function is quite powerful, as it can also split around specific points or in a given axis for multidimensional arrays. Also matplotlib as a similar function, called pieces, under the submodule matplotlib.cbook, which contains several small recipes from the matplotlib cookbook (http://matplotlib.org/api/cbook_api.html). If you use matplotlib give it a look, it is not very well documented, but contains a lot of useful objects.

In conclusion, I would like to gave you a puzzle that i found on StackOverFlow. It is a piece of BAD PYTHON, very bad indeed, but is curious how it get the job done.

{% highlight python %}
>>> f = lambda x, n, acc=[]: f(x[n:], n, acc+[(x[:n])]) if x else acc
>>> f(range(11), 3)
[[0, 1, 2], [3, 4, 5], [6, 7, 8], [9, 10]]
{% endhighlight %}

I have to admit that my first reaction was "wait...what happened?!?". I'm actually still confused on how someone could have thought of something like that. sure as hell no one would be able to debug it if something goes in the bad direction. This is a recursive function, that take a list, extract a first chunk and add it to the accumulator value acc, then pass to itself the shorter list and the accumulator, until the list is empty and it return the accumulator. We can visualize what is happening by rewriting the function in a more explicit way and by printing the intermediate results:

{% highlight python %}
>>> def g(n, x, acc=[]):
...     print (n,x,acc)
...     #recursive until exausted
...     if x:
...         # launch the same function on a shorter
...         # version of the list with the
...         # accumulated list of lists  
...         return g(n, x[n:], acc+[(x[:n])])
...     #when exausted return the result
...     else: 
...         return acc
>>> s = g(3, range(11))
>>> print '\n',s
(3, [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10], []) 
(3, [3, 4, 5, 6, 7, 8, 9, 10], [[0, 1, 2]]) 
(3, [6, 7, 8, 9, 10], [[0, 1, 2], [3, 4, 5]]) 
(3, [9, 10], [[0, 1, 2], [3, 4, 5], [6, 7, 8]]) 
(3, [], [[0, 1, 2], [3, 4, 5], [6, 7, 8], [9, 10]])

[[0, 1, 2], [3, 4, 5], [6, 7, 8], [9, 10]]
{% endhighlight %}

Yes, it is close to black magic, but it works. But remember kids: recursion is bad, unless it is the only sensible solution to your problem (and 99% of the time, it is not).

Of course these methods are not the only that one can think, but should cover a wide range of necessity, teaching us something in between.

See you next time! 
