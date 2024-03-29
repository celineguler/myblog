---
layout: post
title:  "ONNs"
---

While electrons often considered more suitable for information processing (reasons such they are easy to manipulate by EM fields since they have charge and mass, they do interact unlike photons and you have switching speed since it's easier to stop electrons), ONN offers to find novel ways of information processing and advantages such as high-speed parallelism, low power consumption, and potential for high throughput. We can discuss ONN better if we talk more on photonics for computing first. So, why photonics for computing?

>	 Light provides an enormous bandwidth, possibility of wavelength division multiplexing.

While it more sounds like communication then computing and just helps computing.

Here's an article called [optical-neural-networks-hold-promise-for-image-processing][optical-neural-networks-hold-promise-for-image-processing] about more energy-efficient image sensors.

[CONF-CDS 2020][CONF-CDS 2020] offers inference is the only optically suitable part and not the training. This article [training neural networks with end-to-end optical backpropagation][training neural networks with end-to-end optical backpropagation] focused on this problem (I only read the abstract).


{: .italic-gray.indented}
> *"However, to reach the full capacity of an optical neural network it is necessary that
the computing not only for the inference, but also for the training be implemented optically."*

<br>

Let's discuss this where the above articles casually lead me to it. But first, I'm gonna try to clarify the concepts of ONNs (optical neural networks) and ANN's (artificial neural networks) where I only have the sence of ANNs. Also, here's a casual [brainfuck][quantum] called QNNs for companions.

Firstly, I can declare my early confussions on the physical structures on neural networks. We know that ANNs are mathematical models at their core which represents a virtual structure for connections between artificial neurons. So the computations in ANNs occurs through mathematical operations which needs to be executed on a computational device, such as [CPU or GPU][cpu] (there's a discussion on which unit is more suitable for machine learning). So physical electronic structure is the implementation of ANNs into hardware or basically direct implementation of any mathematical computation --- this involves using electronic components to simulate mathematical operations performed by artificial neurons. Which is called computation. Thank you for your time. But it was a challange to understand the virtual structure representation of the architecture and computations are conceptually independent of any specific physical implementation. 

Optical neural networks, on the other hand, are designed and implemented using optical components like lasers, waveguides, and photodetectors. The physical design of these networks is different from traditional electronic neural networks and requires an sense of optical computing. ONNs manipulate and process information using light. So mathematical operation architecture based on the manipulation of the light signals through the interference, diffraction and other optical phenomena. Implementing ONNs typically involves designing and building physical optical structures. While traditional ANNs can be run on standard CPUs and GPUs, ONNs are a specialized area that involves the design and construction of physical optical structures due to their use of optics for computation. We test [*optical computation*][oc].

You may now get my confussion on this physical structure implementation, it's neither an *upgrade*, *transformation* or *replacement* on classical computing. Which is confusing. 























![onn2](/myblog/images/onn.png)

<br>











[oc]: https://en.wikipedia.org/wiki/Optical_computing
[tinygrad]: tinygrad.org
[cpu]: https://www.analyticsvidhya.com/blog/2023/03/cpu-vs-gpu/#:~:text=In%20conclusion%2C%20several%20steps%20of,GPUs%20may%20both%20be%20utilized.
[quantum]: https://en.wikipedia.org/wiki/Quantum_neural_network
[training neural networks with end-to-end optical backpropagation]: https://arxiv.org/abs/2308.05226#:~:text=However%2C%20to%20reach%20the%20full,the%20training%20be%20implemented%20optically.
[CONF-CDS 2020]: https://www.youtube.com/watch?v=EfGLJ47dg80
[optical-neural-networks-hold-promise-for-image-processing]: https://news.cornell.edu/stories/2023/04/optical-neural-networks-hold-promise-image-processing