---
layout: post
title:  "Parser Combinator"
date:   2014-05-15 22:33:21
categories: jekyll update
---
<code>Parser Combinator</code>好久之前就听说过的东西。

开始觉得呢，这tm不就是递归下降么！为啥还要整的这么高级啊！

仔细看看就知道区别了，递归下降全部都要自己去写啊，好球麻烦啊！而且逻辑和代码是在一起的！Parser Combinator呢，我们只用去写每个小小的Parser，然后把他们combin在一起啦。逻辑什么的可以分出来了，写完感觉自己就是写了BNF而已！

标准的说Parser Combinator是一个higher-order函数，输入是parser然后返回值是新的parser！Parser就是input是个String，返回一些其他你需要的东西的东西！比如返回AST什么的。

我们用Haskell来玩下吧。

###Parser

{% highlight Haskell %}
newtype Parser a = Parser (String -> [(a , String)])
{% endhighlight %}

看这个Haskell的定义，input是String，返回一系列可能的值的list，如果parse失败肯定就是[]空列表了。

来定义个Parser看看
{% highlight Haskell %}
--搞掉第一个character
item :: Parser Char
item = Parser (\cs -> case cs of
                        "" -> []
                        (c:cs) -> [(c,cs)])
{% endhighlight %}


{% highlight Haskell %}
instancd Monad Parser where
     return a = Parser (\cs -> [(a,cs)])
     p >>= f = Parser (\cs -> concat [parse (f a') cs' | (a',cs') <- parse p cs])
{% endhighlight %}

作为monad当然要满足一些law啦
{% highlight Haskell %}
return a >>= f = f a
p >>= return = p
p >>= (\a -> (f a >>= g)) = (p >>= (\a -> fa)) >>= f
{% endhighlight %}
不只是Parser Monad，而是所有的Monad都该满足这些law。

顺便提一下，Haskell的do notation挺好用的，我们parser肯定是这样的啦
{% highlight Haskell %}
p1 >>= \a1 ->
p2 >>= \a2 ->
...
pn >>= \an ->
f a1 a2 ... an
{% endhighlight %}

等效的do notation是
{% highlight Haskell %}
do  a1 <- p1
    a2 <- p2
    ...
    an <- pn
    f a1 a2 ... an
{% endhighlight %}

直观了许多，do只是个语法糖而已，这两种写法其实是一样的，只是看起来do要舒服点。

###combinators

说了这么多还没到重点，combinator啊

像context-free languages一样，combinators有三个基本的combinator！

1. Alternation
{% highlight Haskell %}
instance MonadZero Parser where
    zero = Parser (\cs -> [])

instance MonadPlus Parser where
    p ++ q = Parser (\cs -> parse p cs ++ parser q cs)
{% endhighlight %}
2. Concatenation

3. Kleene star closure


