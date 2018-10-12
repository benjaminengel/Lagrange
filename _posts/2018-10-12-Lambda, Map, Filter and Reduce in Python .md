---
layout: post
title: "Lambda, Map, Filter and Reduce in Python"
author: "Ben Engel"
categories: python
tags: basic
image: 
---
This is a small introduction to functional operators in python. 
The basic operators are:

* Lambda functions, sometime called anonymous function
* Map a basic function that map value to another value
* Filter, like the name suggests filters elements
* Reduce, applys a function to multiple elements and returns a value

For each of these types we are going to see a few example, and usecases.
But before we do so we are clearifing some terminology.

An iterable is an object that has an iter method, these are objects like lists, tuples, sets and so on.
A parameter is a variable in an function header, for example `x` is an parameter of the function `power` in the following example.
```python
def power(x): # here is x the parameter of power
    return x**2
```

## Lambda

Syntax 
```python
lambda parameter1,parameter2,...: function_code
```


```python
l = [1,2,3,4]
power = lambda x: x**2
l_prim = [power(x) for x in l]
l_prim
```




    [1, 4, 9, 16]



Here we can see the that the function `power` is applied to every element in l during list concatination.
I personally often use lambda functions with pandas apply.


```python
import pandas as pd
s = pd.Series(l)
s
```




    0    1
    1    2
    2    3
    3    4
    dtype: int64




```python
s.apply(lambda x: x**2)
```




    0     1
    1     4
    2     9
    3    16
    dtype: int64



## Map

Syntax
```python
map( function, iterable1,iterable2,...)
```

If we are not using pandas we can still have the need to apply a function to an iterable. Thats what we can do with map


```python
def power(x):
    return x**2

l = [1,2,3,4]
result = map(lambda x: x**2, l)
#or
result = map(power,l)
```

we just have to keep in mind that this proccedure alters the type of our iterable (keyword lazy evaluation)


```python
result
```




    <map at 0x7f8442fa93c8>



but we can cast it back to the original type


```python
list(result)
```




    [1, 4, 9, 16]



This is often usefull if we want to combine multiple lists.


```python
l1 = [1,2,3,4]
l2 = [10,20,30,40]
result = map(lambda x,y: x+y , l1, l2)
```


```python
list(result)
```




    [11, 22, 33, 44]



her we added the first element of the first list with the first element of the second list and so on.

## Filter

Syntax
```python 
filter(function, iterable1)
```

here we need a boolean function (a function that only returns True or False)
Now we filter a list so that only even elements will be returned.


```python
l = [1,2,3,4,5]
def is_even(n):
    if n % 2 == 0:
        return True
    return False
result = filter(is_even, l)
result
```




    <filter at 0x7f8442fa9e48>



here again we have lazy evaluation so we need to cat the result back to a list


```python
list(result)
```




    [2, 4]



## Reduce

Syntax
```python
reduce(function(parameter1,parameter2), itterable)
```


```python
from functools import reduce
l = [1,2,3,4,5,6]
result = reduce(lambda x,y: (x+y), l)
result
```




    21



which is equivalent to 


```python
summ = 0 
for num in l:
    summ = summ + num
summ
```




    21



there is also a buil in reduce function in python `sum`. Which leads to the same result as the code above.


```python
sum(l)
```




    21



Thats the end of the introduction, if you want to dig deeper in the topic I would suggest to take a close look at the module `functools`.
