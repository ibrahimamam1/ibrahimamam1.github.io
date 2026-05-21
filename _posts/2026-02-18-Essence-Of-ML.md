# Introduction 
Three months ago I completed my machine learning course at Uni and learned all the classic algorithms and some introduction to deep learning. I learned enough to do well on my exams but I felt I didn't really get the intuition behind it all so I decided to start again from the ground up and this time make sure I do. This post is basically me sharing my notes on what I learned. It is very light on the actual math (which I thoroughly worked on pen and paper and you should too), it aims purely at presenting the essence of machine learning and its algorithms.

# The Essence Of Machine Learning
Imagine you have a magic black box and you can feed it any input and the magic box tells you something about the input. There are no restrictions to the type of data you can feed in, it could be anything really: text, image, list of numbers (we call them feature vectors), whatever. The information we get from the magic box can be anything from a continuous number representing something like for example the estimated price of RAM sticks next year, or a probability that a certain thing is true or simply a label saying this image is a cat.

The magic black box is what we call a machine learning model and is essentially a mathematical model of how the inputs map to the output. This model is learned using statistical and optimization techniques applied to large amounts of data.

There are three types of learning algorithms which differ by the type of data available: Supervised Learning, Unsupervised Learning and Reinforcement Learning. 

I will not talk about reinforcement learning because I believe it deserves an article of its own which I'm planning to write soon. Hence we will focus only on supervised and unsupervised learning.

# Supervised Learning
Supervised Learning algorithms work on data for which the labels (outputs) are known. That is, for each data point in our training dataset we know what the correct answer should be. This class of algorithms learns the mathematical relationship between the inputs and the correct outputs and uses that to make predictions on new inputs never seen before.

## Linear Regression
Suppose you have data on the price of wine based on its age. Your data might look like this:

| Age (Years) | Price ($) |
|:-----------:|:---------:|
|      1      |     10    |
|      2      |     20    |
|      3      |     28    |
|      4      |     42    |
|      5      |     50    |

If we plot the data it looks like this:

```
Price ($)
  ^
50|             x
40|           x
30|         x
20|       x
10|     x
  |__________________> Age
      1   2   3   4   5
```

The main idea of linear regression is to find the best straight line that can fit this data. With a proper equation for the straight line we can find the output value y (price) for any given input x (age). 

```
Price ($)
  ^
50|             x  /
40|           x  /
30|         x  /  <-- prediction(x) = Wx + b
20|       x  /
10|     x  /
  |___/____________> Age
      1   2   3   4   5
```

Our model (also called hypothesis) is hence:

$prediction(x) = Wx + b$

The question remains of how to find the values of W and b that are best for the data. What we want is to minimise the difference between the prediction and the actual value on average. This is called an objective function, sometimes we want to minimize it (in which case we call it a loss function) and other times we want to maximize it. Here we want to minimize it.


$$J(W, b) = \frac{1}{2m} \sum_{i=1}^{m} (prediction(x^{(i)}) - y^{(i)})^2$$

This optimization problem can be solved with an iterative algorithm called gradient descent. For every prediction we calculate the error, compute its gradient (the direction of steepest ascent of the objective function) and go in the reverse direction. We use the negative gradient to update our values of W and b and eventually we will end up at the absolute minimum.


## Logistic Regression
Suppose instead of the price of wine we want to know its quality i.e. good or bad based on the age. This is a Binary Classification problem and logistic regression is great at this. 

This time the data looks like this:

| Age (Years) | Quality (0=Bad, 1=Good) |
|:-----------:|:-----------------------:|
|      1      |            0            |
|      2      |            0            |
|      4      |            1            |
|      5      |            1            |

We should not fit a straight line to this data because we do not want continuous values. We want a binary value, either 0 (bad) or 1 (good). 

One function we could fit to this data is the sigmoid function which has output between 0 and 1.

```
     1 |            ___________
       |           /
P(y=1) |          /
   0.5 | - - - - + - - - -
       |        /
     0 |_______/_________
       |       |
      Bad     Good
```

Since the output of a sigmoid is always between 0 and 1 it can be interpreted as the probability that the given input is from the positive class. If the value is greater than or equal to 0.5 we output 1 (Good) otherwise we output 0 (Bad). 

The logistic regression hypothesis is then:

$$h_\theta(x) = \frac{1}{1 + e^{-(Wx + b)}}$$

So how can we find the best sigmoid? What kind of loss function can we use? Think about it for a second.

We need a loss function that encourages confident (higher probability) and right predictions while heavily penalising confident and wrong predictions. A perfect loss function for this is the cross entropy loss or information gain. It is defined as:


$$J(W,b) = -\frac{1}{m} \sum_{i=1}^{m} [y^{(i)}\log(h_\theta(x^{(i)})) + (1-y^{(i)})\log(1-h_\theta(x^{(i)}))]$$

When the prediction is right, the equation simplifies to $-\log(p)$ which results in lower loss the higher p was (the more confident we were). When the prediction is wrong, the equation simplifies to $-\log(1-p)$ which results in greater loss the more confident we were in our prediction. Exactly what we wanted.

Through gradient descent we learn the optimal $W$ and $b$ for our data and that's logistic regression.

## Softmax Regression
In the previous example we wanted to know whether the wine was good or bad. Now suppose we want to know whether it is Great, Good, Average or Bad.

Our data now looks like:

| Age | Acid | Class   |
|:---:|:----:|:-------:|
|  10 |  High| Bad     |
|  12 |  Mid | Average |
|  15 |  Low | Good    |
|  20 |  Low | Great   |

This is a multiclass Classification Problem, and a great algorithm for solving such problems is Softmax Regression. Logistic regression is actually a special case of Softmax Regression where the number of cases is 2. 

In Softmax Regression we build a model for each of the classes independently. For each class we ask "what is the probability this sample belongs to this class?". This can be known through the computation:

$$P(y=j\mid x) = \frac{e^{W_j x + b_j}}{\sum_{k=1}^{K} e^{W_k x + b_k}}$$

We then simply proceed to choose the class with the highest probability of owning the sample.

The loss function for Softmax is similar to that of logistic regression. It simply generalises it from 2 classes to k classes:

$$J(\theta) = - \sum_{i=1}^{m} \sum_{k=1}^{K} \mathbb{1}\{y^{(i)} = k\} \log \frac{e^{W_k x^{(i)}}}{\sum_{j=1}^{K} e^{W_j x^{(i)}}}$$

That is softmax Regression.

## Gaussian Discriminant Analysis
GDA is a binary classification algorithm but unlike all the previous algorithms it is what is called a Generative Learning Algorithm. 

Logistic Regression and Softmax are a class of learning algorithms called Discriminative learning algorithms. Discriminative because they discriminate between a number of classes by modeling the probability $p(y \mid x)$.

Generative Learning algorithms on the other hand model the probability $p(x \mid y)$. Having a model for $p(x \mid y)$ gives them the ability to generate new data points, hence the name Generative Learning algorithms.

GDA assumes the distribution for $p(x \mid y)$ is Gaussian. This is not a far-fetched assumption as most of the data out there is Gaussian. However, if you know for a fact your data is definitely not Gaussian then GDA might fail and you probably should not use it. 

If $p(x \mid y)$ is Gaussian then using the Gaussian PDF we can compute:

$$p(x\mid y=0) = \frac{1}{(2\pi)^{n/2}|\Sigma|^{1/2}} \exp\left(-\frac{1}{2}(x-\mu_0)^T\Sigma^{-1}(x-\mu_0)\right)$$
$$p(x\mid y=1) = \frac{1}{(2\pi)^{n/2}|\Sigma|^{1/2}} \exp\left(-\frac{1}{2}(x-\mu_1)^T\Sigma^{-1}(x-\mu_1)\right)$$

$$\mu_0 = \frac{\sum_{i=1}^m \mathbb{1}\{y^{(i)}=0\} x^{(i)}}{\sum_{i=1}^m \mathbb{1}\{y^{(i)}=0\}}$$

$$\mu_1 = \frac{\sum_{i=1}^m \mathbb{1}\{y^{(i)}=1\} x^{(i)}}{\sum_{i=1}^m \mathbb{1}\{y^{(i)}=1\}}$$

$$\Sigma = \frac{1}{m} \sum_{i=1}^m (x^{(i)} - \mu_{y^{(i)}})(x^{(i)} - \mu_{y^{(i)}})^T$$

With what we obtained we can apply Bayes Rule to find $p(y=1\mid x)$:

$$p(y=1\mid x) = \frac{p(x\mid y=1) \cdot p(y=1)}{p(x)}$$

$$\phi = \frac{1}{m} \sum_{i=1}^m \mathbb{1}\{y^{(i)}=1\}$$

As you can see there is no optimization needed for this algorithm, hence gradient descent and loss functions are not needed. Just by computing probabilities from the data we can make predictions about unseen data. 

When should you use GDA over a discriminative algorithm like logistic regression? Generally if you have enough data you should use logistic regression as it is more robust, but for small amounts of data you think might be Gaussian, GDA is the better choice.

## Naive Bayes 
Naive Bayes is similar to GDA except that we do not assume $p(x \mid y)$ is Gaussian. Rather we compute it from the data.

$$\phi_y = \frac{\sum_{i=1}^m \mathbb{1}\{y^{(i)}=1\}}{m}$$

$$\phi_{j|y=1} = \frac{\sum_{i=1}^m \mathbb{1}\{x_j^{(i)}=1 \land y^{(i)}=1\}}{\sum_{i=1}^m \mathbb{1}\{y^{(i)}=1\}}$$

$$p(y=1\mid x) = \frac{p(x\mid y=1)p(y=1)}{p(x)}$$

Naive Bayes assumes that the features are conditionally independent:

$$p(x\mid y) = \prod_{j=1}^{n} p(x_j\mid y)$$

## Support Vector Machines 
Support vector machines are one of the most important algorithms in ML. They can be used both for classification and regression. 

What makes SVMs special is that they have the capacity to learn a non-linear decision boundary. All the algorithms we have seen so far can only learn a linear decision boundary (meaning a straight line that can separate the classes). For some data this is just not possible; a linear decision boundary does not exist. A classic example of this is XOR.

| x1 | x2 | y (Class) |
|:--:|:--:|:---------:|
| 0  | 0  |     0     |
| 0  | 1  |     1     |
| 1  | 0  |     1     |
| 1  | 1  |     0     |

```
x2
^
|  1(x)    0(o)
|
|  0(o)    1(x)
|____________> x1
   (No straight line can separate x and o)
```

There is no possible straight line that can accurately separate the two classes. The only possible way to separate the two classes is with a non-linear function.

```
x2
^
|  1(x)  /  0(o)
|       /
|      /
|  0(o)  /  1(x)
|_______/_____> x1
(A non-linear boundary wraps around the classes)
```

What SVMs look for is the optimal hyperplane which separates the classes (this will be a straight line if the data is linearly separable). 

Previously we encoded our classes as positive class = 1, negative class = 0. We need to switch this notation a little bit to positive class = 1, negative class = -1. This makes the upcoming maths much easier.

Let's define what is called the functional margin as:

$$\hat{\gamma}^{(i)} = y^{(i)}(w^T x^{(i)} + b)$$

The functional margin represents the confidence we have in a given prediction. If the class is positive then the higher the value of $wx + b$ the higher our functional margin. If the class is negative, the higher the value of our prediction the lower the functional margin (because $y_i = -1$).

We can compute the actual distance between each data point and the decision line; this is called the Geometric margin:

$$\gamma^{(i)} = \frac{y^{(i)}(w^T x^{(i)} + b)}{||w||}$$


The best hyperplane we are looking for is the one that maximises the geometric margin, hence the distance between the closest data points to the decision boundary (called the support vectors). This allows us to maximize the probability of correctly classifying each class because they are well distinguished.


First let's consider the case where the data is linearly separable.

Our objective is to maximize the geometric margin:

$$\max_{\gamma, w, b} \gamma \quad \text{s.t.} \quad y^{(i)}(w^T x^{(i)} + b) \geq \gamma, \quad ||w|| = 1$$

The solution to this optimization problem is:

$$\min_{w,b} \frac{1}{2} ||w||^2 \quad \text{s.t.} \quad y^{(i)}(w^T x^{(i)} + b) \geq 1$$

That is the loss function of this model which is called the optimal margin classifier. It is the baby SVM.

The SVM we saw is what is called a hard margin SVM. It never allows any misclassifications and strictly enforces a decision boundary that clearly separates the two classes. On some datasets with noise or outliers this will backfire. 

The solution is to introduce the concept of soft assignment. Instead of rigidly trying to classify each data point correctly we accept some degree of error. We introduce slack variables that allow the constraint to be violated.

$\delta_i \geq 1-\epsilon_i$

The new objective becomes:

$$\min_{w,b} \frac{1}{2} ||w||^2 + C \sum_{i=1}^m \delta_i \quad \text{s.t.} \quad y^{(i)}(w^T x^{(i)} + b) \geq 1 - \delta_i, \quad \delta_i \geq 0$$

We can optimize this new objective through gradient descent.


What about when the data is non-linearly separable? For that we need something called the kernel trick. The idea is simple: if the data is non-linearly separable then let's map it to a higher-dimensional space where it is linearly separable. The kernel is the function used to map the data to the higher-dimensional space; there are a variety of them and you can even design your own if you want (and are really good at maths lol). For all the details and the maths please refer to [this](https://kuleshov-group.github.io/aml-book/contents/lecture12-suppor-vector-machines.html), [this](https://kuleshov-group.github.io/aml-book/contents/lecture13-svm-dual.html) and [this](https://kuleshov-group.github.io/aml-book/contents/lecture14-kernels.html) lecture.

## Neural Networks
Neural Networks are also a supervised learning algorithm and can be used for pretty much any task under the sun. The theory and applications of neural networks is so broad it deserves an article of its own (coming soon) so I do not discuss it here. In the meantime you can have a look at this [lecture](https://www.youtube.com/watch?v=MfIjxPh6Pys&list=PLoROMvodv4rMiGQp3WXShtMGgzqpfVfbU&index=11).

## K Nearest Neighbours
KNN can be both a classification and a regression algorithm. As a classification algorithm it classifies each data point the same as the k data points that are geometrically closest to it. As a regression algorithm it predicts the mean of the k data points that are closest to it.

There is no learning or optimization taking place in this algorithm. KNN just stores all the training data and to make a prediction it iterates through all training examples and computes the distance between each example and the input, it then selects the k closest ones and either returns the most common class among those k neighbours in a classification task or the mean value of the labels in a regression task.

This concludes our discussion on supervised learning algorithms.

# Unsupervised Algorithms
Unsupervised learning works on training data for which we do not have the ground truth; we only have the inputs and need to figure out something useful about them.

## K Means Clustering
K-means is a clustering algorithm whose goal is to group similar data points together into clusters. Suppose we have the data:

```
Feature 2
  ^
  |   xx      oo
  |  xxx     oo
  |   x       o
  |
  |      ++
  |     +++
  |_________________> Feature 1
```

Given a value k, the goal of the algorithm is to form k distinct clusters each containing similar items. The algorithm is very straightforward:

We need to find k points that will serve as centers for the clusters. We start with any random points. We call them centroids.
We compute the distance between each datapoint and all k centroids.
We assign each data point to the centroid it is closest to.
We update the position of each centroid with the mean of all the datapoints assigned to it. 
We repeat this process until an update does not change the datapoint assignments, in which case we have converged.

That's all. It's really a simple algorithm.

## Mixture of Gaussians
The MoG algorithm extends the concepts of k-means to be probabilistic. We assume there are k Gaussian distributions which generate this data and each data point is generated by one of the Gaussians. We model these gaussians and at test time given an input x assign it to the gaussian it is most likely from.

Each of the gaussians have 3 parameters:

- Mean vector ($\mu$) : n dimensional where n is feature size
- Covariance Matrix($\Sigma$): n x n matrix
- Mixing Weight($\pi$): Probability that a certain data point is generated by this gaussian. Reflects size of the cluster

We suppose there is a latent(unobserved) variable $z \in [1,K]$ which determines which gaussian generated a given point. To fit the parameters of each gaussian we use an iterative algorithm called Expectation Maximisation. It is similar in concept to gradient descent but it's used to fit models which depend on latent variables. EM consists of two steps: An Expectation(E-Step) and a Maximisation(M-Step).

Initially we randomly initialise the parameters of each model and in the E-step we use the current parameters of the model to compute the probability of each gaussian generating a given point:

    - $P(Z_i=k | X_;\theta) = \frac{P(X_i | Z_i=k;\theta)P(Z_i=k;\theta)}{P(X_i;\theta)}$ - [[Bayes Rule]]
	- $P(X_i|z_i=k) \rightarrow$ Obtained using Gaussian pdf with current parameters
	- $P(Z_i=k;\theta) \rightarrow$ $\pi_k$ 
	- $P(X_i;\theta)\rightarrow$ Obtained by weighted sum of all gaussians.

In the M step we update the parameters of the model to maximise the Expected log likeligood  $Q(\theta)$. 

	- $Q(\theta) = \sum_i^n\sum_{k=1}^KP(z_i=k | x_i)log(\pi_kN(X_i;\mu,\Sigma))$
	- Taking derivatives with respect to each parameter and setting to 0 we get:
	- $\mu_k = \frac{\sum_i^nP(z_i=k | x_i)x_i}{\sum_i^nP(z_i=k | x_i)}$ $\rightarrow$ Weighted Average of all datapoints
	- $\Sigma_k = \frac{\sum_i^nP(z_i=k | x_i)(x_i - \mu_k)(x_i - \mu_k)^T}{\sum_i^nP(z_i = k | X_i)}$ $\rightarrow$ Weighted Average of squared deviations from mean
	- $\pi_k = \frac{1}{n}\sum_i^nP(z_i=k | x_i)$ $\rightarrow$ Normalised probability of belonging to this gaussian
## Principal Component Analysis
Suppose you have a dataset with 100 features; are all one hundred of them really relevant and necessary? The answer is probably not. 

PCA is a dimensionality reduction algorithm. Given features in high dimensions it finds the lower-dimensional subspace onto which to project the data points while maintaining the variance between the data points and hence the meaning.

First PCA computes the Covariance Matrix of the features:

$$\Sigma = \frac{1}{m} \sum_{i=1}^m x^{(i)} (x^{(i)})^T$$

This Covariance matrix holds information about how each pair of variables vary together.

PCA then finds the eigenvectors of the Covariance matrix and those are the principal components of the dataset.

$$\Sigma u_i = \lambda_i u_i$$

Eigenvectors are the vectors that do not change direction under the effect of a transformation.

Try to think about what all of this means. We have some data, find the covariance relationship between each pair of variables then find the eigenvectors of the covariance matrix. Those eigenvectors serve as a basis for the lower-dimensional subspace onto which the data can be projected and still maintain its variance and meaning.

## Factor Analysis
Factor Analysis is the probabilistic cousin of PCA. Suppose we have a dataset with m examples and n features where m is much less than n (small data problems). Fitting any algorithm to this type of data is quite challenging as we just do not have enough data to do so. 

Factor Analysis works great with this type of data; it assumes there are hidden factors that drive the observed features. For example, consider a dataset of marks students got in different exams:

| Student | Math | Physics | History | Lit |
|:-------:|:----:|:-------:|:-------:|:---:|
|    1    |  95  |    90   |    60   |  55 |
|    2    |  50  |    55   |    88   |  92 |

What Factor Analysis does is assume there is a very small set of hidden features that drive the observed features. In this example the hidden features are Math ability and Linguistic Ability. We can then model all those features as a linear combination of only these 2 features, greatly simplifying our problem.

The goal of Factor Analysis is not to make predictions or anything like that but rather to give us more information on the data. It can also be used as a preprocessing step for another algorithm. For example our dataset could now be:

$$x = \mu + \Lambda z + \epsilon$$

| Student | Logical Ability | Verbal Ability |
|:-------:|:---------------:|:--------------:|
|    1    |       High      |       Low      |
|    2    |       Low       |       High     |

And now we have a much easier time fitting other algorithms to this data.

So how do we find the values of $\Lambda$, z and $\Psi$? Again we get to use our friend Expectation Maximization (Factor Analysis is a standard case for EM).

In the E step we start with random estimates of $\Lambda$ and $\Psi$, then assume $p(z\mid x_n)$ is Gaussian and compute it using the Gaussian PDF.

In the M step we update the values of $\Lambda$ and $\Psi$:

$$\Lambda = \left[ \sum_{i=1}^m (x^{(i)} - \mu) \mathbb{E}[z^{(i)}\mid x^{(i)}]^T \right] \left[ \sum_{i=1}^m \mathbb{E}[z^{(i)}z^{(i)T}\mid x^{(i)}] \right]^{-1}$$

$$\Psi = \frac{1}{m} \text{diag} \left( \sum_{i=1}^m (x^{(i)} - \mu)(x^{(i)} - \mu)^T - \Lambda \mathbb{E}[z^{(i)}\mid x^{(i)}] (x^{(i)} - \mu)^T \right)$$


$$p(z\mid x) = \mathcal{N}(\mu_{z\mid x}, \Sigma_{z\mid x})$$

Factor Analysis is essentially just probabilistic PCA.

# Other Concepts
Other than the algorithms there are other concepts that are important in ML and those are: Normalization, feature selection, bias and variance tradeoff, regularization and debugging. Let's briefly talk about them here.

## Normalization
We have already seen how ML algorithms operate and make predictions; they essentially use data to build a mathematical model that accurately makes predictions as defined by some objective function. 

These operations happen to be very sensitive to the scale of the numbers being used. For example, if we have two features — salary of an employee ranging from 50k to 200k and age ranging from 18 to 65 — using these values as-is will lead to the salary overly dominating the gradient computations and our model will think salary is much more relevant than age when in fact it is not; it just operates on a different scale.

Normalization is the process of bringing all our features to the exact same scale so that none dominates the other. There are 2 main ways of normalizing features:

### Z-score Normalisation

$$x' = \frac{x - \mu}{\sigma}$$

### Min-Max Normalization

$$x' = \frac{x - \min(x)}{\max(x) - \min(x)}$$

## Feature Selection and Engineering
For some prediction tasks not all the features available in the data are relevant for the prediction; they may be dropped without harming the result of our predictions.

Feature selection is the process of deciding which features are relevant for our task.

Feature engineering is one step ahead of that, where we come up with our own features we think will lead to making better predictions. It can be a combination of existing features or a brand new feature. Doing feature engineering requires domain knowledge or thorough statistical analysis of the data.

## Bias and Variance Tradeoff

This is the major problem of ML models; when they do not find the optimal model they tend to either underfit or overfit. Underfitting means the model did not learn the patterns in the data properly and hence makes wrong predictions. Overfitting on the other hand means the model memorised the patterns in the training data and cannot generalise to unseen data.

```
Error
  ^
  | \          /  <-- Test Error
  |  \        /
  |   \      /
  |    \____/
  |     \
  |      \__________ <-- Training Error
  |
  |____________________> Model Complexity
   Underfit   Optimum    Overfit
```

The symptom for underfitting is when the training loss is very high, meaning the model cannot make predictions even on the training data it is seeing.

```
Loss
  ^
  |------------------ Test Loss
  |
  |------------------ Training Loss
  | (Both High)
  |____________________> Epochs
```

The symptom for overfitting is very low training loss but high test loss, which means the model does very well on the data it has memorised but cannot do well on unseen data.

```
Loss
  ^
  |        __________ Test Loss (High)
  |      /
  |     /
  |____/_____________ Training Loss (Low)
  |____________________> Epochs
```

The reasons for underfitting include:
    - The model is too simple. Like fitting a straight line to data that has a quadratic relationship between features and target.
    - Not enough training data.
    - Trained for too few iterations.
    - Learning rate too high.

The reasons for overfitting include:
    - The model is too complex. Like fitting a polynomial to linear data.
    - Trained for too long.

## Regularization
Regularization is the main technique used to prevent overfitting. The idea is very simple: to avoid overfitting we add a term to our loss function that forces the weights to be smaller.

$$J_{reg}(\theta) = J(\theta) + \lambda \sum_{j=1}^{n} \theta_j^2 \quad \text{(L2 / Ridge)}$$
$$J_{reg}(\theta) = J(\theta) + \lambda \sum_{j=1}^{n} |\theta_j| \quad \text{(L1 / Lasso)}$$

## Debugging

Many times our implementations of an ML algorithm for a certain task will not work. When that happens how can we do the debugging process? This is more of an art than a science but here are some general tips to follow:

- Perform a proper bias vs variance analysis. Plot both the training and test error and look at them carefully. You should be able to identify if it is an underfitting or an overfitting problem.

If it's an overfitting problem:
    - Simplify the model (use fewer features)
    - Add regularization
    - Is your algorithm suitable?
    - Are your hyperparameters correct?

If it's an underfitting problem:
    - Get more data
    - Add more features
    - Do more training iterations

If the above does not work, here are some more things you can do:

You have to determine if your optimisation algorithm is not converging or if it is converging but you are optimising the wrong loss function. If you used a loss function J and converged to weights W1, try a separate algorithm with a different loss function $\alpha$ which will converge to weights W2. If $\alpha(W1) > \alpha(W2)$ and $J(W1) > J(W2)$ this means your loss function is not the problem — the algorithm fails to converge to optimal weights. Check the learning rate and maybe the training data. On the other hand if $\alpha(W1) > \alpha(W2)$ and $J(W1) < J(W2)$ then you are optimising the wrong loss function. Change it.

# Conclusion
That was our brief tour of the ML landscape. Thanks for reading and sticking through. Stay tuned for the RL and Neural Network articles coming soon. See ya.

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>
<script>
  window.MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']],
      displayMath: [['$$', '$$'], ['\\[', '\\]']]
    }
  };
</script>
