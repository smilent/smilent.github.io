---
layout: post
title: Common Python Operations
tag:
- python
comments: true
keywords: "python, time, file"
---

* toc
{:toc}
---

This post lists some common operations in Python such as file operation and time handling.
<br><br>

##File Operations

{% highlight python %}
try:
    with open('infile','r') as infile, open('outfile','r') as outfile:
        for line in infile:
            # handle each line in the file
            outfile.write(line.strip() + '\n')
except IOError:
    # do something
    
{% endhighlight %}

##Time Handling

`datetime` in [Python2.x](https://docs.python.org/2/library/datetime.html) or [Python 3.x](https://docs.python.org/3/library/datetime.html).

> Note that the format codes listed in `strftime` are slightly different from what is mentioned [here]({{site.baseurl}}/blogs/time-and-date-routines)
