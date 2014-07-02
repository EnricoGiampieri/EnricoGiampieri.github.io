---
layout: post
title:  "Signature of a Function"
date:   DRAFT
categories: introduction
tags: [python]
--- 

The first thing we do when we create a function in python is to define it's signature, i.e. the list of arguments that the function will take.

This part is vital for the fucntion and is often overlooked in the creation.
Pyhton gives us a lot of method to define the signature with a lot of power and several gotchas.

{% highlight python %}
def f():   #function definition and signature
    pass   #function body
{% endhighlight %}

The first point that need to be understood is the steps of a function creation in python. The first time the interpreter receive a function, it will execute the definition (with the signature) and store the function body to be executed in a second time. Not understanding this point is the source of countless problem with the famous "mutable argument" gotcha.

This is why in the function body you can use a reference to a variable that doesn't exist yet, but not so in the signature.

{% highlight python %}
def f():
    print a
a=4
f(a)
# print 4

def f(number = b):
    print number
#gives error, as b is not defined
{% endhighlight %}