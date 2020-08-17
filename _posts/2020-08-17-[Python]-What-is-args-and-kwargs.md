---
layout: post
title: [Python] What is *args and **kwargs?
date: 2020-08-17
excerpt: "Let's learn about these two special syntaxes."
tags: [python, code]
comments: true
---



*args : to pass a variable number of arguments to a function. 

**kwargs : to pass a keywarded, variable-length argument list. 

### Example Code

{% highlight python %}

def plus(param, *args, **kwargs):

  result = 0

  print(args)

  #(1,2,3,1,2,3,1,2,3)

  print(type(args))

  #<class 'tuple'

  print(*args)

  #1,2,3,1,2,3,1,2,3

  print(type(*args))

  #TypeError: type() takes 1 or 3 arguments



  print(kwargs)

  #{'elfd': 123, 'leof': True, 'alwkr': 'awer'}

  print(type(kwargs))

  #<class 'dict'>

  for number in args:

​    result += number

  print(result)



plus(1,2,3,1,2,3,1,2,3,

​      elfd=123,leof=True,alwkr="awer")

{% endhighlight %}

