---
title:  "A Survey of Large Language Models 논문 리뷰"
excerpt: "A Survey of Large Language Models 논문 리뷰"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - Large Language Model
  - Deep learning

last_modified_at: 2023-06-05T08:06:00-05:00
mathjax: true
# classes: wide
toc: true
toc_sticky: true
---

# **A Survey of Large Language Models 논문 리뷰**

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/045fafab-40a7-4e8e-9936-af11a4b307b9)


## Introduction

단어 시퀀스에서 다음 단어를 예측하는 분야인 언어 모델링(LM)은 4개의 발전 단계로 나뉨

1. Statistical Language Modeling (SLM): 통계적 언어모델 
    
    methods from the 1990s where a simple n-gram model predicts the next word based on recent context (Markov assumption)
    
2. Neural Language Models (NLM): 신경망 언어모델
    
    use neural networks like RNNs, LSTMs, GRUs, word2vec
    
3. Pretrained Language Models (PLM): 사전 학습 언어모델
    
    ELMo, BERT, BART, GPT-2
    
4. Large Language Models (LLM): 거대 언어모델
    
    larger PLMs like GPT-4, ChatGPT, PaLM, Sparrow, Claude, Microsoft 365's AI, etc

LLM과 PLM의 차이점

1. surprising emergent abilities

2. LLMs revolutionize the way we develop and use AI algorithms

3. development of LLM draws no clear distinction between research and engineering

LLM의 emergent ability로 인해 연구뿐만 아니라 산업 및 응용 분야에서도 관련성이 있다는 것을 알게 되었고 엔지니어와 연구원의 기술이 혼합되어야 한다고 주장.

LLM의 불확실성

1. 왜 emergent ability가 발생하는지?

2. LLM은 주로 산업 분야(대기업)에서 학습하기 때문에 연구자들이 LLM을 훈련시키는 데 어려움이 있다

3. alignment problem (LLM 답변을 인간의 원하는 답변과 정렬)

이 논문에서는 4가지 주제를 다룬다. 

1. 어떻게 Pre-train하는지

2. 효율성과 안전성을 위한 adaptation tuning

3. Downstream task에 LLM을 사용하는 방법

4. LLM의 평가방법

## Overview
이 챕터에는 LLM의 배경에 대한 설명과 함께 GPT 시리즈 모델의 기술적 발전 서술

### Background for LLM
이 섹션에는 Scaling Law, Emergent Ability 및 핵심 기술을 소개
Scaling Law는 LLM을 효율적으로 학습시키기 위해 모델 아키텍쳐의 크기, 데이터셋의 크기, 컴퓨팅 연산량에 대한 관계를 정립한것.
Scaling Law는 대표적으로 KM scaling law과 Chinchilla scaling law가 있음. 

**KM scaling law (Scaling Laws for Neural Language Models (Kaplan et al., 2020))**

- LM의 성능은 파라미터 수 N, 데이터 크기 D, 연산능력 C와 관계 있으며, 모델의 형태는 중요하지 않다.
- 즉, 컴퓨팅 파워가 세질수록, N과 D의 크기에 따라 LM 성능이 좌우된다.
- 모델의 오버피팅을 막기 위해서는 N이 10배 수준으로 커질 때, D도 5.5배 수준으로 커져야 한다.
- N이 큰 모델은 더 적은 데이터와 스텝만으로 N이 작은 모델과 동일한 수준에 도달할 수 있다.
- N이 고정되었을 때, 하이퍼 파라미터(n_layer, d_ff, n_head)는 Loss에 큰 영향은 주지 않는다.
- 따라서 Optimal Compute Budget allocation을 위해서는 컴퓨팅의 대부분을 N을 늘리는데 써야한다

**Chinchilla scaling law (Training Compute-Optimal Large Language Models (Hoffmann et al., 2022))**

한정된 자원 안에서 초거대 언어 모델의 최적의 성능을 낼 수 있는 모델의 크기와 데이터 양의 상관 관계에 대해 다양한 실험 및 분석을 수행

모델의 Floating point operations(FLOPs)이 주어졌을 때, 가장 좋은 성능을 낼 수 있는 Parameters와 Training Tokens를 찾는 방법을 제시

Compute Budget(FLOPs)를 증가시키면 모델 사이즈와 학습 데이터의 양은 비슷한 비율로 증가해야 한다. (Parameters가 x2 증가한다면 Tokens 또한 x2 증가해야 한다.)
더 큰 Compute budget에서 더 작은 모델이 더 최적이다.

**Emergent Ability**

1. In-Context Learning (ICL): 프롬프트의 내용만으로 task 를 수행하는 작업. 프롬프트 내의 맥락적 의미(in-context)를 모델이 이해하고, 이에 대한 답변을 생성하는것을 의미. 즉 in-context learning 은 pre-train 이나 fine-tuning 처럼 모델을 학습시키지 않고 inference만 진행. ex) zero-shot, few-shot

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/01fc42ef-e7b2-47bd-8c92-682c4beae076)

2. Instruction Following(instruct tuning): LLM 모델을 Instruction 형태의 데이터셋을 fine-tuning을 하고 이를 통해 zero-shot 성능을 높이는 방법
Instruction 형태의 데이터셋은 instruct와 answer의 pair로 된 data

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/3d80a983-cbd1-4e8d-a54e-e35787ce0070)

3. Step-by-Step Reasoning(chain-of-thought(CoT)): LLM 모델로 복잡한 추론 문제를 풀 때, 중간 추론 단계를 설명한 프롬프트를 사용해 성능을 높이는 방법.  

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/0387d254-bc82-4192-9c13-295911680094)

**Key Techniques for LLMs**
이 챕터에선 LLM의 크기를 성공적으로 향상시키는 여러 가지 중요한 기술을 나열.

Scaling: scaling the size of the model (GPT-3 sits at 175B and PaLM at 540B!)
Transformer 언어 모델에는 분명한 스케일링 효과가 있다. 더 큰 모델/데이터 크기와 더 많은 컴퓨팅 연산량은 모델 용량을 향상, 컴퓨팅 연산량은 일반적으로 제한되어 있기 때문에 scaling law을 사용하여 효율적으로 모델과 데이터 크기를 할당할 수 있습니다. ex) Chinchilla는 동일한 컴퓨팅 자원으로 데이터 규모를 늘림으로써 Gopher(모델 크기가 더 큼)보다 성능이 뛰어남.

Training: effectively training an LLM requires a lot (e.g. DeepSpeed and Megatron-LM for parallelization, restarting after loss spike for stable training, mixed precision training)

Alignment Tuning: Reinforcement Learning with Human Feedback (RLHF) from InstructGPT is one example of tuning the alignment of the model w.r.t human values

LLM은 사전 훈련 데이터를 학습했기 때문에 데이터의 특성에 따라 사람에게 유해하거나 편향되거나 유해한 콘텐츠를 생성할 가능성이 있다. LLM을 사람의 가치(예: 유용하고 정직하며 무해함)에 맞추는 것이 필요. 이를 위해 InstructGPT는 LLM이 예상 지침을 따를 수 있도록 사람의 피드백을 통한 강화 학습 기술을 활용

Tool Manipulation: plug-ins for ChatGPT and GPT-4 give it a much wider range of application

### Technical Evolution of GPT-series Models
이 챕터는 GPT 모델의 기술적 발전에 대해 간략히 요약  -> 생략

## LLM Resources
이 섹션에서는 모델 체크포인트(또는 API), 말뭉치 및 라이브러리를 포함하여 LLM 개발을 위해 공개적으로 사용 가능한 리소스를 간략하게 요약

**Publicly Available Model Checkpoints or APIs.**

LLM 10B 범위 기준.

For models with tens of billions of parameters, consider mT5,  T0, GPT-NeoX-20B, CodeGen, UL2, Flan-T5, mT0, PanGu-α, and LLaMA.  

For models with	hundreds of billions of parameters, consider OPT, BLOOM, and BLOOMZ. 

It's important to measure the FLOPS (floating point operations per second) to see how much compute/memory is needed for these models.

**Public API of LLMs.** 

OpenAI has released the ChatGPT API and they also provide a web-based interface for interacting with their GPT models. HuggingFace also has a model registry in which you can download model weights, use their inference endpoints, or even just query in the browser.

**Common Datasets**

We can separate the most popular datasets into these categories:

- Books
  - BookCorpus
  - Project Gutenberg
  - Books1 and Books2
- CommonCrawl
  - C4 (and its variants)
  - CC-Stories
  - CC-News
  - RealNews
- Reddit Links
  - WebText and/or OpenWebText
  - PushShift.io
- Wikipedia
- Code
  - GitHub
  - Stack Overflow
  - Google BigQuery dataset
- Others
  - The Pile
  - ROOTS

**Library Resources**

Hugging Face **Transformers** lets you easily access, train, and use hundreds of models

PyTorch **DeepSpeed** developed by Microsoft is a DL optimization library

**Megatron-LM** developed by NVIDIA provides an array of optimization techniques for training LLMs

**JAX** is a relatively new ML library developed by Google Brain

**Colossal-AI** developed by HPC-AI Tech is for training LLMs

**etc** BMTrain, FastMoE, PyTorch, TensorFlow, MXNet, PaddlePaddle, MindSpore and OneFlow







**참고**
[A Survey of Large Language Models - W&B 리뷰](https://wandb.ai/vincenttu/blog_posts/reports/A-Survey-of-Large-Language-Models--VmlldzozOTY2MDM1?galleryTag=ml-news)
