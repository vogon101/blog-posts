---
layout: post
title: "Fixing Tensorflow-GPU on anaconda for Windows"
date: 2017-06-09
categories:
 - programming
 - neural-networks
 - machine-learning
 - tensorflow
 - tensorflow-gpu
 - anaconda
 - windows
 - windows10
---

Installing Tensorflow is pretty simple on windows, especially using anaconda: simply install anaconda, create a new environment and then install tensorflow:

<!--more-->

```cmd
conda create -n tensorflow python=3.5 
activate tensorflow
pip install --ignore-installed --upgrade tensorflow
```

However, the GPU version of tensorflow is more tricky. It requires that you install CUDA Toolkit (version 8.0 for the latest version of tensorflow). The problem comes that with some installs (My suspicion is that the problem arises when you create the anaconda environment *before* installing CUDA) you can get some very cryptic errors:

```python
ImportError: Traceback (most recent call last):
  File "C:\Users\Freddie\Anaconda3\envs\tensorflow\lib\site-packages\tensorflow\python\pywrap_tensorflow_internal.py", line 18, in swig_import_helper
    return importlib.import_module(mname)
  File "C:\Users\Freddie\Anaconda3\envs\tensorflow\lib\importlib\__init__.py", line 126, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
  File "<frozen importlib._bootstrap>", line 985, in _gcd_import
  File "<frozen importlib._bootstrap>", line 968, in _find_and_load
  File "<frozen importlib._bootstrap>", line 957, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 666, in _load_unlocked
  File "<frozen importlib._bootstrap>", line 577, in module_from_spec
  File "<frozen importlib._bootstrap_external>", line 938, in create_module
  File "<frozen importlib._bootstrap>", line 222, in _call_with_frames_removed
ImportError: DLL load failed: The specified module could not be found.

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "C:\Users\Freddie\Anaconda3\envs\tensorflow\lib\site-packages\tensorflow\python\pywrap_tensorflow.py", line 41, in <module>
    from tensorflow.python.pywrap_tensorflow_internal import *
  File "C:\Users\Freddie\Anaconda3\envs\tensorflow\lib\site-packages\tensorflow\python\pywrap_tensorflow_internal.py", line 21, in <module>
    _pywrap_tensorflow_internal = swig_import_helper()
  File "C:\Users\Freddie\Anaconda3\envs\tensorflow\lib\site-packages\tensorflow\python\pywrap_tensorflow_internal.py", line 20, in swig_import_helper
    return importlib.import_module('_pywrap_tensorflow_internal')
  File "C:\Users\Freddie\Anaconda3\envs\tensorflow\lib\importlib\__init__.py", line 126, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
ImportError: No module named '_pywrap_tensorflow_internal'


Failed to load the native TensorFlow runtime.

See https://www.tensorflow.org/install/install_sources#common_installation_problems

for some common reasons and solutions.  Include the entire stack trace
above this error message when asking for help.
```

The problem here is with the `PATH` environment variable. In this case PATH does not include the home directory of CUDA 8.0 (Normally `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\bin`). The problem becomes confounded by the fact that you might not want to add it to your general system path (you are using anaconda after all). One fix is to add an envornment variable named something like `CUDA_8_HOME`, set to the bin path and then add this code to before any of your tensorflow imports:

```python
import os
os.environ["PATH"] = os.environ["PATH"] + os.environ["CUDA_8_HOME"]
```

You can also just add the cuda directory to the system's path vairable using the windows dialogue box and that _should_ work ok


