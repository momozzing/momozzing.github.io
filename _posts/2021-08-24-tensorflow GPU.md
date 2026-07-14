---
title:  "tensorflow GPU 동작확인"
excerpt: "tensorflow GPU 동작확인"

categories:
  - Dev Setup
tags:
  - Dev Setup
  - pytorch
  - torchtext
  - 윈도우
  - window
  - tensorflow
  - GPU
last_modified_at: 2021-08-24T08:06:00-05:00
mathjax: true
---
```
from tensorflow.python.client import device_lib

device_lib.list_local_devices()
```
GPU 인식

![image](https://user-images.githubusercontent.com/60643542/130589519-5fcb1ff1-ad05-436a-9ac7-c03328c90902.png)


nvcc --version   cuda 실제버전확인

![image](https://user-images.githubusercontent.com/60643542/130589601-e16dd2be-cca4-4dd0-8f46-f447722db52a.png)

nvidia-smi  GPU메모리 확인 및 smi에 나와있는 cuda version 은 최대 이용가능한 드라이버 버전임 

![image](https://user-images.githubusercontent.com/60643542/130589653-af2e59b3-7b9a-4668-abd1-50a1c21820e1.png)
