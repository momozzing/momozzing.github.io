---
title:  "Deepspeed background process kill"
excerpt: "딥스피드 백그라운드 프로세스 죽이는 법"

categories:
  - Error resolution
tags:
  - Error resolution
  - Deepspeed
  - python
  - Deep learning
  - OOM

last_modified_at: 2022-08-17T08:06:00-05:00
mathjax: true
# classes: wide
toc: true
toc_sticky: true
---

## **딥스피드 백그라운드 프로세스 죽이는 법**

Deepspeed를 사용해서 여러 대의 GPU를 병렬로 사용할 때   
 ```dist.init_process_group(backend="nccl")```를 사용하기 때문에   
 Backend에서 프로세스가 실행된다.   

이 때문에 터미널에서 Ctrl+C를 눌러 파이썬 실행파일을 종료해도 Backend에서 프로그램이 실행된다.    

그러므로 GPU의 메모리는 아직도 사용중이다. -> OOM 발생.

이를 해결하는 법은 간단하다. 

```bash
pgrep python | xargs kill
```

위 문구만 입력하면 실행되고 있는 python Backend process들을 깔끔하게 죽일 수 있다.

**주의**
* 하나의 프로그램을 여러대의 GPU를 묶어서 사용할 때 Backend python process가 종료가 안될 경우 (OOM 발생 시) 사용하자. 

* 여러 개의 프로그램을 GPU에 각각 분할해서 사용할 경우 모든 Python 실행 프로세스가 종료된다. 이를 주의하자.  