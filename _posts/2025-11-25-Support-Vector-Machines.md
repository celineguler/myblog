---
layout: post
title: "Support Vector Machines"
---

<link rel="stylesheet" href="file:///C:/Users/Selin/notes_style/style.css">

<div class="note">

<p>
<h1>Support Vector Machines</h1>
What are support vector machines?
It is a supervised learning framework.
<br>
First of all, what are support vectors? They are a subset of training points in the decision function. They are called support vectors.
<br>
What does it mean that different Kernel functions can be specified for the decision function?
<br>
<b>What is a Kernel?</b>
<br>
Note that we follow the https://scikit-learn.org/stable/modules/svm.html# page for content. We will add AIMA 18.6 and 18.9 as well.
<br><br>

![3](blog/img/shot_1763858316.png)

![4](blog/img/shot_1763858516.png)
<br>







<br>
<h2>1.4.1. Classification</h2>
![5](blog/img/shot_1763824580.png)

<h4>SVC, NuSVC, LinearSVC...</h4>
![6](blog/img/shot_1763922269.png)

Support vector machines finds an optimal seperation of the data. Either linear or non-linearly. Obviously, nonlinearly it is harder.<br>
<b>
<br>
<p style="text-align: center;"> margin = min(P, K)</p>
 where P and K are the distances to the separation line of data points belonging to two different classes.
<br><br>SVM trying to maximize the margin around the hyperplane. It means it tries to optimize the distance from the data points (support vectors) around the hyperplane.
<i>It is a maximization problem.</i> We can use Lagrangian multipliers (always!) when there is a maximization problem and some possible constraints.</b>
<br>
<h2>2D-case</h2>
Hyperplane is defined by (a,b,c) coefficient where,
<p style="text-align: center;">(1) ax+by-c>=0</p>
<p style="text-align: center;">(2) ax+by-c=< 0</p>
The (1) for class 1 and (2) for class 2 data points. This is the equation of the seperation line. Now you can change your notation, defining:
<br>
<h3>Notation</h3>
![7](blog/img/shot_1764030396.png)
the weight vector and the position vector:
![8](blog/img/shot_1764030437.png)
and get:
![9](blog/img/shot_1764030683.png)
notation for the hyperplane (if you are confused, theta=w, x=xi and theta_0 is c).
<br>Here the w is the weight vector. c is the shift from the origin. You may ask, if w elements are nonzero, then xi is a support vector?
<br>
Check: https://kuleshov-group.github.io/aml-book/contents/lecture13-svm-dual.html#the-dual-of-the-svm-problem
<br>
<b>No.</b> We actually check the contribution to the weight by checking if lagrange multiplier of xi is 0 or nonzero. Within:
![11](blog/img/shot_1764030915.png)

xi: ith training point<br>
yi: ith label<br>
λi: Lagrange multiplier.<br>
<br>
if λi is 0 for a xi, then xi is not a support vector. 

<br>
There a lot of solutions for (a,b,c).
<br>
Which points should influence the optimality?
<br>
If you say all points, you are thinking of <b>Linear regression</b> or<b> Neural nets</b> probably.
<br>
If you are smart enough to conclude we don't need the points that probably won't influence the hyperplane, and we need only diffuculut points close the boundary, 
you are thinking of <b>Support Vector Machines</b>.
<br>
<h3>Support vectors are the data points of the training set that would influence or change the position of the dividing hyperplane if removed!!!</h3>
Note that, support vectors are actual vectors indeed. Mostly the magnitude of these vectors are discussed, as it is the actual distance of the support vector 
data point from the seperation line, but since we have an origin (0,0) and a coordinate system, these data points are actually vectors.
![22](blog/img/shot_1764027490.png)
The d is the 1/2 of "street width" (the margin).
<br>
Note that, support vectors have nonzero weights, and by maximazing the margin we try to <b>reduce the number of the weights</b>. Important to note, not reduce weights, but the number of the weights!!!
<br>
Each non-zero weight αᵢ corresponds to one support vector. Maximizing margin naturally tries to minimize the number of non-zero αᵢ.
![33](blog/img/shot_1764028335.png)

<br>
<h4>Q1: Can a support vector become non-support during optimization? (Meal: Can the nonzero weight of a support vector reduce to zero during optimization?)<br>
Q2: What if a support vector’s αᵢ decreases but not to zero? Does it affect optimality?</h4>
Discuss. Test your understanding. Both answers are yes.


![44](blog/img/shot_1763922764.png)
You can find the proof: https://en.wikipedia.org/wiki/Hyperplane_separation_theorem
<br>
Also check out: https://arxiv.org/pdf/1107.1358 <b><i>"On the Furthest Hyperplane Problem and Maximal Margin Clustering"</i></b>
<br>
<h3>PROBLEM: The problem of finding an optimal the hyperplane is an optimization problem. We need <i>Lagrange Multipliers</i>.</h3>


<br><br><br><br>
<h2 style="text-align: center;">LAGRANGE MULTIPLIERS</h2>
![55](blog/img/shot_1764024683.png)
If you have a minimization or a maximization problem and some constraints in your system, you need Lagrange Multipliers.
<br>
As you have the gradient of your function and gradient of your constraints:
![66](blog/img/shot_1764024868.png)

<br><br>
Here is the Lagrangian of the max-margin optimization problem.
![111](blog/img/shot_1764028909.png)

(Is it accurate for the 2D-case or..?)

<h3>Primal and Dual Problems of SVM</h3>
Every constrained optimization problem has a paired problem called its <b>dual problem</b>, constructed from the Lagrangian of the original problem.
![1111](blog/img/shot_1764030161.png)

<br>
<br><br><br>

<h3>IMPORTANT!!!</h3>

Kernel function is <b>NOT</b> the hyperplane or decision surface. Hyperplane is the decision boundary, the seperation line.
<br>
Kernel function is a similarity measure that allows the SVM to compute inner products in the feature space without ever computing 𝜙(𝑥) explicitly.
<br>
<br>
<br><br>
The following is the hyperplane:
![1111111](blog/img/shot_1763928961.png)
It determines the decision boundary. x is the samples or data points vector. w is the normal line to the seperation line. b is the shift from the origin.
transpose(w).x is the dot product and it is basically sum over w.x vectors. Or dot products of w and x vectors. What does it refer to?
<br>
We try to optimize the margin. Where margin = 2 / ||w|| .

<h3>Larger the margin, better the SVM!!!</h3>
smaller ||w||, larger the margin, better the SVM. Therefore the optimization goal is:
<br>
![1111111111](blog/img/shot_1763929359.png)



<br>
<h3>Kernel... Kernel is just a function. A similarity function that measured the similarity between two points in a high dimensional space.</h3>
<br>
What does high-dimensional feature space refer to? If your data has two features, it lives in 2D space. If 3 features, then 3D space and if 100 features then
100-dimensional space. All of the data points or samples must have the same number of features (dimensions)!!! Otherwise the algorithm will fail..
<br><br>
![222](blog/img/shot_1763853284.png)
In scikit learn, you can model these kernel like following:
<br>
clf = svm.SVC(kernel="linear", C=C)<br>
clf = svm.SVC(kernel="rbf", gamma=, C=C)<br>
clf = svm.SVC(kernel="poly", degree=, gamma=, C=C)<br>
then you can fit them.<br>
clf.fit(X: your_samples_array, y: your_classes_array)
![333](blog/img/shot_1763854216.png)

also check: https://en.wikipedia.org/wiki/Radial_basis_function_kernel
<br>
https://en.wikipedia.org/wiki/Similarity_measure
<br>
https://en.wikipedia.org/wiki/Kernel_method
<br><br>
Do not mix up the kernel (statistics) which is a different definition and basically refers to kernel as the unnormalized form of a distribution.

<br><br>

<h3>1.4.1.1. Multi-class classification</h3>
Multi-class classification refers to when your label count is > 2 basically.
<br>
y = [0,1] => binary Classification
<br>
y = [0,1,2] => Multi-class classification already.
<br>
SVM was designed for binary classification. SVM is originally designed by Vladimir Vapnik in 1990s for binary classification, +1 and -1.
<br>
![444](blog/img/shot_1763854845.png)
<b>This is SVM optimization problem. Leading us to understand why is it originally a binary classifier and not Multi-class classifier.</b><br>
Multi-class clf. approach:    https://machinelearningmastery.com/one-vs-rest-and-one-vs-one-for-multi-class-classification/
<br><br>
For multi-classification problems, we split the multi-class classification dataset into multiple binary classification datasets and fit a binary classification model on each.
![555](blog/img/shot_1763855018.png)

<br>
one-versus-one is applied. What is ovo method? It reshapes the decision_function. to n_classes * (n_classes - 1) / 2.
<br>
Decision function returns an array, and distance from the "hyperline". It's shape refers to what? Reshaping it how?
<br>
Check out One-vs-Rest and One-vs-One strategies as well.



<h3>1.4.1.2. Scores and probabilities</h3>

For each class (or label), the datapoint is assigned to a score (or a probability). For binary case it is a single score for each sample, for multiple classes the samples
is assigned to scores for each class. Looking something like:
<br>



<h3>1.4.1.3. Unbalanced problems</h3>


<h4>SVMs decision function. WTF is that?</h4>

<h2>1.4.2. Regression</h2>

Support vector classification can be used to solve regression problems. As it is called <b>Support Vector Regression</b>.
<br>First things first, what the fuck is a support vector?
<br>I will plug this example right here from svm.py code:
![666](blog/img/shot_1763915897.png)
Support vector(s) are returned by a classifier object. What is a classifier object?
<br>
>>> clf2.support_vectors_
<br>
array([[ 0.,  0.],
       [-1., -1.],
       [ 1.,  1.],
       [ 9.,  9.]])
<br>
also this is the support vector of clf2. Let's understand what it refers to.
<br>
<b>These two classifiers who are using the same sample set and label set, returning different support vectors refer to their different decision boundaries!!!</b>
<br>
![1](blog/img/shot_1763916481.png)
Therefore support vectors refer to the closest points (from sample set) to the decision boundary of the classifier. They are the points that satisfy:
![2](blog/img/shot_1763916549.png)
When a new sample (a data point) is provided to the SVM, it is only compared to the support vectors, not all data points.
<br>
Support vectors are the most difficult data points to classify.
<h2>The decision (or prediction) function!!!!</h2>
Welcome to the math. 
































<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
_________________________________________________________________________________________________________________


<h3>Source:</h3>
An Idiot’s guide to Support vector machines (SVMs) https://web.mit.edu/6.034/wwwbob/svm.pdf 






</p>
</div>