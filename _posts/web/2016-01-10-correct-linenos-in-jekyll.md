---
layout: post
title: Correct linenos in Jekyll
tags:
- web
comments: true
keywords: "jekyll, code block, linenos"
---

I use kramdown as Jekyll's default markdown implementation. By default, line numbers in a code block are inline with the code. As a result, you'll get line numbers in your clipboard if you use "select and copy", which is quite annoying. A solution is to use `linenos=table`. However, there are some problems in aligning line numbers and codes, and the layout seems quit strange, as shown <a href="http://thanpol.as/jekyll/jekyll-code-highlight-and-line-numbers-problem-solved/">here</a>. This post also gives three possible solutions but they seems to be complicated for me since I'm quite fresh to web development.

Here I give my solution, which is based on <a href="http://botleg.com/stories/line-numbers-in-jekyll-code-blocks/">this post</a>. Add following code to *_syntax.scss*:

{% highlight css linenos %}
  .lineno { 
      color: #ccc; 
      display:inline-block; 
      text-align: right;
      width: 25px;
      border-right:1px solid #3a3742;
      padding-right: 2px;
      margin-left: -15px;
      -webkit-touch-callout: none;
      -webkit-user-select: none;
      -khtml-user-select: none;
      -moz-user-select: none;
      -ms-user-select: none;
      user-select: none;
  }
{% endhighlight %}

I make line numbers right-aligned. The last six lines are to prevent user from selecting line numbers.

***NOTE***:

Thanks to *yajunwf*, I noticed that line numbers are still copied to clipboard even though they are not highlighted. According to <a href="https://css-tricks.com/almanac/properties/u/user-select/">this post</a>, whether line numbers are copied depends on the web browser you use. 
