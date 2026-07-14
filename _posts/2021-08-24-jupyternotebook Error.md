---
title:  "Jupyter notebook Error The port 8888 is already in use, trying another port."
excerpt: "Jupyter notebook Error The port 8888 is already in use, trying another port."

categories:
  - Error resolution
tags:
  - Error resolution
  - ubuntu
  - Jupyter notebook Error
  
last_modified_at: 2021-08-24T08:06:00-05:00
mathjax: true
---

이 오류가 뜨는 이유는 서버에서 port 8888로 주피터를 사용하고 있었는데

ctrl+z로 프로그램을 일시정지 Stopped를 시킨것이다. 

그러므로 stopped된 포트를 넘어서

![image](https://user-images.githubusercontent.com/60643542/130587961-c04fdaec-68ee-4c30-ba9c-a3a9b0693562.png)

 8891까지 동작을 하는것이다.

 

이를 해결하기 위해서는 stopped된 프로그램을 다시 동작시킨후 ctrl+c로 정상적으로 커널을 종료하는것이다. 

 

### 일시정지된 프로세스를 보는 방법

```bash
jobs
```
### 일시정지된 프로세스를 foreground 로 돌리는 방법
```bash
fg
```

![image](https://user-images.githubusercontent.com/60643542/130588158-9b8a8a78-8b42-462c-a0ab-7772398d976f.png)

jobs을 사용해 이렇게 정지된 목록을 확인하고

![image](https://user-images.githubusercontent.com/60643542/130588307-94ae2751-a3e9-491e-8c08-4a9764178fce.png)

fg로 foreground로 돌려서 ctrl+c로 정상적으로 프로그램을 종료하는것이다. 

 

다 날리면 정상 작동한다.