---
title: Py
layout: page
permalink: /py/svm/
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


### Datetime

Python's default constructor does not include timezone information and hence
`isoformat` will not include this information (which is actually non-conformant
to the ISO 8601 specification).

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


### Prepend and append shortcuts with Numpy

{% highlight python %}
import numpy as np

a = np.arange(6).reshape((3,2))
b = np.c_[np.ones(3), a, np.zeros(3)]
{% endhighlight %}
