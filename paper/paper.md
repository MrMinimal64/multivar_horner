---
title: 'multivar_horner: A Python package for computing Horner factorisations of multivariate polynomials'
tags:
    - python
    - mathematics
    - polynomial
    - evaluation
    - multivariate
    - horner
    - factorisation
    - factorization


authors:
    - name: Jannik Michelfeit
      orcid: 0000-0002-1819-6975
      affiliation: "1, 2" # (Multiple affiliations must be quoted)
affiliations:
    - name: Technische Universität Dresden
      index: 1
    - name: Max Planck Institute of Molecular Cell Biology and Genetics
      index: 2
date: 20 April 2020
bibliography: paper.bib


---

# Summary


Polynomials are a central concept in mathematics and find application in a wide range of fields.
Thereby, one often asks for numerically stable and efficient evaluation schemes, which mostly rely on compact representations of (multivariate) polynomials [@LeeFactorization2013; @leiserson2010efficient; @Hecht1].

The `multivar_horner` Python package implements a multivariate Horner scheme ("Horner's method", "Horner's rule") [@horner1819xxi] and thereby allows computing Horner factorisations of multivariate polynomials.
Compared to the canonical form of polynomials this representation offers some important advantages.
The Horner factorisation is more compact in the sense that it requires less mathematical operations in order to evaluate the polynomial (cf. \autoref{fig:num_ops_growth}).
Consequently, evaluating a multivariate polynomial in Horner factorisation is faster and numerically more stable [@pena2000multivariate; @pena2000multivariate2; @greedyHorner] (cf. \autoref{fig:num_err_growth}).
These advantages come at the cost of an initial computational effort required to find the factorisation.

Accordingly the package presented here can be helpful always when (multivariate) polynomials have to be evaluated efficiently, the numerical error has to be small or a compact representation of the polynomial is required.



# Functionality

In one dimension there is only a single possible Horner factorisation of a polynomial.
In the multivariate case however the factorisation is ambiguous as there are multiple possible factors to factorise with.
The key functionality of `multivar_horner` is finding a good instance among the many possible Horner factorisations of a multivariate polynomial.

Let's consider the the example multivariate polynomial $p(x) = 5 + 1 x_1^3 x_2^1 + 2 x_1^2 x_3^1 + 3 x_1^1 x_2^1 x_3^1$.
The polynomial $p$ is the sum of $5$ monomials and has dimensionality $3$.
$p$ can also be written as $p(x) = 5 x_1^0 x_2^0 x_3^0 + 1 x_1^3 x_2^1 x_3^0 + 2 x_1^2 x_2^0 x_3^1 + 3 x_1^1 x_2^1 x_3^1$
The coefficients of the monomials are 5, 1, 2 and 3.

``multivar_horner`` allows to find a Horner factorisation of $p$ and to evaluate $p$ at a point $x$:

```python
from multivar_horner import HornerMultivarPolynomial
coefficients = [5.0, 1.0, 2.0, 3.0]
exponents = [[0, 0, 0], [3, 1, 0], [2, 0, 1], [1, 1, 1]]
p = HornerMultivarPolynomial(coefficients, exponents, rectify_input=True)
x = [-2.0, 3.0, 1.0]
p_x = p.eval(x, validate_input=True)
```

The found factorisation is $p(x) = x_1^1 (x_1^1 (x_1^1 (1 x_2^1) + 2 x_3^1) + 3 x_2^1 x_3^1) + 5$.

This is achieved by recursively factorising with respect to the most commonly used factor in all monomials (greedy heuristic described in [@greedyHorner]).
When no leaves of the resulting binary "Horner Factorisation Tree" can be factorised any more, a computational "recipe" for evaluating the polynomial is being compiled.
This recipe encodes all operations required to evaluate the polynomial in numpy arrays [@numpy].
Functions just in time compiled by Numba [@numba] enable computationally efficient polynomial evaluation.


# Degrees of multivariate polynomials


It is important to note that in contrast to the one dimensional case, several concepts of degree exist for polynomials in multiple dimensions.
Following the notation of [@trefethen2017multivariate] the usual notion of degree of a polynomial, the maximal degree, is the maximal sum of exponents of all monomials.
This is equal to the maximal $l_1$-norm of all exponent vectors of the monomials.
Accordingly the euclidean degree is the maximal $l_2$-norm and the maximal degree is the maximal $l_{\infty}$-norm of all exponent vectors.
Refer to [@trefethen2017multivariate] for precise definitions.

A polynomial is called fully occupied with respect to a certain degree if all possible monomials having a smaller or equal degree are present.
The occupancy of a polynomial can then be defined as the amount of existing monomials relative to the fully occupied polynomial of this degree.
A fully occupied polynomial hence has an occupancy of $1$.


![the amount of coefficients of fully occupied polynomials of different degrees in 3 dimensions.\label{fig:num_coeff_growth}](num_coeff_growth.png)


The amount of coefficients (equal to the amount of possible monomials) in multiple dimensions highly depends on the type of degree a polynomial has (cf. \autoref{fig:num_coeff_growth}).
This effect intensifies as the dimensionality grows.


# Benchmarks

![numerical error of evaluating randomly generated polynomials of varying sizes.\label{fig:num_err_growth}](../docs/_static/num_err_growth.png)


For benchmarking our method the following procedure is used:
In order to draw polynomials with uniformly random occupancy, the probability of monomials being present is picked randomly.
For a fixed maximal degree $n$ in $m$ dimensions there are $(n+1)^m$ possible exponent vectors corresponding to monomials.
Each of these monomials is being activated with the chosen probability.

For each maximal degree up to 7 and until dimensionality 7, 5 polynomials were drawn randomly.
In order to compute the numerical error, each polynomial has been evaluated at the point of all ones.
The true result in this case should always be the sum of all coefficients.
The resulting numerical error is being averaged over 100 tries with uniformly random coefficients in the range $[-1; 1]$.

![numerical error of evaluating randomly generated polynomials in canonical form relative to the Horner factorisation.\label{fig:num_err_heatmap}](../docs/_static/num_err_heatmap.png)

Note that even though the original monomials are not actually present in a Horner factorisation, the amount of coefficients however is identical to the amount of coefficients of its canonical form.
With increasing size in terms of the amount of included coefficients the numerical error of both the canonical form and the Horner factorisation found by `multivar_horner` grow exponentially (cf. \autoref{fig:num_err_growth}).
However, in comparison to the canonical form, the Horner factorisation is more numerically stable as it has also been visualised in \autoref{fig:num_err_heatmap}.

Even though the amount of operations required for evaluating the polynomials grow exponentially with their size irrespective of the representation, the rate of growth is lower for the Horner factorisation (cf. \autoref{fig:num_ops_growth}).
As a result, the Horner factorisations are computationally easier to evaluate.

![amount of operations required to evaluate randomly generated polynomials.\label{fig:num_ops_growth}](../docs/_static/num_ops_growth.png)

# Related work

The package has been created due to the recent advances in multivariate polynomial interpolation [@Hecht1; @Hecht2].
High dimensional interpolants of large degrees create the demand for evaluating multivariate polynomials computationally efficient and numerically stable.

Instead of using a heuristic to choose the next factor, one can allow a search over all possible Horner factorisations in order to arrive at a minimal factorisation.
The amount of possible factorisations, however, is increasing exponentially with the degree and dimensionality of a polynomial (the amount of monomials).
One possibility to avoid computing each factorisation is to employ a version of A-star search [@hart1968formal] adapted for factorisation trees.
This approach, which is similar to the branch-and-bound method suggested in [@kojima2008efficient, ch. 3.1], has been implemented by `multivar_horner`.


[@carnicer1990evaluation] shows how factorisation trees can be used to evaluate multivariate polynomials and their derivatives.

In [@kuipers2013improving] Monte Carlo tree search has been used to find more performant factorisations than with greedy heuristics.

Other representations of polynomials are being specified in [@LeeFactorization2013] and [@leiserson2010efficient].


# Further reading

The documentation of the package is hosted on [readthedocs.io](https://multivar_horner.readthedocs.io/en/latest/).

Any bugs or feature requests can be issued on [GitHub](https://github.com/MrMinimal64/multivar_horner/issues) [@github].
The [contribution guidelines](https://github.com/MrMinimal64/multivar_horner/blob/master/CONTRIBUTING.rst) can be found there as well.



# Acknowledgements

Thanks to Michael Hecht (Max Planck Institute of Molecular Cell Biology and Genetics) and Steve Schmerler (Helmholtz-Zentrum Dresden-Rossendorf) for valuable input enabling this publication.


TODO Reviewer and editor contributions, like any other contributions, should be acknowledged in the repository.

# References
