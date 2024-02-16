---
layout: post
title: "selinguler94/beam_analysis"
---


This paper covers the analysis of laser profiles since these studies of intensity distribution, laser power, number of modes and laser beam are quite important in many applications. I studied the paper and build up some Python code that can useful for the data management through out the analysis process. Let me break down concepts first but I need mention that I'll be quoting my poorly written report from a century ago. So the concept covering for this post is not bit of a clean-cut.


A laser beam is a highly focused and collimated beam of light that is produced by a laser. Laser beam analysis is a set of techniques used to measure and analyze the properties of laser beams. This can include the *beam's intensity, power, shape, and spatial and temporal* characteristics. Laser beam analysis is important in a variety of applications, including manufacturing, scientific research, and military and defense. There are several methods for analysis of a laser beam profile such as __scanning aperture__, __scanning knife-edges__, __detector arrays__, __burn patterns__ and __photographic technique__ which studied in this paper.

<br>

__PRINCIPLES AND DEFINITIONS__ 

1. Determining the laser source and type of laser beam we are working with. This will determine the appropriate method for analyzing the beam profile. In this paper He-Ne and diode lasers studied that emits pure Gaussian beam and elliptical beam shape, respectively as results concludes.

2. Measuring the dimensions of the laser beam. We will need to know the width, height, and length of the beam in order to calculate its properties. This can be done using a variety of techniques, such as using a beam profiler or imaging the beam using a camera or other imaging device.

__Beam Diameter__: The beam diameter is the width of the laser beam at a particular point. Using photographic tecnique for a focused beam, using a CCD to capture the spot and fitting it with 2D Gaussian function will be enable to calculate beam diameter of the beam.

	D = 2 * √(P / I π)
	I₀ = 2 * P / (π * w₀ ²)

__D__ is the beam diameter at at a given distance

__P__ is the power of the laser

__I__ is the intensity at a given distance 

{% highlight ruby%}

import matplotlib.pyplot as plt
from scipy import special
from scipy.optimize import curve_fit
from matplotlib import pyplot as plt
import numpy as np

# data
power, distance = [], []


with open("power.txt", "r") as power_values:
    power_lines = power_values.readlines()
with open("distance.txt", "r") as distance_values:
    distance_lines = distance_values.readlines()
    
    
for i in range(len(power_lines)):
    power.append(float(power_lines[i]))
    distance.append(float(distance_lines[i]))

    
# the function to fit to the data
def power_func(x, a, b):
    return a * np.exp(-b * x)


# fit
params, covar = curve_fit(power_func, distance, power)
a, b = params


# generated data
x_curve = np.linspace(0, 2, 50)
y_curve = power_func(x_curve, a, b)


plt.plot(distance, power, '-', color='g')
plt.plot(x_curve, y_curve, '--', color='b')


plt.xlabel('Distance (m)')
plt.ylabel('Power (mW)')
plt.show()

{% endhighlight %}

__Beam Waist__: The beam waist is the point at which the beam has the smallest diameter. The beam waist can be measured using a beam profiler or by performing a series of measurements at different distances from the source to determine the point at which the beam reaches its minimum diameter.
