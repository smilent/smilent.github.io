---
layout: post
title: Compare Efficiency among if-else, ?:, and min()/max()
tag:
- cpp
comments: true
keywords: "c++,if-else, ? :, min/max, efficiency"
---

We often need to calculate the maximum/minimum value of a set of numbers. if-else, ?: and min()/max() are three methods we usually use to achieve the goal. In this post, I compare the efficiency among them. Here is the code:

{% highlight cpp linenos %}

#include <iostream>
#include <algorithm>
#include <time.h>
#include <vector>
#include <cstdlib>

using namespace std;

int main(){
    int pair_num;
    cout << "Please input the number of pairs to be compared:" << endl;
    cin >> pair_num;
    
    vector<vector<int>> pairs;
    for(int i = 0; i < pair_num; ++i){
        vector<int> pair;
        pair.push_back(rand());
        pair.push_back(rand());
        pairs.push_back(pair);
    }

    long result = 0;

    clock_t start = clock();
    for(auto pair: pairs){
        if(pair[0] > pair[1]){
            result += pair[0];
        }
        else{
            result += pair[1];
        }
    }
    double ifelse_time = (clock() - start) * 1.0 / CLOCKS_PER_SEC;

    start = clock();
    for(auto pair: pairs){
        result += (pair[0] > pair[1] ? pair[0] : pair[1]);
    }
    double op_time = (clock() - start) * 1.0 / CLOCKS_PER_SEC;

    start = clock();
    for(auto pair: pairs){
        result += max(pair[0], pair[1]);
    }
    double libfunc_time = (clock() - start) * 1.0 / CLOCKS_PER_SEC;

    cout << "ifelse cost: " << ifelse_time << endl;
    cout << "?: cost: " << op_time << endl;
    cout << "max() cost: " << libfunc_time << endl;
}
{% endhighlight %}

I generated 10,000,000 pairs (pair_num = 10,000,000). Here is the output:
{% highlight cpp %}
ifelse cost: 1.57152
?: cost: 1.55749
max() cost: 1.61714
{% endhighlight %}

