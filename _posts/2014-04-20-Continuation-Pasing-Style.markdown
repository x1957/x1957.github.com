---
layout: post
title:  "Continuation Passing Style(CPS)"
date:   2014-04-20 20:44:11
categories: jekyll update
---

Continuation Passing Style(CPS)
一个很有意思的东西

我的理解大概就是传递what is the next，下一步做啥作为了一个参数。

并且在最后函数结束的时候调用。

{% highlight Scheme %}

(define (fact n)
 (cond
   [(= n 0) 1]
   [(* n (fact (- n 1)))]))

{% endhighlight %}

一个简单的求阶乘的递归函数，如果用CPS风格写的话就是。

{% highlight Scheme %}

(defind (fact n k)
 (cond 
   [(= n 0) (k 1)]
   [(fact (- n 1) (lambda (ans) (k (* n ans))))]))

{% endhighlight %}

代码不一定对，随手写的，大概就是这么个意思。

https://cgi.soic.indiana.edu/~c311/lib/exe/fetch.php?media=cps-notes.scm

在这个lecture上面讲的比较清楚。


<code>Don't" sweat the small stuff!</code>

<code>Small sutff is stuff we know will terminate right away.</code>

<code>Don't sweat the small stuff if we know it will be evaluated.</code>

<code>Don't sweat the small sutff if it might be evaluated, but instread pass it to k.</code>

记住每个CPS风格的函数最后的参数都是continuate！

就是说你接下来要做的，就是说你这个函数结束要做的！

然后就是普通代码到CPS代码的转换T_T，这个我还需要看下。


下面是一个使用call/cc暴露continuation实现的一个简单的generator

{% highlight Scheme %}

#lang racket
(define call/cc call-with-current-continuation)

(define (gen lst)
  (define (get-next return)
    (for-each
     (lambda (ele)
             (call/cc (lambda(now)
                      (set! get-next now)
                      (return ele))))
     lst)
    (return 'end-of-the-list))
  (define (next)
    (call/cc (lambda (return)
               (get-next return))))
  next)


{% endhighlight %}