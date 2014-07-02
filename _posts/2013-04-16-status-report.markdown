---
layout: post
title:  "Status reports"
date:   2013-04-16 14:39:00
categories: introduction
tags: [python]
--- 

First of all, I'm finally moving to python 3.3! as I pointed out in my previous post, [Moving to python 3k for numerical computation]({{site.baseurl}}/introduction/2012/12/10/moving-to-python3-for-numerical-computation/index.html), the situation is still from perfect, but in the last months it got better: still no good response from mayavi and tables (and that's a real shame), but both scikits.learn and biopython (even if only installing from sources) got the golden status of py3k ready. Given that I rarely use tables or mayavi, almost 100% of my forkflow is ready for the transition. The reason for the transition is quite simple: I like new shiny toys :) aside from that, in the last year I've been bitten frequently by the unicode management in python 2.7, and several times I've desired strongly to move to a better behaved language like python 3. The python team made a great job, and the result is a language that has a cleaner and more logical structure. Why should I stick to the less than optimal python 2? I don't particularly love giving myself troubles for free, and I still have very few strings attached so that my transition can be carefree. The only thing that kept me back was the status of the support of my everyday packages. To keep track of the evolution of the support of your preferred package, two good point of reference are [python 3 <s>wall of shame</s> superpowers](http://python3wos.appspot.com/) and the official page of the top support in python 3 [http://py3ksupport.appspot.com/](http://py3ksupport.appspot.com/). And do me a favor, start writing python 3 compatible script with a proper use of the `__future__` statement and the [six](http://pythonhosted.org/six/) library!

Aside from that, I spent a lot of time working with the guys of the [statsmodels](http://statsmodels.sourceforge.net/) project. I got a pull request accepted to implement the mosaic plot and two more are waiting a response: one is an poor-man implementation of the facet plot and one is targeted to microarray and pathway analysis. I have to admit that this has been a TREMENDOUS experience. I learned a lot of things from them, first of all the huge gap that exist between writing code that do something and code that let other do the same thing. Code readibility, good docstring, package organization and a lot of new, fun things to do. It's has been a good excuse to get more confident with the git workflow, that I have always wanted to learn but always postponed.

I also tried to post a package on pipy, the central python packages repository. The package is named [keggrest](https://pypi.python.org/pypi/keggrest/0.1.1), and it's a basic implementation of the rest API of the KEGG biological database. It's a shame, as it has close to no documentation and no support to python 3. Talk about throwing the first stone :D. In the next few days I will give it some love to make it a package like the one that I expect to use.

See you soon with more updates and material!


