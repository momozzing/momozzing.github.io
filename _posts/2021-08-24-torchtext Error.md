---
title:  "AttributeError: module 'torchtext.data' has no attribute 'Field'"
excerpt: "AttributeError: module 'torchtext.data' has no attribute 'Field' 해결"

categories:
  - Error resolution
tags:
  - Error resolution
  - pytorch
  - torchtext
  - ubuntu
  - AttributeError
last_modified_at: 2021-08-24T08:06:00-05:00
mathjax: true
---

터미널에서 pip install torchtext 설치하면 가장 최신버전인 0.9.0이 설치가 된다.

 

버전 업그레이드 이후 torchtext.data 안에 있는것들이 폴더가 살짝 바뀌었다. 

![image](https://user-images.githubusercontent.com/60643542/130588663-da669c18-e0a5-4cc6-b443-a674491d0001.png)

torchtext.legacy 라는 폴더가 생기면서 기존의 0.8.1이하의 버전과 다르게 legacy를 import해줘야한다.


[참고](https://github.com/pytorch/text/tree/master/torchtext/legacy)