---
layout: post
title: "How to install tensorflow-gpu on windows"
date: 2018-08-08
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

# How to install tensorflow-gpu on windows

Every time I try and start a new machine learning project it seems my `tensorflow-gpu` setup has broken, or is no longer compatible with X or has forgotten how to use my GPU or etc... This means I have to fight to reinstall it again and again which is always more of a fight than it should be. Tutorials are often out of date, incomplete or plain wrong. So here is how to install tensorflow GPU on Windows as of August 2018.

<!--more-->

I will be using Anaconda3 to do this so that I can keep multiple different python environments side by side easily. Whilst not officially supported, I have never had problems stem from my use of tensorflow under Anacondna. **This methid will also work with standard python installs**

### Step 1 - CUDA and cuDNN
The first step to using tensorflow GPU has nothing to do with python at all. First we need to install Nvidia CUDA and cuDNN. The key here is to get the EXACT correct versions of these tools installed otherwise tensorflow will not work. Currently, for tensorflow `1.9.0` we need CUDA 9.0 (NOT 9.2) and cuDNN 7.0 (NOT 7.1).

CUDA 9.0 can be downloaded from Nvidia [here](https://developer.nvidia.com/cuda-90-download-archive) and cuDNN from [here](https://developer.nvidia.com/rdp/cudnn-archive).

**In the future these required versions __will__ be different so check the tensorflow docs [here](https://www.tensorflow.org/install/install_windows)**

First install CUDA 9.0, then patch 4 (availiable from the same download page). Next download cuDNN and open the zip file then navigate into the `cuda` folder. This will contain three folders in each of which there is one file. These need to be moved as so:

```
cuda/lib/x64/cudnn.lib  -->  C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0\lib\x64\cudnn.lib
cuda/include/cudnn.h    -->  C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0\include\cudnn.h
cuda/bin/cudnn64_7.dll  -->  C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0\bin\cudnn64_7.dll
```

Effectively you're just copying the files into their respective locations in `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0`.

Finally to set this up add the path `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0\bin` to your %PATH% variable.

### Step 2 - Installing tensorflow-gpu

That was the hard part. Now all is left to do is actually install tensorflow.

```
conda create -n tensorflow python=3.6
activate tensorflow
pip install tensorflow-gpu
```

Hopefully that will work.

### Step 3 - Test it

To test it start a python prompt and run the following code:

```python
import tensorflow as tf
sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
hello = tf.constant('Hello, TensorFlow!')
print(sess.run(hello))
```

That should confirm you are running on a GPU device.