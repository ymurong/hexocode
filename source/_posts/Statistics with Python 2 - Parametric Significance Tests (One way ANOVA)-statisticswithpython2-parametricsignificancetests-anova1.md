---
title: Statistics with Python 2 - Parametric Significance Tests (One way ANOVA)
date:  2021-09-01 12:00:00
tags:
- statistics
---
There are sometimes situations where we may have multiple independent data samples. We can perform the Student’s t-test pairwise on each combination of the data samples to get an idea of which samples have different means. This can be onerous if we are only interested in whether all samples have the same distribution or not. To answer this question, we can use the analysis of variance test, or ANOVA for short. 

ANOVA is a statistical test that assumes that the mean across 2 or more groups are equal. If the evidence suggests that this is not the case, the null hypothesis is rejected and at least one data sample has a different distribution.

The test requires that the data samples are a Gaussian distribution, that the samples are independent, and that all data samples have the same standard deviation. 

|Comparison|H0|Ha|
|-------|-------|-------|
|t-Test|μ1 = μ2|μ1 ≠ μ2|
|ANOVA|μ1 = μ2 = ... = μk|μ1,μ2,μ3...μk not all equal|


# One way ANOVA 
For one way ANOVA, we have one dependent variable and one independent variable. In order to measure the relationship between the dependent variable and the independent variable，we follow the formula below:

```
DV(delta) = IV(delta explanable)/other(delta inexplanable)
```

Example:

Suppose we have three different teaching methods along with the math result under each teaching method:

|A|B|C|
|-------|-------|-------|
|77|74|93|
|88|88|94|
|77|77|95|
|85|93|83|
|81|91|94|
|72|95|94|
|80|85|85|
|80|88|91|
|76|93|90|
|84|79|96|

1) calculate the average math result of each group (A,B,C)
{% katex %}
\overline {y}_{j} = \frac {1}{n_{j}} \sum_{i=1}^{n}y_{i}
{% endkatex %}

2) calculate the total average of all math results
{% katex %}
\overline{y} = \frac {1}{n}\sum _ {i=1}^ {k}\sum _ {i=1}^ {n} y_ {i}= \frac {1}{n}\sum _ {i=1}^ {k}n_ {j} \times\overline {y}
{% endkatex %}

3) calculate the sum of squares total (SST)
{% katex %}
SST = \sum _ {j=1}^ {k}\sum_{i=1}^{n_{j}}(y_{ij}-\overline{y})^2
{% endkatex %}

4) calculate the sum of squares group (SS between-groups, SS effect)
{% katex %}
SSG = \sum_{j=1}^{k}n_{j}(\overline{y}_{j}-\overline{y})^2
{% endkatex %}

5) calculate the sum of squares error(SSE, SS within-groups)
{% katex %}
SSE = \sum _ {j=1}^ {k}\sum_{i=1}^{n_{j}}(y_{ij}-\overline{y}_{j})^2
{% endkatex %}

6) calculate the MSG (mean of squares group) and MSE (mean of squares error)
{% katex %}
MSG = SSG / df_{G} = SSG / (k-1)
MSE = SSE / df_{E} = SSE / (n-k)
{% endkatex %}
> in fact, we have the below formula

{% katex %}
\frac{MSG}{MSE} \sim F(df_{G}, df_{E})
{% endkatex %}

7) ANOVA Table

|-|Df|Sum sq| Mean sq | F value| Pr(>F)|  
|-------|-------|-------|-------|-------|-------|
|Group|2|663.3|331.7|10.4|0.004|
|Residuals|27|860.6|31.9| - | - |

> as Pr < alpha then reject H0 => accept H1

8) multiple comparisons (post-hoc comparisons)
As ANOVA only told us that μ1,μ2,μ3 are not equal but we don't know which one is not equal to which one. Thus, we need to do multiple comparisons
```math
k*(k-1)/2 = 3 
```
> According to the above formula, we need to do 3 times t-test separately. In order to keep the alpha equal to 0.05, we need to calculate the new alpha with Bonferroni correction: a* = a/times = a/(k*(k-1)/2) = a/3 = 0.017

9) Individual t-Test for A and B, B and C, A and C

As the total variance of given two populations are equal then we should use the following formula:

{% katex %}
T = \frac{(\overline{X} - \overline{Y}) -(\mu_{1} - \mu_{2})}{\sqrt{MSE}\sqrt{\frac{1}{n_{1}}+\frac{1}{n_{2}}}} \sim t(df_{E})
{% endkatex %}

H0: μ1 = μ2  H1: μ1 ≠ μ2
{% katex %}
T = (80-91.5)-0/\sqrt{31.9/10 + 31.9/10} = -2.49 
{% endkatex %}
=> p = 2 * 0.0096 = 0.0019 > a* = 0.017
=> accept H0 

do the same for the rest two pairs



# One way ANOVA python implementation
Indeed, we could implement it by using python like the following:

```python
def anova_oneway(data: List[List[float]]):
    """
    单个factor的方差分析
    """
    k = len(data)
    assert k > 1

    group_means = [mean(group) for group in data]
    group_szs = [len(group) for group in data]
    n = sum(group_szs)
    assert n > k

    grand_mean = sum([group_mean * group_sz for group_mean, group_sz in zip(group_means, group_szs)]) / n

    sst = sum(sum((y - grand_mean) ** 2 for y in group) for group in data)
    ssg = sum((group_mean - grand_mean) ** 2 * group_sz for group_mean, group_sz in zip(group_means, group_szs))
    sse = sst - ssg

    dfg = k - 1
    dfe = n - k
    msg = ssg / dfg
    mse = sse / dfe

    f_value = msg / mse
    p = 1 - f.cdf(f_value, dfg, dfe)

    return f_value, dfg, dfe, p

if __name__ == '__main__':
    data = [
        [77, 88, 77, 85, 81, 72, 80, 80, 76, 84],
        [74, 88, 77, 93, 91, 95, 85, 88, 93, 79],
        [93, 94, 95, 83, 94, 94, 85, 91, 90, 96]
    ]
    print(anova_oneway(data))

    data1 = [77, 88, 77, 85, 81, 72, 80, 80, 76, 84]
    data2 = [74, 88, 77, 93, 91, 95, 85, 88, 93, 79]
    data = [data1, data2]
    print(t_test(data1, data2, tail="both", equal=True, mu=0))
    print(anova_oneway(data))
    print(t_test(data1, data2, tail="both", equal=True, mu=0)[0] ** 2)

output: 
(10.40448524285382, 2, 27, 0.00044671859193812224)
(-2.275127764963155, 18, 0.035360983213831965)
(5.176206346906239, 1, 18, 0.035360983213831965)
5.176206346906241
```

> We could see that when we have only two groups of data, one way ANOVA and t-Test will give the same results. In conclusion, t-Test is a special case of one way ANOVA


