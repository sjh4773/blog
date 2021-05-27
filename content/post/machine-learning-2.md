---
title: 이미지 경계선 검출
date: 2021-03-28 21:00:24
categories:
  - machine_learning
tags:
  - machine_learning
---

```python
# image convolution
# check out horizontal, vertical, tilting, laplacian

import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

from PIL import Image # 파이썬에서 이미지를 담당

def convolution(image,filter,s=1): #stride = 1
    image = np.asarray(image)
    height = image.shape[0]
    width = image.shape[1]
    new_height = int((height - filter.shape[0])/s+1)
    new_width = int((width - filter.shape[1])/s+1)

    new_image = np.zeros((new_height,new_width))
    c_h = 0
    for h in range(new_height):
        c_w = 0
        for w in range(new_width):
            new_image[h,w] = np.sum(np.multiply(image[c_h:c_h+filter.shape[0],c_w:c_w+filter.shape[1]],filter))
            c_w += 1
        c_h += 1
    return new_image[:]

horizon_k = np.zeros((3,3))
vertical_k = np.zeros((3,3))
tilt_k = -np.ones((3,3))
laplacian_k = np.zeros((3,3))

# horizon filter
#  1  1  1
#  0  0  0
# -1 -1 -1
horizon_k[0,:] = 1
horizon_k[-1,:] = -1


# vetical filter
#  1  0 -1
#  1  0 -1
#  1  0 -1
vertical_k[:,0] = 1
vertical_k[:,-1] = -1

# 45 degree filter
# -1 -1  2
# -1  2 -1
# 2  -1 -1
tilt_k[0,-1] = 2
tilt_k[1,1] = 2
tilt_k[-1,0] = 2

# laplacian
# 0 -1 0
# -1 4 -1
# 0 -1 0
laplacian_k[0,1] = -1
laplacian_k[1,:] = -1
laplacian_k[1,1] = 4
laplacian_k[-1,1] = -1
```

```python
# example 1 = horizontal detecting
image = 255*np.ones((30,30))
image[:15,:] = 0 # 위는 어두움, 아래는 밝음
# plt.imshow(image,cmap='gray')
new_image = convolution(image,horizon_k)
plt.imshow(new_image,cmap='gray')
```
![](/image/horizon.png)

```python
# example 2 - lenna test
filter_set = [horizon_k, vertical_k, tilt_k, laplacian_k]
filter_name = ['horizon_edge', 'vertical_edge', 'tilt_edge', 'laplacian_edge']
image = Image.open('./Lenna.png').convert('L')

fig = plt.figure(figsize=(12,12))
columns = 4
rows = 1

for idx,filter in enumerate(filter_set):
    print(str(filter_name[idx]))
    new_image = convolution(image, filter)
    fig.add_subplot(rows,columns,idx+1)
    plt.imshow(new_image,cmap='gray')
```
![](/image/edge_detection.png)