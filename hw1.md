title: Homework 1
date: 09/03/2021
author: Xige Huang

# Math is fun - Euler Project

## Problem 7 - 10001st prime 
_solved by 424633_

**By listing the first six prime numbers: 2, 3, 5, 7, 11, and 13, we can see that the 6th prime is 13.**
**What is the 10 001st prime number?**

My intuition to approach this problem is to check if each number is prime, and record the index of the prime numbers and finally return the 10001st prime; in practice, we only need to check if an odd number is prime because even numbers are not prime by definition. 

More specifically:
If i is divisible by a divisor `div`, then it is not prime and the loop moves on to the next odd number i.
if i is not divisible by any odd divisor `div`, then it is a prime and we add 1 to the index.

Therefore the function can be defined as follows,

``` python
def count_prime(n):
    """Returns the nth prime number"""
    
    if n == 1:
        return 2
    index = 2
    for num in range(3, n**2, 2):
        div = 1
        while div*div < num:
            div += 2
            if num % div == 0:
                break
        else:
            index += 1
        if index == n:
            return num           
```

Run `count_prime(10001)` and then we get **104743**.



## Problem 39 - Integer right triangles
_solved by 73731_

**If p is the perimeter of a right angle triangle with integral length sides, {a,b,c}, there are exactly three solutions for p = 120: {20,48,52}, {24,45,51}, {30,40,50}.**
**For which value of p ≤ 1000, is the number of solutions maximised?**


From a^2 + b^2 = c^2 and a+b+c=p, we can derive that b = (p^2-2pa)/(2p-2a). 
Then we can search the triplets with given p, with a starting from 1.

To place a range on a, if we assume a<b, then 3a<a+b+c=p. Therefore we can get a<p/3.

Then we can define the function to calculate the pairs of {a, b, c} given restraint p.]

``` python
def get_sol(p):
    """Get the number of unique triplets of c^2 = a^2 + b^2 with given restraint a + b + c = p
    """
    
    n_sol = 0
    for a in range(1, p//3):
        b = (p**2-2*p*a)/(2*p-2*a)
        if int(b) == b:
            n_sol += 1
    return n_sol
```

Use the function above, we can define the main function to find p for which gives max number of function in the range 1<p<1000.

``` python
def find_max_sol(p):
    """Find p that a + b + c = p such that gives the maximum number of combination {a, b, c}
    """
    
    num_sol = []
    for p in range(1, 1001):
        num_sol.append(get_sol(p))
    max_at = num_sol.index(max(num_sol)) + 1
    return max_at
```

Run the function `find_max_sol(1000)` and get **840**.


## Problem 112 - bouncy numbers
_solved by 24427_


**Working from left-to-right if no digit is exceeded by the digit to its left it is called an increasing number; for example, 134468.**
**Similarly if no digit is exceeded by the digit to its right it is called a decreasing number; for example, 66420.**
**We shall call a positive integer that is neither increasing nor decreasing a "bouncy" number; for example, 155349.**
**Clearly there cannot be any bouncy numbers below one-hundred, but just over half of the numbers below one-thousand (525) are bouncy. In fact, the least number for which the proportion of bouncy numbers first reaches 50% is 538.**
**Surprisingly, bouncy numbers become more and more common and by the time we reach 21780 the proportion of bouncy numbers is equal to 90%.**
**Find the least number for which the proportion of bouncy numbers is exactly 99%.**


Follow by the description in the problem, we may check if a number with __d__ digits (d <= 3) is increasing with a marker `is_increasing`, which records a `True` if the left digit is less than the right digits. Similarly we can have a marker `is_decreasing`.

Starting from the left-most digit, if the number is increasing then the marker `is_increasing` should record `True` for d-1 times, and the marker `is_decreasing` should record `False` for d-1 times (for example, 3 times for each marker for 2578); they work similarly for decreasing numbers.


However, if `is_increasing` records `False` for at least once within the `True`'s (for example, 2145), it means that somewhere in the number the left digit is larger than the right digit (therefore not increasing anymore). Therefore, if the results for `is_increasing` and `is_decreasing` are not all `True` or `False`, respectively, the number is bouncy.


Follows the logic above, we first define the function to check if a given number is bouncy,

``` python
def is_bouncy(n):
    """Check if a given integer n is bouncy
    """
    
    digits = [int(x) for x in str(n)]
    is_inc, is_dec = False, False
    for idx in range (len(digits)-1):
        if (digits[idx] < digits[idx+1]):
            is_inc = True
        elif (digits[idx] > digits[idx+1]):
            is_dec = True
        if (is_inc and is_dec):
            return True
    return False
```

and then use the function to define the function that calculate the percent of bouncy numbers less than a given number.

``` python
def bouncy_perc(perc_lim, upper_lim = 200_0000):
    """Compute the bouncy percentage of every number below the given upper limit,
        and returns the number once the limit reaches the given p
    """
    
    n_bouncy = 0 
    count = 0
    for i in range(100, upper_lim):
        if(is_bouncy(i)):
            n_bouncy += 1
        bouncy_perc = n_bouncy / i*100
        if bouncy_perc == perc_lim:
            break
    return(i)
```

Run `bouncy_perc(90)` and get **21780**.