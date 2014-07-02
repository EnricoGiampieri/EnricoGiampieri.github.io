---
layout: post
title:  "fit of an curve, explicit or implicit"
date:   DRAFT
categories: introduction
tags: [python]
--- 


{% highlight python %}
data = [ [cos(t)+0.1*randn(),sin(t)+0.1*randn()] for t in rand(100)*2*np.pi ]
contour = array(data)
x,y = contour.T

 

def f(coef):
    a = coef
    return a*x**2+a*y**2-1

 

from scipy.optimize import leastsq
initial_guess = [0.1,0.1]
coef = leastsq(f,initial_guess)[0]
# coef = array([ 0.92811554])

 

def f(coef):
    a,b,cx,cy = coef
    return a*(x-cx)**2+b*(y-cy)**2-1

initial_guess = [0.1,0.1,0.0,0.0]
coef = leastsq(f,initial_guess)[0]
# coef = array([ 0.92624664,  0.93672577,  0.00531   ,  0.01269507])

 

res = leastsq(f,initial_guess,full_output=True)
coef = res[0]
cov  = res[1]
#cov = array([[ 0.02537329, -0.00970796, -0.00065069,  0.00045027],
#             [-0.00970796,  0.03157025,  0.0006394 ,  0.00207787],
#             [-0.00065069,  0.0006394 ,  0.00535228, -0.00053483],
#             [ 0.00045027,  0.00207787, -0.00053483,  0.00618327]])

uncert = sqrt(diag(cov))
# uncert = array([ 0.15928997,  0.17768018,  0.07315927,  0.07863377])    

{% endhighlight %}