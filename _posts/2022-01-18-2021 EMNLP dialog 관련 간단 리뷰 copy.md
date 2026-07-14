---
title:  "2021 EMNLP dialog 관련 간단 리뷰"
excerpt: "2021 EMNLP 간단 리뷰"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - TOD
  - Dialog system
  - Task-Oriented Dialog System
  - Deep learning

last_modified_at: 2022-01-18T08:06:00-05:00
mathjax: true
# classes: wide
toc: true
toc_sticky: true
---

EMNLP

**Contextualize Knowledge Bases with Transformer for End-to-end Task-Oriented Dialogue Systems** 

→ Comet → encoder2 decoder1 model 

**Efficient Dialogue Complementary Policy Learning via Deep Q-network Policy and Episodic Memory Policy**

 → DQN 기반 

**Learning Neural Templates for Recommender Dialogue System**

 → 추천시스템 + 대화시스템

**Neural Path Hunter: Reducing Hallucination in Dialogue Systems via Path Grounding** 

→ 그래프기반 dialog system

**Thinking Clearly, Talking Fast: Concept-Guided Non-Autoregressive Generation for Open-Domain Dialogue Systems** 

→ 그래프기반

**CoLV: A Collaborative Latent Variable Model for Knowledge-Grounded Dialogue Generation** 

→ 휴리스틱기반 knowledge select

**A Three-Stage Learning Framework for Low-Resource Knowledge-Grounded Dialogue Generation**

→ transformer 형태 변경해서 TOD 

**Intention Reasoning Network for Multi-Domain End-to-end Task-Oriented Dialogue** 

→ IR NET ? 

**Domain-Lifelong Learning for Dialogue State Tracking via Knowledge Preservation Networks**         

→ DST 성능 올린 모델

**Exophoric Pronoun Resolution in Dialogues with Topic Regularization**  

→ 대화안에서 대명사 해결

**Different Strokes for Different Folks: Investigating Appropriate Further Pre-training Approaches for Diverse Dialogue Tasks**

→ pre-training model에서 finetuning 방법론 여러개 제시후 tod에 적용후 풀어본 논문 

**Graph Based Network with Contextualized Representations of Turns in Dialogue**

→ 그래프를 TOD에 적용 

**Knowledge Enhanced Fine-Tuning for Better Handling Unseen Entities in Dialogue Generation**    

→ pre-training model에서 finetuning 방법론 제시후 open domain chat에 적용

**Automatically Exposing Problems with Neural Dialog Models**

→ 답변 출력에 MLP를달아 opendomain chatbot 성능 올린 논문 

**Adaptive Bridge between Training and Inference for Dialogue Generation**

→ 생성된 대화와 정답 label과의 임베딩후 confusion matrix후 높은걸로 답변 생성

**Knowledge-Aware Graph-Enhanced GPT-2 for Dialogue State Tracking** 

→ GPT-2 base model에 그래프 단거

**End-to-End Learning of Flowchart Grounded Task-Oriented Dialogs**

→ Flow Chart 형식으로 지식기반 검색해서 TOD 푸는것.

**CR-Walker: Tree-Structured Graph Reasoning and Dialog Acts for Conversational Recommendation**

→ 그래프 기반으로 추천시스템 도입해 대화성능 향상 

**Cross-lingual Intermediate Fine-tuning improves Dialogue State Tracking**

→ 두 언어를 다루는 대화에서 파인튜닝단계 변경해서 적용

**Just Say No: Analyzing the Stance of Neural Dialogue Generation in Offensive Contexts**

→ 데이터 생성해서 공격적인 답변 생성 제거하는것 

**DIALKI: Knowledge Identification in Conversational Systems through Dialogue-Document Contextualization**

→ Dialog와 document 동시에 넣어서 knowledge 찾아서 대화시스템에 쓰는것 

**Self-training Improves Pre-training for Few-shot Learning in Task-oriented Dialog Systems**

→ self training 을 TOD에 적용시킴 

**Continual Learning in Task-Oriented Dialogue Systems**

→ CL을 TOD에 적용   [https://engineering-ladder.tistory.com/94](https://engineering-ladder.tistory.com/94)  → CL 설명 

**Zero-Shot Dialogue State Tracking via Cross-Task Transfer**

→ T5를 사용해 DST를 한 논문 

**MRF-Chat: Improving Dialogue with Markov Random Fields**

→ graph based chatbot model인데 MRF를 단 모델 

**Dialogue State Tracking with a Language Model using Schema-Driven Prompting**

→ T5를 사용해 DST를 한논문. 스키마 설명으로 프롬프트를 보강

**Transferable Persona-Grounded Dialogues via Grounded Minimal Edits**

→ 대화 안에서 Persona를 가지게 Edits로 수정

**Uncertainty Measures in Neural Belief Tracking and the Effects on Dialogue Policy Performance**

→ dialog state tracking의 점수가 Dialog policy의 점수에 미치는 영향

**Generation and Extraction Combined Dialogue State Tracking with Hierarchical Ontology Integration**

→ DST를 계층적으로 쌓아서 품

**Is Information Density Uniform in Task-Oriented Dialogues?**

→ GPT-2 based model들 까는거 → GPT-2 는 정보량이 부족하다. 그래서 성능이 안나온다. 

**Contextual Rephrase Detection for Reducing Friction in Dialogue Systems**

→ 대화 시스템에서 마찰을 줄이기 위한 상황별 표현 감지

**Improving End-to-End Task-Oriented Dialogue System with A Simple Auxiliary Task** 

→ UBAR 후속논문 → Span-based로 대화잘라주니 성능이 올랐다. 

**Task-Oriented Clustering for Dialogues**

→ Graph based feature extract 사용 

**Probing Commonsense Explanation in Dialogue Response Generation**

→ open domain chatbot 에서 Response Generation 단계 건드려서 성능 높인 paper 

**Constructing Emotion Consensus and Utilizing Unpaired Data for Empathetic Dialogue Generation**

→ 두 문장에 한 단어가 겹쳤는데 그게 감정, 공감인 데이터일때 활용 방법 

**Give the Truth: Incorporate Semantic Slot into Abstractive Dialogue Summarization**

→ BART모델용, encoder단에서 Token representation과 Utterance representation을 뽑아서 
 이를 토큰레벨 크로스어텐션, 문장레벨 크로스 어텐션을 진행. 후  대화 요약 

**Improving Dialogue State Tracking with Turn-based Loss Function and Sequential Data Augmentation**

→ Turn-based Loss → belief state loss만들고 belief state 의 data를 augmentation해서 성능올림

**Combining Curriculum Learning and Knowledge Distillation for Dialogue Generation**

→ Curriculum Learning과 Knowledge Distillation를 사용해 Response Generation의 성능을 높임

**Simulated Chats for Building Dialog Systems: Learning to Generate Conversations from Instructions**

→ Simulator를 사용해 Woz의 대화를 지침(안내서)로 만들어 input에 넣어서 좀더 좋은 성능 도출 

**A Model of Cross-Lingual Knowledge-Grounded Response Generation for Open-Domain Dialogue Systems** 

→ KE-T5 : Korean-English T5 model paper  → KoWOW데이터가 있다 → 아니다 WOW를 번역해서 쓴것,

**Effective Sequence-to-Sequence Dialogue State Tracking**

→ Decoder만 쓰는 Auto Regression model은 성능이 잘 안나온다 MLM도 쓰는 Seq2seq모델을 사용하면 성능이 오른다.