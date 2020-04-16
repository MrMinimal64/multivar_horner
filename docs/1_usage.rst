.. _usage:

=====
Usage
=====



.. note::

    For a more detailed documentation of all the features please confer to the API documentation or the code directly.

.. TODO link



Horner factorisation polynomial representations
--------------------------


to create a representation of a multivariate polynomial in Horner factorisation:

.. code-block:: python

    import numpy as np

    # input parameters defining the polynomial
    #   p(x) = 5.0 + 1.0 x_1^3 x_2^1 + 2.0 x_1^2 x_3^1 + 3.0 x_1^1 x_2^1 x_3^1
    #   with...
    #       dimension N = 3
    #       amount of monomials M = 4
    #       max_degree D = 3
    # NOTE: these data types and shapes are required by the Numba jit compiled functions
    coefficients = np.array([[5.0], [1.0], [2.0], [3.0]], dtype=np.float64)  # numpy (M,1) ndarray
    exponents = np.array([[0, 0, 0], [3, 1, 0], [2, 0, 1], [1, 1, 1]], dtype=np.uint32)  # numpy (M,N) ndarray

    from multivar_horner.multivar_horner import HornerMultivarPolynomial

    # Horner factorisation:
    # [#ops=10] p(x) = x_1^1 (x_1^1 (x_1^1 (1.0 x_2^1) + 2.0 x_3^1) + 3.0 x_2^1 x_3^1) + 5.0
    horner_polynomial = HornerMultivarPolynomial(coefficients, exponents)




pass ``keep_tree=True`` during construction of a Horner factorised polynomial,
when its factorisation tree should be kept after the factorisation process


.. code-block:: python

    horner_polynomial = HornerMultivarPolynomial(coefficients, exponents, keep_tree=True)



pass ``rectify_input=True`` to automatically try converting the input to the required ``numpy`` data structures
pass ``validate_input=True`` to check if input data is valid (e.g. only non negative exponents)

.. note::

    the default for both options is ``False`` for increased speed


.. code-block:: python


    coefficients = [5.0, 1.0, 2.0, 3.0]
    exponents = [[0, 0, 0], [3, 1, 0], [2, 0, 1], [1, 1, 1]]
    horner_polynomial = HornerMultivarPolynomial(coefficients, exponents, rectify_input=True, validate_input=True)



canonical form polynomial representation
----------------------------------------

to represent a polynomial without any factorisation:

.. code-block:: python

    from multivar_horner.multivar_horner import MultivarPolynomial
    polynomial = MultivarPolynomial(coefficients, exponents)



string representation
---------------------


in order to compile a string representation of a polynomial pass ``compute_representation=True`` during construction


.. code-block:: python

    polynomial = MultivarPolynomial(coefficients, exponents)
    print(polynomial) # [#ops=27] p(x)

    polynomial = MultivarPolynomial(coefficients, exponents, compute_representation=True)
    print(polynomial)
    # [#ops=27] p(x) = 5.0 x_1^0 x_2^0 x_3^0 + 1.0 x_1^3 x_2^1 x_3^0 + 2.0 x_1^2 x_2^0 x_3^1 + 3.0 x_1^1 x_2^1 x_3^1
    # NOTE: the number in square brackets indicates the number of operations required
    #   to evaluate the polynomial (ADD, MUL, POW).
    # NOTE: in the case of unfactorised polynomials many unnecessary operations are being done
    # (internally uses numpy matrix operations)


the formatting of the string representation can be changed with the parameters ``coeff_fmt_str`` and ``factor_fmt_str``:

.. code-block:: python

    polynomial = MultivarPolynomial(coefficients, exponents, compute_representation=True,
                        coeff_fmt_str='{:1.1e}', factor_fmt_str='(x{dim} ** {exp})')


the string representation can be computed after construction as well.


.. note::

    for ``HornerMultivarPolynomial``: ``keep_tree=True`` is required at construction time


.. code-block:: python

    polynomial.compute_string_representation(coeff_fmt_str='{:1.1e}', factor_fmt_str='(x{dim} ** {exp})')
    print(polynomial)
    # [#ops=27] p(x) = 5.0e+00 (x1 ** 0) (x2 ** 0) (x3 ** 0) + 1.0e+00 (x1 ** 3) (x2 ** 1) (x3 ** 0)
    #                   + 2.0e+00 (x1 ** 2) (x2 ** 0) (x3 ** 1) + 3.0e+00 (x1 ** 1) (x2 ** 1) (x3 ** 1)



change the coefficients of a polynomial
---------------------------------------

in order to access the polynomial string representation with the updated coefficients pass ``compute_representation=True``
with ``in_place=False`` a new polygon object is being generated


.. note::

    the string representation of a polynomial in Horner factorisation depends on the factorisation tree.
    the polynomial object must hence have keep_tree=True


.. code-block:: python

    new_coefficients = [7.0, 2.0, 0.5, 0.75]  # must not be a ndarray, but the length must still fit
    new_polynomial = horner_polynomial.change_coefficients(new_coefficients, rectify_input=True, validate_input=True,
                                                           compute_representation=True, in_place=False)




optimal Horner factorisation
----------------------------


cf. Readme: "Optimal Horner Factorisation"

.. TODO link!


pass ``find_optimal=True`` during construction of a Horner factorised polynomial
to start an adapted A* search through all possible factorisations:



.. note::

    BETA: untested feature


.. note::

    time and memory consumption is MUCH higher!

.. code-block:: python
    horner_polynomial_optimal = HornerMultivarPolynomial(coefficients, exponents, find_optimal=True,
                                                         compute_representation=True, rectify_input=True,
                                                         validate_input=True)




caching polynomials
-------------------


export

.. code-block:: python

    path = 'file_name.pickle'
    polynomial.export_pickle(path=path)


import

.. code-block:: python
    from multivar_horner.multivar_horner import load_pickle
    horner_polynomial = load_pickle(path)




evaluating a polynomial
-----------------------

in order to evaluate a polynomial at a point ``x``:


.. code-block:: python
    # define a query point and evaluate the polynomial
    x = np.array([-2.0, 3.0, 1.0], dtype=np.float64)  # numpy (1,N) ndarray
    p_x = polynomial(x) # -29.0


or


.. code-block:: python

    p_x = polynomial.eval(x)  # -29.0






computing the partial derivative of a polynomial
------------------------------------------------


.. note::

    BETA: untested feature


.. note::

    partial derivatives will be instances of the same parent class



.. note::

    all given additional arguments will be passed to the constructor of the derivative polynomial


.. note::

    dimension counting starts with 1 -> the first dimension is #1!


.. code-block:: python

    deriv_2 = polynomial.get_partial_derivative(2, compute_representation=True)
    # [#ops=5] p(x) = x_1 (x_1^2 (1.0) + 3.0 x_3)




computing the gradient of a polynomial
------------------------------------------------

.. note::

    BETA: untested feature



.. note::

    all given additional arguments will be passed to the constructor of the derivative polynomials



.. code-block:: python

    grad = polynomial.get_gradient(compute_representation=True)
    # grad = [
    #     [#ops=8] p(x) = x_1 (x_1 (3.0 x_2) + 4.0 x_3) + 3.0 x_2 x_3,
    #     [#ops=5] p(x) = x_1 (x_1^2 (1.0) + 3.0 x_3),
    #     [#ops=4] p(x) = x_1 (x_1 (2.0) + 3.0 x_2)
    # ]