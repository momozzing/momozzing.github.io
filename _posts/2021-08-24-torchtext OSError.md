---
title:  "torchtext OSError 해결"
excerpt: "torchtext OSError"

categories:
  - Error resolution
tags:
  - Error resolution
  - pytorch
  - torchtext
  - ubuntu
  - OSError
last_modified_at: 2021-08-24T08:06:00-05:00
mathjax: true
---

![image](https://user-images.githubusercontent.com/60643542/130584545-afcd71da-fbe3-4f96-88fa-1a406bff0b97.png)
이런 문제가 발생 하였다. 

```
OSError: /home/nlplab/anaconda3/envs/momo/lib/python3.7/site-packages/torchtext/_torchtext.so: undefined symbol: _ZNK3c104Type14isSubtypeOfExtESt10shared_ptrIS0_EPSo 
```

알아보니 torch와 torchtext의 버전이 맞지 않아서 였다. 

[파이토치 홈페이지](https://pytorch.org/get-started/locally/)에 가서 맞는 버전 다운받으면 된다. 



 
```
conda install pytorch==1.7.1 torchvision==0.8.2 torchaudio==0.7.2 cudatoolkit=10.1 -c pytorch
```

나한테 맞는 버전으로 다시 설치하니 돌아갔다.

 

위에꺼 코드 그냥 돌리면 전에 있던 버전 삭제후 새로 설치 해주는 것이다.

 

맘편하게 위에꺼 코드 돌리자. 아니면 [파이토치 홈페이지](https://pytorch.org/get-started/locally/) 들어가서 본인에 맞는 버전 선택후 설치하면 될것이다. 
