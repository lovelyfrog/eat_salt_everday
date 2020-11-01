最近用pip3 intall的时候发现有的时候装的包很混乱，有的装在python3.9下，有的装在3.6下。而用pip3 unintall numpy的时候报错，说它不存在。这很奇怪，然后我发现了可以直接用python3 -m pip install/uninstall直接操作，就完全不会报错。

在装tensorboardX的时候，遇到一些warning说，numpy和一些包可以升级。于是我想着numpy确实好久没升级了，然后我就升级了一下，然后发现升级后import numpy总是会报错，提示：

```bash
Traceback (most recent call last):
  File "/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/numpy/core/__init__.py", line 22, in <module>
    from . import multiarray
  File "/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/numpy/core/multiarray.py", line 12, in <module>
    from . import overrides
  File "/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/numpy/core/overrides.py", line 7, in <module>
    from numpy.core._multiarray_umath import (
ModuleNotFoundError: No module named 'numpy.core._multiarray_umath'
```

然后我索引到`/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/numpy/core/` 发现目录下虽然有`_multiarray_umath....so `的文件，但是我发现前面的显示是39，我猜想这可能是因为它下了python3.9相关的包导致的。于是我又把numpy卸载后，重新装了一下，发现这时显示的就是36，也就不报错了。