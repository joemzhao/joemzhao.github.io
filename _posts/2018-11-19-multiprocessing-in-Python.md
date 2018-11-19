---
layout: post
title: "Multiprocessing and Code Migrations"
author: "Mengjie Zhao"
categories: study
tags: [programming]
image: python.png
---

Since the beginning of this year I started to heavily rely on the [multiprocessing](https://docs.python.org/2/library/multiprocessing.html) package provided by Python 2. One wrapper is also available in [joblib](https://github.com/joblib/joblib), which can be used to pipeline tasks easily. The main application case to me is that I need to train a large group of classifiers (just linear SVMs, what a relief!) in my master's thesis.

The package works great in my case -- it takes less than 1 hour to finish training around 800 classifiers on a machine with 48 cores. The dataset for each classifier consists of ~8k vectors with 200 dimensions and a 5-fold cross validation is applied for finding the best _C_.

The difficulty I met is how to organize the codes. Since the bound methods are not picklable: 
```
from __future__ import print_function

from multiprocessing import Pool

class Foo(object):
	def __init__(self):
		pass

	def foo(self, a):
		print ("Bonjour!")

myfoo = Foo()
pool = Pool(2)
pool.map(myfoo.foo, range(4))
pool.close()
pool.join()
```
```
Traceback (most recent call last):
cPickle.PicklingError: Can't pickle <type 'instancemethod'>: attribute lookup __builtin__.instancemethod failed
```
thus I have to define `Foo.foo` outside the class at the module level, which is ugly and unsafe. Some solutions are available on stackoverflow but most solutions can be grouped into two ways -- do it ugly or register the bound methods such that they are picklable. The latter may not be ideal either as reproducibility is crucial.

However, it seems `Foo.foo` can be paralleled in Python 3 directly (by making bound methods picklable):
```
from multiprocessing import Pool

class Foo(object):
	def __init__(self):
		pass

	def foo(self, a):
		print ("Bonjour!")

myfoo = Foo()
pool = Pool(2)
pool.map(myfoo.foo, range(4))
pool.close()
pool.join()
```
```
Bonjour!
Bonjour!
Bonjour!
Bonjour!
```
I've never think about migrating to Python 3, given the great compatibility managing libraries/modules like `six` and `__future__`, but it seems the `multiprocessing` would be a deal-breaker ...