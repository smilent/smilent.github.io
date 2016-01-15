---
layout: post
title: "Common String Operations in C++"
tags:
- cpp
comments: true
keywords: "c++, split"
---

* 
{:toc}

---
<br><br>

##split

The code below splits a given string `s` by any character in `delim`:

{% highlight cpp linenos %}
#include <iostream>
#include <string>
#include <vector>
#include <cstring>
using namespace std;

// s: the string that needs splitting.
// delim: the delimiters. Each containing charater(not the whole delim) is used to split s.
// return a vector, which contains all the tokens
vector<string> split(const string& s, string delim){
    //char* pch = strtok(strdup(s.c_str()), delim.c_str());
    char* pch = strtok(const_cast<char*>(s.c_str()), delim.c_str());
    vector<string> tokens;
    while(pch != NULL){
        string str(pch);
        tokens.push_back(str);
        pch = strtok(NULL, delim.c_str());
    }

    //free(pch); // must be called if strdup is used
    return tokens;
}
{% endhighlight %}

I adopt `strtok(char* str, const char * delimiters)`, which is in `<cstring>`. It accepts `char*` as the type of its first parameter. However, `string::c_str()` returns a pointer of type `const char*`. There are two ways to address the mismatch:

1. use `strdup()`, which accepts a `const char *` and returns a `char *`
2. use const_cast<> to cast `const char *` to `char*`

I tested these two methods. I split "this is for test lalal" by whitespace for 1000 times for each of the two method. Here is the total elapsed time (both including fileIO):

{% highlight bash %}
strdup: 0.006906
const_cast: 0.00841
{% endhighlight %}

I also copied the program for 1000 times into a file and split each line in that file by whitespace. Here is the result:

{% highlight bash %}
strdup: 0.110915
const_cast: 0.10048
{% endhighlight %}

It seems that for small strings, using `strdup()` is more efficient. However, `strdup()` is not a standard C++ function but a well-known POSIX function. What's more, `strdup()` makes a duplication of the original string, which means that you should use `free()` to release the memory after.

**I tested these two versions further, and found that their performance in efficiency are quite close. I think the previous results may contain bias. Anyway, choose either one you like.**
