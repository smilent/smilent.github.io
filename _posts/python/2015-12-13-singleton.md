---
layout: post
title: Singleton in Python
comments: true
---

* generate table of contents
{:toc}
----
<br><br>

##Version 1

{% highlight python linenos %}
class Singleton:
	class _Singleton:
		def __init__(self, arg):
			self.val = arg

		def __str__(self):
			return `self` + str(self.val)
	
	instance = None

	def __init__(self, arg):
		if not Singleton.instance:
			Singleton.instance = Singleton._Singleton(arg)
		else:
			Singleton.instance.val = arg
	
	def __getattr__(self, name):
		return getattr(self.instance, name)

if __name__ == '__main__':
	x = Singleton('eggs')
	print x
	y = Singleton('sausage')
	print y

{% endhighlight %}

The output in Python 2.7 is :

{% highlight python %}
<__main__._Singleton instance at 0x109b10f80>sausage
<__main__._Singleton instance at 0x109b10f80>eggs
{% endhighlight %}

However, it doesn't work in Python 3.4. Here's the output:

{% highlight python %}
<__main__.Singleton object at 0x101b7a320>
<__main__.Singleton object at 0x101b7a400>
{% endhighlight %}

As we know, `print x`/`print(x)` displays `x.__str__`. Accessing `x.__str__` is delegated by the method `__getattr__()` defined in line 17 in Python 2.7. However, it doesn't hold for Python 3.4. I think the reason is that the class `Singleton` inherits `object` implicitly in Python 3.4. Since `object` contains the attribute `__str__`, `x__str__` gets `object.__str__` although `Singleton` does not define `__str__` explicitly. If we make `Singleton` inherit `object`, the output in Python 2.7 would be similar to the one executed in Python 3.4.

---

##Version 2

{% highlight python linenos %}
class Singleton(object):
	class _Singleton:
		def __init__(self, arg):
			self.val = arg

		def __str__(self):
			return `self` + str(self.val)

	instance = None

	def __new__(cls, arg):
		if not Singleton.instance:
			Singleton.instance = Singleton._Singleton(arg)
		else:
			Singleton.instance.val = arg
		return Singleton.instance

{% endhighlight %}

It works in both Python 2.7 and 3.4. But in Python 2.7, `Singleton` must inherit `object` explicitly so that `__new__()` can be called before the class object is created.

---

##Version 3

{% highlight python linenos %}
class Borg:
	_shared_state = {}
	def __init__(self):
		self.__dict__ = self._shared_state;

class Singleton(Borg):
	def __init__(self, arg):
		Borg.__init__(self)
		self.val = arg

	def __str__(self):
		return repr(self) + str(self.val)

if __name__ == '__main__':
	single = Singleton('sausage')
	print single
	single2 = Singleton('eggs')
	print single2
{% endhighlight %}

I found it in "Thinking in Python" written by Bruce Eckel: "Alex Martelli makes the observation that what we really want with a Singleton is to have a single set of state data for all objects". But in the strict sense, it is not Singleton. Here is the output:

{% highlight python %}
<__main__.Singleton instance at 0x1080feea8>sausage
<__main__.Singleton instance at 0x1080feef0>eggs
{% endhighlight %}

Please notice that the addresses of the two class objects are different.
