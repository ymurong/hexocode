---
title: Machine Learning with Python 1 - Linear Regression
date:  2020-10-14 12:00:00
tags:
- machine learning
---
> As the most common way of addressing regression issues, linear regression model stands out for its simple mathematical idea and easy implementation in computer language. The results of linear regression model has a strong interpretability and a clear realistic meaning. It is even the fundamentals of many non linear models such as logistics regression and support vector machines.

# Simple Linear Regression

![image.png](/img/2020/10/image-fc530ab885724b30a4fcc10dd591a3bd.png)

Linear regression attempts to model the relationship between two variables by fitting a linear equation to observed data. One variable is considered to be an explanatory variable, and the other is considered to be a dependent variable. For example, a modeler might want to relate the weights of individuals to their heights using a linear regression model.


## Mathematical Background
The idea of linear regression model is to minimize the errors between the predicted value $\hat{y}$ and the real value y.

{% katex %}
|{y}^{(i)}-\hat{y}^{(i)}|
{% endkatex %}

Unfortunately, this function is not differentiable at 0, hence in linear regression scenario, we use another function:
{% katex %}
\sum_{i=0}^{m} ({y}^{(i)}-\hat{y}^{(i)})^{2}
{% endkatex %}

The **objective** here is find the a and b so as to minimize the function:

{% katex %}
\sum_{i=1}^{m} ({y}^{(i)}-\hat{y}^{(i)})^{2}     \Rightarrow 
\sum_{i=1}^{m} ({y}^{(i)}- ax^{(i)}-b)^{2}
{% endkatex %}

> This type of expression could also be called as **loss function**, which is usally used in optimization problem. In machine learning, it is very common to find a loss function or utility function by analysing the problem in order to acquire the the machine learning model

Then by caculcating the partial differentiations:
{% katex %}
\frac{\partial{J}(a,b)}{\partial{b}} = \sum_{i=1}^{m}2({y}^{(i)}- ax^{(i)}-b)(-1) = 0
$$

$\Rightarrow$

$$
\sum_{i=1}^{m}{y}^{(i)}- \sum_{i=1}^{m}ax^{(i)}-mb = 0
$$

$\Rightarrow$
$$
b = \overline{y} -a\overline{x}
{% endkatex %}



Similarly,
{% katex %}
\frac{\partial{J}(a,b)}{\partial{a}} = \sum_{i=1}^{m}2({y}^{(i)}- ax^{(i)}-b)(-x^{(i)}) = 0
$$

$\Rightarrow$

$$
a= \frac{\sum_{i=1}^{m}(x^{(i)}y^{(i)}-x^{(i)}\overline{y})}{\sum_{i=1}^{m}((x^{(i)})^2)-\overline{x}x^{i})}
$$

$\Rightarrow$

$$
a= \frac{\sum_{i=1}^{m}(x^{(i)}-\overline{x})(y^{(i)}-\overline{y})}{\sum_{i=1}^{m}(x^{(i)}-\overline{x})^{2}}
{% endkatex %}

By knowing a and b, we could then express the prediction formule
```math 
$$
\hat{y} = ax^{(i)} + b 
{% endkatex %}

by 

{% katex %}
\hat{y} = \frac{\sum_{i=1}^{m}(x^{(i)}-\overline{x})(y^{(i)}-\overline{y})}{\sum_{i=1}^{m}(x^{(i)}-\overline{x})^{2}}x^{(i)} + \overline{y} -\frac{\sum_{i=1}^{m}(x^{(i)}-\overline{x})(y^{(i)}-\overline{y})}{\sum_{i=1}^{m}(x^{(i)}-\overline{x})^{2}}\overline{x}
{% endkatex %}

## Python Implementation

Normally we could implement the method like the following:
```python
def fit(self, x_train,y_train):
	x_mean = np.mean(x_train)
        y_mean = np.mean(y_train)

        numerator = 0.0
        denominator = 0.0

        for x, y in zip(x_train, y_train):
		numerator += (x - x_mean)(y - y_mean)
		denominator += (x - x_mean) ** 2


        self.a = numerator/denominator 
        self.b = y_mean - x_mean * a

def predict(self, x):
    return self.a*x_predict + self.b
```

Alternatively, we have another way with better performance, which is treat the problem from vector perspective.

By looking closely at the expression:

{% katex %}
a= \frac{\sum_{i=1}^{m}(x^{(i)}-\overline{x})(y^{(i)}-\overline{y})}{\sum_{i=1}^{m}(x^{(i)}-\overline{x})^{2}}
{% endkatex %}

we could find that the numerator and the denominator are actually the dot product of vectors. hence we could simply implement the fit method as following:

```python
def fit(self, x_train,y_train):
	x_mean = np.mean(x_train)
        y_mean = np.mean(y_train)

        numerator = (x_train-x_mean).dot(y_train-y_mean)
        denominator = (x_train-x_mean).dot(x_train-x_mean)

        self.a = numerator/denominator 
        self.b = y_mean - x_mean * a
```
By using vertors, the performance could be drastically improved. And of course, we could also directly use scikit learn library.


## Best Regression Evaluation Metrics
The major flaw of RMSE and MSE is that we cannot do comparison with difference models. However, R squaired metric could achieve that.

{% katex %}
R^2 =  1- \frac{SSresidual}{SStotal} = 1 - \sum_{i=1}^{m}\frac{(\hat{y}^{(i)}-{y}^{(i)})^{2}}{(\overline{y}^{(i)}-{y}^{(i)})^2}
{% endkatex %}

>  1 -> Best fit
>  0 -> Same as Baseline model 
> <0 -> Worse than Baseline model or no correlation at all


# Multi Linear Regression

In real world cases, normally a sample could have many features. In this case, we would need to use multi linear regression.  

![image.png](/img/2020/10/image-e7c671299bed407ebc2fdb0a4ed0354b.png)

As illustrated in graph, for a given feature vector
{% katex %}
X^{(i)}=(X_1^{(i)},X_2^{(i)},...,X_n^{(i)})
{% endkatex %}

we need to predict the value y by applying the parameter vector

```math 
$$
\theta^{(i)}=(\theta_1^{(i)},\theta_2^{(i)},...,\theta_n^{(i)})
{% endkatex %}

Hence we will have the following expression:

{% katex %}
\hat{y}^{(i)} = X^{(i)}\cdot\theta
{% endkatex %}

in matrix:

{% katex %}
\begin{pmatrix}
1 & X_1^{(1)} & X_2^{(1)} & ... & X_n^{(1)} \\
1 & X_1^{(2)} & X_2^{(2)} & ... & X_n^{(2)} \\
...  	    \\
1 & X_1^{(m)} & X_2^{(m)} & ... & X_n^{(m)}
\end{pmatrix}
\cdot
\begin{pmatrix}
\theta_1 \\
\theta_2 \\
...  	    \\
\theta_n
\end{pmatrix}
{% endkatex %}

Similarly, the objective here is find the vector $\theta$ so as to minimize the function:

{% katex %}
\sum_{i=1}^{m} ({y}^{(i)}-\hat{y}^{(i)})^{2}     \Rightarrow 
({y}-X_b\theta)^{T}({y}-X_b\theta)
{% endkatex %}
The derivation process is out of scope of colledge mathematics so here we only show the result.

{% katex %}
\theta=(X_b^{T}X_b)^{-1}X_b^Ty
{% endkatex %}


## Python Implementation

```python
def fit(self, x_train,y_train):
    X_b = np.hstack([np.ones(len(x_train),1),x_train])
    self._theta=np.linalg.inv(X_b.T.dot(X_b)).dot(X_b.T).dot(X_b)
    self._interception=self._theta[0]
    self.coef=self._theta[1:]
    return self
```
