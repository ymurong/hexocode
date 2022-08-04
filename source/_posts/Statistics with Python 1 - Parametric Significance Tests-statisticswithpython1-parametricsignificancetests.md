---
title: Statistics with Python 1 - Parametric Significance Tests
date:  2021-08-25 12:00:00
tags:
- statistics
---
Parametric statistical tests assume that a data sample was drawn from a specific population distribution. They often refer to statistical tests that assume the Gaussian distribution. Because it is so common for data to fit this distribution, parametric statistical methods are more commonly used. If their assumptions are met, they have greater power than the non-parametric tests. Otherwise, non-parametric tests should be used. Thus, parametric tests should only be used after carefully evaluating whether the assumptions of the test are sufficiently fulfilled.

Code source here: https://github.com/ymurong/stats_python_playground

This table gives an overview of the most popular parametric tests:

|Test|Test for what?|
|-------|-------|
|One sample t-Test|mean of a given population|
|Two sample t-Test|mean difference of two independent populations|
|Paired t-Test|mean difference of two paired populations (not independent)|
|Chi-squared test|variance of a given population|
|F-Test|variance ratio of two independent population|


# One sample t-Test

For a given sample that satisfies the Gaussian distribution, based on its sample variance, we could know that whether the mean of this population is statistically different from a known or hypothesized value by doing one sample t-Test.

{% katex %}
T = \frac{\overline{X} - \mu}{S / \sqrt{n}} \sim t(n-1)
{% endkatex %}

According to the above formula, we could have the following python code to perform One sample t-Test.

```python
def t_test(data1: List[float], tail: TAIL = 'both',
           mu: float = 0, equal: bool = True) -> Tuple[float, float, float]:
    assert tail in ['both', 'left', 'right'], 'tail should be one of "both", "left", "right"'

    n1 = len(data1)
    mean_val = mean(data1)
    se = std(data1) / sqrt(n1)
    t_val = (mean_val - mu) / se
    df = n1 - 1

    return t_val, df, p
```

> mean() and std() represents the mean and standard deviaition function

# Two sample t-Test
For two independent samples that satisfy the Gaussian distribution, based on their sample variances, we could know that whether the mean difference of these two populations is statistically different from a known or hypothesized value by doing Two sample t-Test.

The assumption or null hypothesis of the test is that the means of two populations are equal. A rejection of this hypothesis indicates that there is sufficient evidence that the means of the populations are different, and in turn that the distributions are not equal.

If the total variance of given two populations are equal then we have:

{% katex %}
T = \frac{(\overline{X} - \overline{Y}) -(\mu_{1} - \mu_{2})}{S_{w}\sqrt{\frac{1}{n_{1}}+\frac{1}{n_{2}}}} \sim t(n1+n2-1)
{% endkatex %}

{% katex %}
S_{w} = \sqrt{\frac{(n1-1)S_{1}^2+(n2-1)S_{2}^2}{n1+n2-2}}
{% endkatex %}

otherwise, we have:
{% katex %}
T = \frac{(\overline{X} - \overline{Y}) -(\mu_{1} - \mu_{2})}{\sqrt{\frac{S_{1}^2}{n_{1}}+\frac{S_{2}^2}{n_{2}}}} \sim t(df=\frac{(S1^2/n1)^2}{n1-1}+\frac{(S2^2/n2)^2}{n2-1})
{% endkatex %}

According to the above formula, we could have the following python code to perform One sample t-Test. Notice that here  the equal parameter refers to whether the total variance of given two populations are equal or not, which will lead to different way of calculation.

```python
def t_test(data1: List[float], data2: List[float] = None, tail: TAIL = 'both',
           mu: float = 0, equal: bool = True) -> Tuple[float, float, float]:
    assert tail in ['both', 'left', 'right'], 'tail should be one of "both", "left", "right"'

    n1 = len(data1)
    n2 = len(data2)
    mean_diff = mean(data1) - mean(data2)
    sample1_var = variance(data1)
    sample2_var = variance(data2)

    if equal:
        sw = sqrt(((n1 - 1) * sample1_var + (n2 - 1) * sample2_var) / (n1 + n2 - 2))
        t_val = (mean_diff - mu) / (sw * sqrt(1 / n1 + 1 / n2))
        df = n1 + n2 - 2
    else:
        se = sqrt(sample1_var / n1 + sample2_var / n2)
        t_val = (mean_diff - mu) / se
        df_numerator = (sample1_var / n1 + sample2_var / n2) ** 2
        df_denominator = (sample1_var / n1) ** 2 / (n1 - 1) + (sample2_var / n2) ** 2 / (n2 - 1)
        df = df_numerator / df_denominator

    if tail == "both":
        p = 2 * (1 - t.cdf(abs(t_val), df))
    elif tail == "left":
        p = t.cdf(t_val, df)
    else:
        p = 1 - t.cdf(t_val, df)

    return t_val, df, p
```


# Paired t-Test
We may wish to compare the means between two data samples that are related in some way. For example, the data samples may represent two independent measures or evaluations of the same object. These data samples are repeated or dependent and are referred to as paired samples or repeated measures. Because the samples are not independent, we cannot use the Student’s t-test. Instead, we must use a modified version of the test that corrects for the fact that the data samples are dependent, called the paired Student’s t-test.

The test is simplified because it no longer assumes that there is variation between the observations, that observations were made in pairs, before and after a treatment on the same subject or subjects. The default assumption, or null hypothesis of the test, is that there is no difference in the means between the samples. The rejection of the null hypothesis indicates that there is enough evidence that the sample means are different.

{% katex %}
\frac{\overline{X_{diff}} - \mu_{diff}}{S_{diff} / \sqrt{n}} \sim t(n-1)
{% endkatex %}

According to the above formula, we could easily reuse the python implementation of the first one-sample t-test.

```python
def t_test_paired(data1: List[float], data2: List[float], tail: TAIL = 'both', mu: float = 0):
    """
    配对t检验
    """
    data = [e1 - e2 for (e1, e2) in zip(data1, data2)]
    return t_test(data, tail=tail, mu=mu)
```


# Chi-squared test

Chi-squared test could be used to test the variance of a given sample.

{% katex %}
\frac {(n - 1)S^ {2}}{\sigma ^ {2}}  \sim \chi ^ {2}(n -1)
{% endkatex %}
According to the above formula, we could have the following python code to perform One sample Chi-squared test.

```python
def chi2_test(data: List[float], tail: TAIL = "both", sigma2: float = 1):
    assert tail in ['both', 'left', 'right'], 'tail should be one of "both", "left", "right"'

    n = len(data)
    sample_var = variance(data)
    chi2_val = (n - 1) * sample_var / sigma2

    if tail == "both":
        p = 2 * min(1 - chi2.cdf(chi2_val, n - 1), chi2.cdf(chi2_val, n - 1))
    elif tail == "left":
        p = chi2.cdf(chi2_val, n - 1)
    else:
        p = 1 - chi2.cdf(chi2_val, n - 1)

    return chi2_val, n - 1, p
```

# F-Test
F-test could be used to test the variance ratio of two independent populations.

{% katex %}
\frac {S_{1}^ {2}/S_{2}^ {2}}{\sigma_{1}^{2}/\sigma _ {2}^ {2}} \sim F(  n_ {1}  -1,  n_ {2} -1)
{% endkatex %}

According to the above formula, we could have the following python code to perform Two sample Chi-squared test.

```python
def f_test(data1, data2, tail="both", ratio=1):
    n1 = len(data1)
    n2 = len(data2)

    sample1_var = variance(data1)
    sample2_var = variance(data2)

    f_val = sample1_var / sample2_var / ratio
    df1 = n1 - 1
    df2 = n2 - 1

    if tail == "both":
        p = 2 * min(1 - f.cdf(f_val, df1, df2), f.cdf(f_val, df1, df2))
    elif tail == "left":
        p = f.cdf(f_val, df1, df2)
    else:
        p = 1 - f.cdf(f_val, df1, df2)

    return f_val, df1, df2, p
```