---
layout: post
title:  "natural sorting (with a hint of regular expressions)"
date:   2012-11-05 00:35:17
categories: introduction
tags: [sorting python regularexpression]
---

###{{page.tags}}###

 When we talk about sorting of strings in informatics we usually mean the lexicographic ordering, i.e. the same ordering that we have in dictionary (a paper one, not the python one). This is formally correct, but have a notorious drawback when we have to present those string to a human.

if we have the following list:

{% highlight python %}
>>> strings = [ 'a1', 'a2', 'a10' ]
{% endhighlight %}

and we sort it, we encounter an unexpected problem:

{% highlight python %}
>>> sorted(string)
['a1', 'a10', 'a2']
{% endhighlight %}

What is happening is that the string `'a10'` is lexicographically before the string `'a2'`.
This is very counterintuitive for our users, and in the long run can sometimes give a little headache even to us.

So, what if we want to sort our objects in a lexicographic order? The basic idea is that we want to order the string dividing the proper string part from the numeric part.
if we know how our strings are composed, as in the preceding example, we can simply tamper with the sorted key parameter. This parameter allow us to use a derivated object to order our list instead of the original one. in our case what we need is a tuple with a string part and a numeric part:

{% highlight python %}
>>> splitter = lambda s: ( s[0],int(s[1:]))
>>> sorted(strings, key=splitter)
['a1', 'a2', 'a10']
{% endhighlight %}

Ok, this works, but is far from general. the basic idea is good, but we need a way to split a string into his numerical parts, no matter where and how many of them there are!
One method is to use the itertools module (yes, my favourite standard library module), the groupby function, to be exact. 
This function run over an iterable and group it's elements based on a lambda given by the user. In our case we need the isdigit function of the string to identify which pieces are numbers and which aren't. The solution is a simple one-liner

{% highlight python %}
>>> from itertools import  groupby
>>> string = 'aaa111aaa111aaa111aaa111'
>>> [ (a,''.join(b)) for a,b in groupby(string, lambda s: s.isdigit())] 
[(False, 'aaa'), 
(True, '111'), 
 (False, 'aaa'), 
(True, '111'), 
 (False, 'aaa'), 
 (True, '111'), 
 (False, 'aaa'), 
 (True, '111')]
{% endhighlight %}

Where the first value of each tuple is the results of the splitting and the second is the matched text. This is already a solution to our problem, but is rough around the edges. To cite one, it read wrongly the dot inside a floating point number, and it's not easy to insert any knowledge of the structure of our string.

To solve the first problem we can fuse together  the triplets number-dot-number, while the other is quite hard to implement.

{% highlight python %}
>>> string = 'aaa111aaa1.11aaa111aaa111'
>>> res = [ (a,''.join(b)) for a,b in groupby(string, lambda s: s.isdigit())]
>>> res2 = []
>>> idx = 0>>> while idx<len(res):>>>     if idx<len(res)-2:>>>         i,j,k = res[idx],res[idx+1],res[idx+2]>>>     else: >>>         i=None>>>     if i and i[0] and not j[0] and k[0] and j[1]=='.':>>>         res2.append((True,"".join([i[1],j[1],k[1]])))>>>         idx+=3>>>     else:>>>         res2.append(res[idx])>>>         idx+=1>>> res2
[(False, 'aaa'),
 (True, '111'),
 (False, 'aaa'),
 (True, '1.11'),
 (False, 'aaa'),
 (True, '111'),
 (False, 'aaa'),
 (True, '111')]
{% endhighlight %}

Ok, this works, but is ugly as hell. We need to find a better way. To do this, we need to borrow the power of the regular expressions. The regular expressions (or regex, for short) are a standard way to analyze a string to obtain pieces of it, using a road tested state machine.

To use the regex we need to import the re module, using the findall function to search a string for the given pattern. The pattern is described with another string with a special syntax, but we will come to that later.

Let's see some basic usage of the re module. We need to feed the findall function with a pattern string, in this case the word dog, to search into the given string. The r before the pattern is to indicate that it is a regex string, and will simplify how to write the patters

{% highlight python %}
>>> import re 
>>> string = "i have two dogs, the first one is called fido, while the second dog is rex"
>>> re.findall(r'dog', string) 
['dog', 'dog']
{% endhighlight %}

So, the re module reply to us that it has found two occurences of the word dog. Note that the resulting list only contains the exact match: so even if the first word was plural (dogs), the matched string is just the `'dog'` component.

If one of the words starts with a capital letter, the search will find only one of them. If we want to find both the cases we can use the square brackets to indicate that the strings inside are equivalent. So our new code look like this

{% highlight python %}
>>> string = "i have two Dogs, the first one is called fido, while the second dog is rex"
>>> re.findall(r'[Dd]og', string)  
['Dog', 'dog']
{% endhighlight %}

 Ok, next step, we want to include the s of the plural if found. To obtain this, we have to say that the last s is optional: if is present, include it, but don't worry if it's missing. This is done with a question mark following the subject of interest, the letter s.

{% highlight python %}
>>> re.findall(r'[Dd]ogs?', string)  
['Dogs', 'dog']
{% endhighlight %}

Ok, for now I will stop, you can find a huge amount of material online that explain how to use them. Prepare to suffer a little bit, understanding the regex has quite a learning curve.
The pattern to separate the any number of string block from number is the following:

{% highlight python %}
r'[0-9]+|[^0-9]+'
{% endhighlight %}

It say that you can alternatively (the `|` operator) match one or more (the `+` operator) groups of digits (`[0-9]`) or something that is not a digit (`[^0-9]`).

Let's put it to the test:

{% highlight python %}
>>> test = [ 'aaa123bbb.tex', '123aaa345.txt' ]
>>> for string in test:
>>>     res = re.findall(r'[0-9]+|[^0-9]+', string)
>>>     print string,res
aaa123bbb.tex ['aaa', '123', 'bbb.tex'] 
123aaa345.txt ['123', 'aaa', '345', '.txt']
{% endhighlight %}

It's not perfect around the edges, but with a little work it can be perfect. What we can do is to specify that a dot that interrupt a number is part of that number, while one that is not between numbers should be on it's own

{% highlight python %}
>>> test = [ 'aaa123bbb.tex', '123aaa345.txt', "aaa3.14bbb.jpg" ]
>>> for string in test:
>>>     res = re.findall(r'[0-9]+\.?[0-9]+]?|[^.0-9]+|.', string)
>>>     print string,res 
aaa123bbb.tex ['aaa', '123', 'bbb', '.', 'tex'] 
123aaa345.txt ['123', 'aaa', '345', '.', 'txt'] 
aaa3.14bbb.jpg ['aaa', '3.14', 'bbb', '.', 'jpg']
{% endhighlight %}

Dig deeper in the regex module... a lot of power is in it! 