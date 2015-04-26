---
layout: post
title:  "FRP Signal"
date:   2015-04-26 13:20:18
categories: jekyll update
---

最近在coursera上[Reactive Programming](https://www.coursera.org/course/reactive)，挺有意思的。刚做了week2的作业，看论坛上有人说这次作业太简单了，确实，因为做完我也不知道那个Signal怎么回事，也不需要知道是怎么回事，然后一步一步就搞完了。不过，不知道怎么回事肯定是不爽的嘛，还是看看。

看看源代码，很短，正好可以分析下。T_T但是我对于Scala并不了解。

{% highlight Scala %}
class Signal[T](expr: => T) {}
object Signal {}
{% endhighlight %}

我一个不懂Scala的人只能把这个理解为一个class，一个object两个不相关的东西了。。。但是发现class Signal代码里面又调用了object Signal里面的东西。于是只好去问更懂Scala的基友了，得到答复是这叫伴生对象，简单理解就是，Scala并没有static method，static变量什么的，于是就用object这个单例模式就达到了这个目的。

ok，这里看懂了，其实不懂的就是，每次Signal改变是怎么通知和他有关的东西的，那我们继续啦。

{% highlight Scala %}
protected def update(expr: => T): Unit = {
    myExpr = () => expr
    computeValue()
  }
{% endhighlight%}

上面是改变Signal值的代码，重点就是那个computeValue。

{% highlight Scala %}
  protected def computeValue(): Unit = {
    for (sig <- observed)
      sig.observers -= this
    observed = Nil
    val newValue = caller.withValue(this)(myExpr())
    /* Disable the following "optimization" for the assignment, because we
     * want to be able to track the actual dependency graph in the tests.
     */
    //if (myValue != newValue) {
      myValue = newValue
      val obs = observers
      observers = Set()
      obs.foreach(_.computeValue())
    //}
  }
{% endhighlight %}

可以看到在赋予Signal新值的时候，我们首先是把它从指向它的obsered里面删除掉。然后作为一个新值，还没有指向他的Signal，oberved = Nil

好，重点来了<code>val newValue = caller.withValue(this)(myExpr())</code>，这里我们使用了DynamicVariable，可以简单理解为一个线程安全的stack吧。计算时把上下文设置为当前Signal(withValue(this))。然后开始计算当前Signal的expr。

expr是个函数或者说就是一段代码，比如
{% highlight Scala %}
	a() + b() + 2
{% endhighlight %}

其中a，b也是Signal，在计算a(),b()的时候又会调用他们的apply方法，我们来看看apply

{% highlight Scala %}
  def apply() = {
    observers += caller.value
    assert(!caller.value.observers.contains(this), "cyclic signal definition")
    caller.value.observed ::= this
    myalue
  }
{% endhighlight %}

其中caller.value其实就是当前我们调用<code>call.withValue(this)(myExpr())</code>里面的this，比如<code>s() = a() + b() + 2</code>，这个caller.value就是s，因此s是a，b的observers,因为a，b的值改变都需要通知s。而a，b是s的observed。

所以我们在计算<code>call.withValue(this)(myExpr())</code>的同时也更新了this的observers，问题解决！


有人问

{% highlight Scala %}
def testA(text: Signal[Int]): Signal[String] = {
      val a =text().toString 
      Signal(a)
}

def testB(text: Signal[Int]): Signal[String] = Signal{
    text().toString
} 
{% endhighlight %}

testA,testB有啥区别，区别就是testA里面text这个Signal并不在你返回的Signal的expr里面，而只是一个常数a，因此text的observers里面不会有你返回的这个Signal，所以不论text怎么变，返回的Signal也不会有变化了。testB里面返回的Signal里面是包含text这个Signal的，所以text改变时会通知testB返回的这个Signal重新计算。