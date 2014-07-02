---
layout: post
title:  "Moving to python 3k for numerical computation"
date:   2012-12-10 17:33:00
categories: introduction
tags: [python python3]
---

In the last few years of working with python, I've always suffered from being kept back to the 2.x version of python by the need of the scientific libraries. The good news is that in the last year most of them made the great step and shipped a 3.x ready version (or, to be honest, a 3.x convertible 2.x version).

So right now I'm having fun trying to install everything on my laptop, an Ubuntu 12.10.

The first step is to install python 3.2 and the appropriate version of the pip packaging system:

    sudo apt-get install python3 python3-pip

then we can just plug the normal installation process using the pip-3.2

    sudo pip-3.2 install -U numpy
    sudo pip-3.2 install -U scipy
    sudo pip-3.2 install -U matplotlib
    sudo pip-3.2 install -U sympy
    sudo pip-3.2 install -U pandas
    sudo pip-3.2 install -U ipython
    sudo pip-3.2 install -U nose
    sudo pip-3.2 install -U networkx
    sudo pip-3.2 install -U statsmodels
    sudo pip-3.2 install -U cython

Sadly mayavi, scikit-learn, numexpr, biopython and tables are still working on the transition, so they're not yet available. This leave the numerical side of python quite crippled, but I hope that they will soon reach the others and allow us to use py3k as the rest of the world out there. 
