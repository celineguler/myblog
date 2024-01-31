---
layout: post
title: "beam_analysis.py"
---

This paper covers the analysis of laser profiles since these studies of intensity distribution, laser power, number of modes and laser beam are quite important in many applications. I studied the paper and build up some Python code that can useful for the data management through out the analysis process. Let me break down concepts first but I need mention that I'll be quoting my poorly written report from a century ago. So the concept covering for this post is not bit of a clean-cut.


A laser beam is a highly focused and collimated beam of light that is produced by a laser. Laser beam analysis is a set of techniques used to measure and analyze the properties of laser beams. This can include the *beam's intensity, power, shape, and spatial and temporal* characteristics. Laser beam analysis is important in a variety of applications, including manufacturing, scientific research, and military and defense. There are several methods for analysis of a laser beam profile such as __scanning aperture__, __scanning knife-edges__, __detector arrays__, __burn patterns__ and __photographic technique__ which studied in this paper.

<br>

__PRINCIPLES AND DEFINITIONS__ 

__Determining the laser source and type of laser beam we are working with. This will determine the appropriate method for analyzing the beam profile.__ In this paper He-Ne and diode lasers studied that emits pure Gaussian beam and elliptical beam shape, respectively as results concludes.

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