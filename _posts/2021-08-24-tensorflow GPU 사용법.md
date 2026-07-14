---
title:  "How to use tensorflow GPU "
excerpt: "텐서플로우 GPU 사용법"

categories:
  - Dev Setup
tags:
  - Dev Setup
  - pytorch
  - torchtext
  - window
  - 윈도우
  - tensorflow
  - GPU
last_modified_at: 2021-08-24T08:06:00-05:00
mathjax: true
---
[참고](https://teddylee777.github.io/colab/tensorflow-gpu-install-windows)


일단 해당 가상환경 CMD를 킨후에 pip install tensorflow 를 한다.

그후에 텐서플로우의 버전을 확인후 텐서플로우 공식홈페이지 가서 버전을 확인후

호환가능한 CUDA버전과 CUDNN버전을 다운받고 합친다. 

![image](https://user-images.githubusercontent.com/60643542/130590067-ceef7114-cf07-434e-b41d-d173916c66a5.png)

필자는 텐서플로우 버전이 2.3.1이며 버전 확인은 아래와 같이 하면 된다. 

![image](https://user-images.githubusercontent.com/60643542/130590124-ae46949e-b902-41e2-bab1-bc2fe85b8483.png)

버전 확인후 2.3.1과 맞는 

![image](https://user-images.githubusercontent.com/60643542/130590173-7625927a-4b3d-432d-baad-7068b75ed443.png)

CUDA 10.1을 다운받았으며 CUDNN도 7.4 버전을 다운받았다.

다운받은후 C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.1 폴더안에 CUDNN을 합쳤으며

![image](https://user-images.githubusercontent.com/60643542/130590217-7c242fe9-5893-4604-b4be-dcc719d42dbe.png)

제어판에서 GPU 정상 동작을 확인하였다.