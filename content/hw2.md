title: Homework 2
date: 09/17/2021
author: Xige Huang

# Number theory and a Google recruitment puzzle

##The question:


Find the first 10-digit prime in the decimal expansion of 17π.

First solve sub-problems (and write unit tests for each of the three functions):

* Write a function to generate an arbitrary large expansion of a mathematical expression like π. Hint: You can use the standard library decimal or the 3rd party library sympy to do this
* Write a function to check if a number is prime. Hint: See Sieve of Eratosthenes
* Write a function to generate sliding windows of a specified width from a long iterable (e.g. a string representation of a number



__Outline of this post__:

* [a function that generates an arbitrary large expansion of a mathematical expression](#section1)
* [a function that check if a number is prime](#section2)
* [a function that generates sliding windows of a specified width from a long iterable](#section3)
* [main function](#section4)
* [results](#section5)


## expand_expr<a name="section1"></a>

First I seach on the documentations of `sympy`, and found that `N()` can be used to convert math expressions to floating-point approximation. Then I write the following function with `N()`.

```python
%%file expand_expr.py

from sympy import *
def expand_expr(n, expr, ndigit = 1000):
    """Return up to the given number of digits after decimal points of an expression"""
    return N(n*expr, ndigit+1)
```

unit test for `expand_expr()` to check if the function gives valid result for different inputs


```python
%%file test_expand.py

import unittest
from expand_expr import expand_expr
import math

class Testexpand(unittest.TestCase):
    def test_expand(self):
        self.assertEqual(str(expand_expr(1, math.pi, 4)), str(format(1*math.pi, '.4f')))
    
if __name__ == '__main__':
    unittest.main()
```

running
```python
! python3 -m unittest test_expand.py
```

in the code chunk giving the result of failure `AssertionError: '3.142' != '3.1416'`

The test failed because `N(n*expr, ndigit)` actually prints the math expression `n*expr` with a total of n digits instead of n digits after the decimal point. Then I modified the function, and added error message for type error of input n, and added more randomly chosen test cases.

The test cases check if the function `expand_expr` gives the correct result of the 4-digit expansion of pi, 1000-digit expansion of e, and 5-digit expansion of 3.3e, and also check if the `TypeError` can be raised under right conditions.

```python
%%file expand_expr.py

def expand_expr(n, expr, ndigit = 1000):
    """Return up to the given number of digits after decimal points of an expression"""
    
    if type(n) not in [int, float]:
        raise TypeError("Need to multiple a math expression by a float or an integer")
    
    return N(n*expr, ndigit+1)
```

```python
%%file test_expand.py

import unittest
from expand_expr import expand_expr
import math

class Testexpand(unittest.TestCase):
    def test_expand(self):
        self.assertEqual(str(expand_expr(1, math.pi, 4)), str(format(1*math.pi, '.4f')))
        self.assertEqual(str(expand_expr(1, math.e)), str(format(math.e, '.1000f')))
        self.assertEqual(str(expand_expr(3.3, math.e, 5)), str(format(3.3*math.e, '.5f')))
    
    def test_expand_input(self):
        self.assertRaises(TypeError, expand_expr, "5")
    
if __name__ == '__main__':
    unittest.main()
```

## is_prime<a name="section2"></a>

The most efficient function to check if a number is prime is as follows:

```python
%%file is_prime.py

import numpy as np
def is_prime(n):
    """Return True if a given number n is prime"""
    if n == 2:
        return True
    if n % 2 == 0 or n <= 1:
        return False

    for div in range(3, int(np.sqrt(n)) + 1, 2):
        if n % div == 0:
            return False
    return True
```
unit tests to check is `is_prime` gives correct result with several randomly chosen cases

```python
%%file test_prime.py

import unittest
from is_prime import is_prime

class Testprime(unittest.TestCase):   
    def test_prime(self):
        self.assertTrue(is_prime(2))
        self.assertTrue(is_prime(3))
        self.assertFalse(is_prime(27))
        self.assertTrue(is_prime(7427466391))
    
if __name__ == '__main__':
    unittest.main()
```


Running 

```python
! python3 -m unittest test_prime.py
``` 

gives the result
`
Ran 1 test in 0.004s

OK
`


Then we can move on to the next function.

## gen\_deci\_str<a name="section3"></a>
We want to write a function that first convert all digits after the decimal point to a list, and then return a modified list with the given starting decimal place of the given length.

```python
%%file gen_deci_str.py

def gen_deci_str(n, n_dec, n_len):
    """Returns a list of string of a number n 
       starting from its n_dec'th decimal place of length n_len
    """
    
    if type(n_len) != int or type(n_dec) != int:
        raise TypeError("n_len and n_dec need to be integers")
    
    return [str(x) for x in str(n-int(n))[n_dec+1:n_dec+n_len+1]]
```

unit tests
to check if `gen_deci_str(3.1415926, 3, 4)` returns `['1', '5', '9', '2']` and if the function `gen_deci_str` can return the correct 10 digit starting from the 6th decimal place of 2e.

```python
%%file test_gen.py

import unittest
from gen_deci_string import gen_deci_str
import math

class Testgen(unittest.TestCase):   
    def test_gen(self):
        self.assertEqual(gen_deci_str(3.1415926, 3, 4), ['1', '5', '9', '2'])
        str_e = format(math.e, '.20f')
        res_e = [str(x) for x in str(str_e)[6+1:6+1+10]]
        self.assertEqual(gen_deci_str(math.e, 6, 10), res_e)

    
if __name__ == '__main__':
    unittest.main()
```


give result that indicates there is nothing wrong with the function


## prime\_in\_expr<a name="section4"></a>
Finally we can write the function that gives the first n-digit prime in the given decimal expansion, using the three functions we have tested.


```python
from expand_expr import expand_expr
from is_prime import is_prime
from gen_deci_string import gen_deci_str

def prime_in_expr(n, expr, ndig, threshold = 1000):
    """Return the first prime of length ndig in an math expression n*expr
    """
    
    exp_deci = expand_expr(n, expr, threshold)
    
    for i in range(1, threshold + 1):
        num_list = gen_deci_str(exp_deci, i, ndig)
        num = int(''.join(num_list))
        if (int(num_list[-1])%2 != 0):
            if (is_prime(num)):
                return num
        i += 1
        
        if i == threshold - ndig:
            return("increase the digits of expansion")

```



One thing to highlight here is that if an integer ends with an even number, then it is not prime. Therefore we can be more efficient in checking the digits in the expansion by passing the `is_prime` function only if the n-digit ends with odd number.

We set the default threshold of expanding the given expression to 1000, that is if the first 1000 digit does not contain a n-digit prime, the function returns a message that ask you to increase the threshold.

## Running prime\_in\_expr<a name="section5"></a>
Given the first 5 digits in the decimal expansion of π are 14159, we can test the main function by running
```python
prime_in_expr(1, pi, 5)
```
which gives the correct result.

Furthermore, running 
```python
prime_in_expr(1, E, 10)
```
gives 7427466391, for which we know is the correct first 10-digit prime in the expansion e is 7427466391. One thing to note here is that we need to use `E` from `sympy` (which is the correct Euler's Number) instead of `math.e` or `numpy.exp(1)`.

Finally we can run
```python
prime_in_expr(17, pi, 10)
```
to solve the problem. The answer is 8649375157.