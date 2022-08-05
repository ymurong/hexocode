---
title: Statistics with Python 3 - Parametric Significance Tests (Two way ANOVA)
date:  2021-09-10 12:00:00
tags:
- statistics
---
For two way ANOVA, we have one dependent variable and two independent variable.

# Preconditions 

The prerequisite of two way ANOVA is the same as one way ANOVA:

* inner-group Independence: 
  * random samples
  * sample size < 10% total population

* inter-group Independence: 
  * not pairwise

* Normal Distribution 
  * if sample size >= 10, then we could still use ANOVA if condition not met
  * if sample size < 10, we should use non parametric method if condition not met

* Homogeneity of variance test => variances of each group should be equal
  * if sample size of each group are equal then we could use ANOVA if condition not met
  * if sample size of each group are not equal, but the std ratio between the largest and smallest > 2, then we could still use ANOVA if condition not met


# Example 

DV: math score
IV: teaching method(r=2), scholarship(s=2)
Sample Size: nAN = nAY = nBN = nBY = 10; n = 40

|A|A|B|B|
|-------|-------|-------|-------|
|N|Y|N|Y|
|77|96|93|74|
|88|87|94|88|
|77|94|95|77|
|85|90|83|93|
|81|80|94|91|
|72|99|94|95|
|80|100|85|85|
|80|87|91|88|
|76|96|90|93|
|84|95|96|79|


1) calculate the average math result of each group (AN,AY,BN,BY)

{% katex %}
\overline{y}_{ij} = \frac {1}{n_{ij}} \sum_{k=1}^{n_{ij}}y_{ijk}
{% endkatex %}

2) calculate the total average of all math results
{% katex %}
\overline{y} = \frac {1}{n}\sum _ {i=1}^ {r}\sum _ {i=1}^ {s} n_{ij}*\overline{y}_ {ij}
{% endkatex %}

3) calculate the average of group A and group B
{% katex %}
\overline{y}_{i.} = \frac {1}{\sum_{j=1}^{s}n_{ij}}\sum_{i=1}^ {s} n_{ij}*\overline{y}_{ij}

\overline{y}_{.j} = \frac {1}{\sum_{j=1}^{r}n_{ij}}\sum_{i=1}^ {r} n_{ij}*\overline{y}_{ij}
{% endkatex %}

4) calculate the effect of teaching method
{% katex %}
\overline{y}_{i.} - \overline{y}
{% endkatex %}

5) calculate the effect of scholarship
{% katex %}
\overline{y}_{.j} - \overline{y}
{% endkatex %}

6) hypothesis of teaching method
* H0: μA = μB => μA - μ = μB - μ = 0 only when μA = μB = μ
* HA: μA ≠ μB => μA - μ, μB - μ are not all 0

7) hypothesis of scholarship
* H0: μN = μY => μN - μ = μY - μ = 0 only when μN = μY = μ
* HA: μN ≠ μY => μN - μ, μY - μ are not all 0

8) hypothesis of interaction effect
{% katex %}
\overline{y}_{ij} - \overline{y}_{i.} - \overline{y}_{.j} + \overline{y} = (\overline{y}_{ij} - \overline{y}) - (\overline{y}_{i.} - \overline{y}) - (\overline{y}_{.j} - \overline{y})
{% endkatex %}
* H0: μij - μi - μj + μ is equal to 0 to all i,j combinations
* H1: μij - μi - μj + μ is not equal to 0 to some i,j combinations

9) calculate the sum of squares total (SST)
{% katex %}
SS_{total} = \sum^{r}_{i=1}\sum^{s}_{j=1}\sum^{n_{ij}}_{k=1}(y_{ijk} - \overline{y})^2
{% endkatex %}

10) calculate the sum of squares for teaching method (SSmethod)
{% katex %}
SS_{method} = \sum^{r}_{i=1}ni.(\overline{y}_{i.} - \overline{y})^2
{% endkatex %}

11) calculate the sum of squares for scholarship(SSreward)
{% katex %}
SS_{reward} = \sum^{s}_{j=1}n.j(\overline{y}_{.j} - \overline{y})^2
{% endkatex %}

12) calculate the sum of squares for interaction effect
{% katex %}
SS_{M \times R} = \sum^{r}_{i=1}\sum^{s}_{j=1}n_{ij}(\overline{y}_{ij}-\overline{y}_{i.}-\overline{y}_{.j} + \overline{y})
{% endkatex %}

13) calculate the sum of squares error
{% katex %}
SS_{error} = \sum^{r}_{i=1}\sum^{s}_{j=1}\sum^{n_{ij}}_{k=1}(y_{ijk} - \overline{y}_{ij})^2
{% endkatex %}

14) calculate the mean of squares method
{% katex %}
MS_{method} = \frac{SS_{method}}{r-1} 
{% endkatex %}

15) calculate the mean of squares reward
{% katex %}
MS_{reward} = \frac{SS_{reward}}{s-1} 
{% endkatex %}

16) calculate the mean of squares interaction
{% katex %}
MS_{M \times R} = \frac{SS_{M \times R}}{(r-1) \times (s-1)} 
{% endkatex %}

17) calculate the mean of squares error
{% katex %}
MS_{error} = \frac{SS_{error}}{n - rs} 
{% endkatex %}

18) calculate the F-value
{% katex %}
F = \frac{MS_{method}}{MS_{error}} \sim {(dfm,dfe)}
F = \frac{MS_{reward}}{MS_{error}} \sim {(dfr,dfe)}
F = \frac{MS_{M \times R}}{MS_{error}} \sim {(dfm*r,dfe)}
{% endkatex %}

19) ANOVA Table

|-|Df|Sum sq| Mean sq | F value| Pr(>F)|  
|-------|-------|-------|-------|-------|-------|
|method|1|72.9|72.9|2.16|0.15|
|reward|1|129.6|129.6| 3.83 | 0.06 |
|Method:Reward|1|774.4|774.4| 22.91 | 2.9E-05 |
|Residuals|36|1215.0|33.8| - | - |

As a result, 

* math score is not influenced by teaching method
* math score is not influenced by scholarship reward
* math score is influenced by the interaction of teaching method and scholarship reward
  * the influence of teaching method on math score could be regulated by scholarship reward (A)
  * the influence of scholarship reward on math score could be regulated by teaching method (B)

> we need post hoc t-test to know exactly whether A is true or B is true



