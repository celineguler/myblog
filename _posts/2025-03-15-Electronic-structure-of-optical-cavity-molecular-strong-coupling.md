---
layout: post
title: "Elec. Struct. 4 QED Cavity molecules"
---

  <!-- MathJax Script -->
  <script type="text/javascript" async
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
  </script>

 
{% highlight ruby%}

import numpy as np
import matplotlib.pyplot as plt

temp = [299.95, 323.15, 348.15, 373.15,398.15,423.15, 448.15, 473.15, 523.15, 548.15, 573.15, 598.15, 623.15]
U_up = [0.039, 0.123, 0.241, 0.351, 0.505, 0.688, 0.904]
U_down = [1.49, 1.74, 1.96, 2.09]

T0 = 299.95
Tdif = (np.power(np.array(temp), 4) - np.power(T0,4)).tolist()


m, b = np.polyfit(Tdif[:7], U_up, 1)
s, p = np.polyfit(Tdif[8:12], U_down, 1)

up_fit = m * np.array(Tdif[:7]) + b
down_fit = s * np.array(Tdif[8:12]) + p


plt.scatter(Tdif[:7], U_up)
plt.scatter(Tdif[8:12], U_down)
plt.plot(Tdif[:7], up_fit, linestyle='--', color='red', label='y=mlog(T) + b')
plt.plot(Tdif[8:12], down_fit, linestyle='--', color='pink', label='y=slog(T) + b')
plt.xlabel('log T(K)')
plt.ylabel('log U(mV)')
plt.grid(True)
plt.legend()
plt.show()

{% endhighlight %}



{% highlight ruby%}

import numpy as np
import matplotlib.pyplot as plt

temp = [299.95, 323.15, 348.15, 373.15,398.15,423.15, 448.15, 473.15, 523.15, 548.15, 573.15, 598.15, 623.15]
U_up = [0.039, 0.123, 0.241, 0.351, 0.505, 0.688, 0.904]
U_down = [1.49, 1.74, 1.96, 2.09]


m, b = np.polyfit(np.log(temp[:7]), np.log(U_up), 1)
y_fit = m * np.log(temp[:7]) + b

plt.scatter(np.log(temp[:7]), np.log(U_up))
plt.plot(np.log(temp[:7]), y_fit, linestyle='--', color='red', label='y=mlog(T) + b')
plt.xlabel('log T(K)')
plt.ylabel('log U(mV)')
plt.grid(True)
plt.legend()
plt.show()

{% endhighlight %}