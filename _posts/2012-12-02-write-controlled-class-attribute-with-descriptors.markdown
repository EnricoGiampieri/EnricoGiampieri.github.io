---
layout: post
title:  "write controlled class attribute with descriptors"
date:   2012-12-02 15:59:00
categories: introduction
tags: [python attributes descriptors]
---

Few days ago I had the occasion to play around with the descriptor syntax of python. The normal question of "What is a descriptor" is always replied with a huge wall of text, but in reality are quite a simple concept: they are a generalization of the properties.

For those not familiar with the concept of properties, they are a trick to call function with the same syntax of an attribute. if `prop` is a property, you can write this assignment as a normal attribute:

{% highlight python %}
A.prop = value
{% endhighlight %}

But the property will allow you to perfom check and similar on the value before the real assignment.
The basic syntax start from a normal get/set syntax (never use them unless you plan to work with a property!), but the you add a new element to the class that put togheter this two function under the name x:

{% highlight python %}
class A(object):
    def get_x(self):
        return self.hidden_x
    def set_x(self, value):
        print "set the value of x to",value
        self.hidden_x = value
    x = property(get_x,set_x)
{% endhighlight %}

You can now use it as a normal class attribute, but when you assign a value to it, it will react with the setter function.

{% highlight python %}
a = A()
a.x = 5
#set the value of x to 5
print a.x
{% endhighlight %}

This open a new world of possible interaction between the class and the user, with a very simple syntax. The only limit is that any extra information ha to be stored in the class itself, while sometimes can be useful to keep it separated. It can also become very verbose, which is something that is frown upon when programming in python (Python is not Java, remember).

If we have to create several attribute which behave in a similar way, repeting the same code for the property can be quite an hassle. that's where the descriptor start to became precious (yes, they can do a lot more, but I don't have great requirements).

The descriptor is a class which implements the method `__get__` and, optionally, the method `__set__` and `__delete__`. These are the methods that will be called when you try to use the attribute created with these properties.

Let's see a basic implementation of a constant attribute, i.e. and attribute that is fixed in the class and cannot be modified. To to this we need to implement the `__get__` method to return the value, and the `__set__` method to raise an error if one try to modify it. To avoid possible modification, the actual value is stored inside the Descriptor itself (via the self reference). To interact with the object that possess the Descriptor we can use the instance reference

{% highlight python %}
class ConstantAttr(object):
    def __init__(self,value,name=""):
        self.value=value
        self.name=name
    def __get__(self, instance, type):
        return self.value
    def __set__(self,instance,value):
        raise AttributeError('the attribute {} cannot be written'.format(self.name))
{% endhighlight %}

We can now create a class that use this descriptor. We pass the name of the attribute to the `__init__` otherwise the Descriptor would have no information on which name the class has registered it under.

{% highlight python %}
class A(object):
    c = ConstantAttr(10,'c')
{% endhighlight %}

Using an instance of the class we can see that the value if printed correctly at 10, but if we try to modify it, we obtain an exception.

{% highlight python %}
a = A()
print a.c
#10
a.c = 5
#raise AttributeError: the attribute c cannot be written
{% endhighlight %}

Now we can create as many constant attributes as we need with almost no code duplication at all! That's a good start.

The reason I started playing around with the descriptor was a little more complicated. I needed a set of attributes to have a validity test of the inserted value, raising error if the test wasn't correct. You can performa this with properties, but you can't use raise statement in a lambda, forcing you to write a lot of different setters, polluting the class source code and `__dict__` with a lot of function. To remove the pollution from the dict you can always delete the function you used to create the property

{% highlight python %}
class A(object):
    def get_x(self):
        return self.hidden_x
    def set_x(self, value):
        #do the check
        self.hidden_x = value
    x = property(get_x,set_x)
    del get_x,set_x
{% endhighlight %}

This could work, but you still have 7 or more lines to define something that is no more than a lambda with a message error attached.

So, here come the Descriptor. To keep the pollution to the minimum, I store all the protected values in a intern dictionary called props.
What this code does is to take a test function for testing if the given value is acceptable, then set it if it's correct or raise the given error if it's not.

{% highlight python %}
class CheckAttr(object):
    """create a class attribute which only check and transform an attribute on setting its value"""
    def __init__(self, name, default, test=lambda i,v: True, error='test failed', converter = lambda v:v):
        self.name = name
        self.test = test
        self.error = error
        self.default = default
        self.conv = converter
       
    def checkprops(self,instance):
        try:
            instance.props
        except AttributeError:
            instance.props={}
           
    def __get__(self, instance, type):
        self.checkprops(instance)
        return instance.props.setdefault(self.name,self.default)
       
    def __set__(self, instance, value):
        val =  self.conv(value)
        self.checkprops(instance)
        if not self.test(instance, val):
            raise ValueError(self.error)
        instance.props[self.name]=val
{% endhighlight %}

Now it's time to test it on a simple real case. We want to describe a rectangle, so we have two dimensions, height and width, and we need an attribute to return the area of the rectangle itself.

{% highlight python %}
class Rect(object):
    h = CheckAttr('h', 0.0, lambda i,v: v>= 0.0, 'height must be greater than 0', lambda v:float(v))
    w = CheckAttr('w', 0.0, lambda i,v: v>= 0.0, 'width must be greater than 0', lambda v:float(v))
    area = property(lambda self: self.h*self.w)
    def __init__(self,h=0.0,w=0.0):
        self.w = w
        self.h = h
{% endhighlight %}

Annnnnd...That's it. With this Descriptor code we imposed the condition that bot the width and height should be greater than zero and obtained an attribute area which return the value without giving the possibility of setting it, in only 7 lines of code. Talk about synthesis!

To end with something more difficult let's try to describe the Triangle, which condition use also the values of the other side. This is not a 100% safe version and not performance fine-tuned, but I guess is simple enough to be used:

{% highlight python %}
infty = float('inf')
from math import sqrt
class Triangle(object):
    l1 = CheckAttr('l1', infty, lambda i,v: i.l2+i.l3 > v >= 0.0, 'side 1 must be greater than 0 and smaller than the sum of l2 and l3', lambda v:float(v))
    l2 = CheckAttr('l2', infty, lambda i,v: i.l1+i.l3 > v >= 0.0, 'side 2 must be greater than 0 and smaller than the sum of l1 and l3', lambda v:float(v))
    l3 = CheckAttr('l3', infty, lambda i,v: i.l2+i.l1 > v >= 0.0, 'side 3 must be greater than 0 and smaller than the sum of l2 and l1', lambda v:float(v))
    p = property(lambda self: self.l1+self.l2+self.l3)
    a = property(lambda self: sqrt( self.p/2 * (self.p/2-self.l1) * (self.p/2-self.l2) * (self.p/2-self.l3) ) )
    def __init__(self,l1=0.0,l2=0.0,l3=0.0):
        self.l1 = l1
        self.l2 = l2
        self.l3 = l3
    def __str__(self):
        return "Triangle({t.l1}, {t.l2}, {t.l3})".format(t=self)
    __repr__ = __str__

t = Triangle(1,2,1.5)
print t
# Triangle(1.0, 2.0, 1.5)
print t.p
# 4.5
print t.a
# 0.726184377414
t.l3=5
# ValueError: side 3 must be greater than 0 and smaller than the sum of l2 and l1
{% endhighlight %}
            
###__EDIT:__

I forgot two interesting details for the implementation of the descriptors. The first one address the issue of accessing the descriptor from the class rather than from an instance. I would expect to obtain a reference to the Descriptor instance, but I got the default value. What I should have done was to check if the instance was None (meaning access from the class) and return the descriptor itself:

{% highlight python %}
def __get__(self, instance, type):
    #this allow me to access the descriptor instance
    if not instance:
        return self
    return instance.value
{% endhighlight %}

The second bit is about the documentation. If I write the documentation of the Descriptor, I lose the opportunity to obtain a documentation for each instance, that is one of the cool feature of the property object. This can be done in a simple way...using a property ;)


{% highlight python %}
class DocDescriptor(object):
    def __init__(self,doc):
        self.doc=doc
    __doc__ = property(lambda self: self.doc) 
    #other methods to follow
{% endhighlight %}

This allow to write code like this one:

{% highlight python %}
class A(object):
    x = DocDescriptor("this is the documentation of x")
    y = DocDescriptor("and this is for y")  

help(A.x)
# this is the documentation of x
{% endhighlight %}
