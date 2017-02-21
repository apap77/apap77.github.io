---
title: "Generating all DNA k-mers with python"
layout: post
comments: true
date: 2017-02-21
categories: snippet
---

{% highlight python %}
import itertools

bases = ['A', 'C', 'G', 'T']
def kmers(k):
    for kmer in itertools.product(bases, repeat=k):
        yield ''.join(kmer)

# USAGE
for kmer in kmers(k=2):
    print(kmer, end=' ')
# output : AA AC AG AT CA CC CG CT GA GC GG GT TA TC TG TT

{% endhighlight %}
