---
title: Py
layout: page
permalink: /random/py/
---

### SVMLight

Loading SVMLight files in Python via `sklearn` is rather slow. Cache all the things:

{% highlight python %}
import logging
from pathlib import Path

import sklearn.datasets
import joblib

class SvmLightCache:
    def __init__(self, basedir='.cache'):
        if not basedir:
            raise ValueError('`basedir` is empty')
        location = str(Path(__file__).resolve().parent.parent / basedir)
        mem = joblib.Memory(location=location, verbose=logging.DEBUG)
        self.load_svmlight_file = mem.cache(sklearn.datasets.load_svmlight_file)
{% endhighlight %}


### Dirty Little Secret

In Python 2, there was a problem known as a [dirty little secret][dirty]:

{% highlight python %}
x = 'before'
a = [x for x in 1, 2, 3]
print x # this prints '3', not 'before'
{% endhighlight %}

this behaviour changed in Python 3 so that the loop control variable no longer
leaks out of the list comprehension scope.

[dirty]: http://python-history.blogspot.com/2010/06/from-list-comprehensions-to-generator.html

### Datetime

Python's default constructor does not include timezone information and hence
`isoformat` will not include this information (which is actually does not
follow the ISO 8601 specification).

Be sure to pass a timezone to `now`, etc:

{% highlight python %}
from datetime import datetime
from datetime import timezone

# No tzinfo
datetime.utcnow().replace(microsecond=0).isoformat()
# 2019-06-12T06:16:06

# With tzinfo
datetime.now(timezone.utc).replace(microsecond=0).isoformat()
# 2019-06-12T06:16:06+00:00
{% endhighlight %}


### Configuration

Treat dictionary keys as properties. Typing out the dictionary key syntax is
cumbersome.

{% highlight python %}
import time


class AttrDict(dict):
    def __init__(self):
        self.stamp = time.strftime("%Y%m%d.%H%M%S")
        self.log_fmt = '%(asctime)s: %(levelname)s: %(message)s'
        self.name = 'default-name'
        self.log_dir = 'log'
        self.model_dir = 'model'

    __setattr__ = dict.__setitem__
    # Raise correct exception type so pickle works
    # __getattr__ = dict.__getitem__
    def __getattr__(self, key):
      try:
        return self[key]
      except KeyError:
        raise AttributeError

    @classmethod
    def from_parseargs(cls, args):
        config = cls()
        config.merge_parseargs(args)
        return config

    @classmethod
    def from_attr_dict(cls, other):
        config = cls()
        for key, val in other.items():
            setattr(config, key, val)
        return config

    def merge_parseargs(self, args):
        for key, val in vars(args).items():
            # this class is a `dict` so use `in` instead of `hasattr`
            if key in self.keys() and val is not None:
                setattr(self, key, val)
{% endhighlight %}


### Prepend and append shortcuts with Numpy

{% highlight python %}
import numpy as np

a = np.arange(6).reshape((3,2))
b = np.c_[np.ones(3), a, np.zeros(3)]
{% endhighlight %}


### Jupyter Notebook

Prevent Lynx from mangling the keyboard on remote servers:

{% highlight bash %}
BROWSER=echo jupyter notebook
{% endhighlight %}
