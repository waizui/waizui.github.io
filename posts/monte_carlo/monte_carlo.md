<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

# Monte Carlo Method


##  How to estimate a integral

### Use Riemann Method

this is one way to estimate the value of a integral. but it convergence speed is very slow.

$\lim_{n \to \infty}  \frac{b-a}{n} \sum_{i=1}^{n} f(x_i) \Delta x = \int_a^b f(x) dx $

say if we want integral a gaussian distribution &f(x)& and we use riemann sum to estimate the value.
what are we going to do. We need chose a integer number n that determine how many fragments we want to split
the integral domain, then we estimate $f(x)$ at $\frac{1}{n},  \frac{2}{n}, \dots, 1$.  
the problem is: if we choose inappropriate n, for example, n is too small, we might miss the peak value of $f(x)$.
[pic]()
If we choose a very big n, it would takes very long time to finish the estimation.
[pic]()

### Time is money, we need a better solution.

There is another way of estimating a integral, you can imagine it as guessing:
I randomly chose a value in the integral domain and estimate the value of $f(x)$ at that point.
If I am lucky, the values I choose is very close to the mean value of $f(x)$, 
then I would get a satisfactory result after few guessing.

[pic]()

This is the general idea of Monte Carlo Method, which is a estimate method of integral that incorporate probability.


$E_p[f(x)] = \int_D f(x)p(x) dx$


$E[f(x)] = \frac{1}{N} \sum_{i=1}^{N} \frac{f(x_i)}{p(x_i)}$


$$

\begin{align}
    a &= b + c \\
    d &= e + f
\end{align}

$$
