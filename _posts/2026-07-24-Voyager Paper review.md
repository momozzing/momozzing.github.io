---
title: "Voyager Paper review"
excerpt: "마인크래프트에서 스스로 과제를 정하고, 성공한 행동을 코드로 저장해 재사용하는 lifelong learning 에이전트. ReAct와 Reflexion은 여기서 나무 도구도 못 만들었다."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Paper review
toc: true
toc_sticky: true
field: agent
---

Voyager: An Open-Ended Embodied Agent with Large Language Models

[https://arxiv.org/abs/2305.16291](https://arxiv.org/abs/2305.16291) / [데모 사이트](https://voyager.minedojo.org/)

Voyager는 NVIDIA + Caltech 등에서 만든 마인크래프트 에이전트다. (2023)

[Generative Agents](https://momozzing.github.io/paper%20review/Generative-Agents-Paper-review/)는 에이전트의 경험을 자연어 기억으로 쌓았는데, Voyager는 성공한 행동을 실행 가능한 코드로 쌓는다.

목표는 정해진 과제를 푸는 게 아니라, 끝이 없는 세계에서 계속 탐험하면서 강해지는 lifelong learning(평생 학습)이다.

베이스라인으로 앞에서 리뷰한 ReAct랑 Reflexion이 나온다. 결과부터 말하면 둘 다 나무 도구도 못 만들었다.

좀 더 자세히 알아보자.

![탐험 성능 비교 (논문 Figure 1)](https://momozzing.github.io/assets/images/voyager/fig1-exploration.png)

주황색이 Voyager다.

Voyager가 160번 반복하는 동안 고유 아이템 63종을 찾고 다이아몬드 도구까지 가는 동안, ReAct랑 Reflexion은 바닥에 붙은 수평선이다.

## **1. Introduction**

마인크래프트는 정해진 엔딩이 없다. 나무를 캐고, 도구를 만들고, 더 좋은 도구로 더 깊이 내려가는 테크 트리를 스스로 올라가야 한다.

논문은 이런 세계에서 사람 개입 없이 계속 배우는 에이전트의 조건을 세 가지로 든다.

1. 자기 실력과 세계 상태에 맞는 다음 과제를 스스로 정할 것 (사막에 있으면 철보다 모래, 선인장부터)
2. 환경 피드백으로 스킬을 다듬고, 완성된 스킬을 저장해서 다시 쓸 것
3. 새 과제를 찾아서 계속 탐험할 것

## **2. Method**

위 세 가지를 세 구성 요소로 만든다.

GPT-4를 블랙박스 API로만 부르고, 파인튜닝이나 gradient 업데이트는 없다.

![Voyager 구성 요소 (논문 Figure 2)](https://momozzing.github.io/assets/images/voyager/fig2-overview.png)

### **2.1 Automatic Curriculum**

뭘 배울지부터 모델이 정한다.

"최대한 다양한 것을 발견하라"는 목표 아래에서, GPT-4가 에이전트의 현재 상태(인벤토리, 장비, 주변 블록, 바이옴, 체력)와 지금까지 성공/실패한 과제 목록을 보고 다음 과제를 낸다.

프롬프트에 "너무 어려운 과제는 내지 마라. 아직 자원과 스킬이 부족할 수 있다"는 지시가 들어 있어서, 실력에 맞춰 난이도가 올라가는 커리큘럼이 된다고 한다.

![자동 커리큘럼이 제안하는 과제들 (논문 Figure 3)](https://momozzing.github.io/assets/images/voyager/fig3-curriculum.png)

상황에 따라 제안이 달라진다.

- 나무 곡괭이와 돌이 있으면 돌 곡괭이로 업그레이드
- 강 바이옴에서 낚싯대가 있으면 낚시
- 배고픔이 0인데 근처에 돼지가 있으면 사냥
- 밤에 돌검과 방패를 들고 있으면 좀비 사냥

목표는 같아도("다양한 것을 발견하라") 상태가 다르면 다른 과제가 나온다.

논문은 이걸 in-context 형태의 novelty search(새로움 탐색)라고 부른다.

-> 탐험 자체를 보상으로 쓰는 강화학습의 오래된 아이디어를, 학습 없이 프롬프트로 만든 거다.

### **2.2 Skill Library**

과제를 푼 코드는 버리지 않고 스킬로 저장한다.

스킬 하나는 Mineflayer API를 부르는 JavaScript 함수다. 예를 들어 `combatZombie(bot)`은 무기를 챙기고, 없으면 만들고, 좀비랑 싸우는 절차 전체를 담는다.

![스킬 저장과 검색 (논문 Figure 4)](https://momozzing.github.io/assets/images/voyager/fig4-skill-library.png)

저장할 때는 GPT-3.5가 코드 설명을 만들고, 그 설명의 임베딩을 key로, 코드를 value로 벡터 DB에 넣는다.

새 과제가 오면 해결 계획과 환경 피드백을 쿼리로 관련 스킬 top-5를 찾아서 코드 생성 프롬프트에 넣어준다.

이렇게 하면 두 가지가 좋다고 한다.

1. 복잡한 스킬이 단순한 스킬을 불러서 조합되니까 능력이 쌓인다
2. 완성된 코드는 안 바뀌니까 catastrophic forgetting(파괴적 망각)이 없다

### **2.3 Iterative Prompting Mechanism**

코드가 한 번에 완성되지는 않는다. 세 종류의 피드백으로 고쳐 나간다.

![환경 피드백과 실행 에러 (논문 Figure 5)](https://momozzing.github.io/assets/images/voyager/fig5-feedback.png)

1. 환경 피드백 : "막대기를 못 만든다. 판자 2개가 더 필요하다" 같은 게임 안 중간 결과
2. 실행 에러 : 인터프리터가 내는 에러. 없는 아이템(acacia axe)을 만들려던 코드가 에러를 보고 wooden axe로 고쳐진다
3. Self-verification(자기 검증) : 별도의 GPT-4 에이전트가 현재 상태와 과제를 보고 성공했는지 판단하고, 실패면 비평을 남긴다

생성 → 실행 → 피드백 반영을 반복하다가 자기 검증이 성공을 확인하면 스킬 라이브러리에 넣는다. 계속 실패하면 커리큘럼이 다른 과제를 낸다.

논문은 자기 검증이 Reflexion의 self-reflection보다 포괄적이라고 한다. 실패하고 반성만 하는 게 아니라 성공 판정까지 하기 때문이다.

## **3. Experiments**

베이스라인은 ReAct, Reflexion, AutoGPT다. 전부 같은 GPT-4와 같은 Mineflayer API를 쓴다.

### **3.3 Evaluation Results**

탐험 결과는 앞의 Figure 1 그대로다. 고유 아이템 63종으로 베이스라인의 3.3배다.

"다양한 것을 발견하라" 같은 열린 목표는 추론 루프만으로는 실행 계획으로 안 바뀐다는 게 논문의 분석이다.

![테크 트리 결과 (논문 Table 1)](https://momozzing.github.io/assets/images/voyager/table1-techtree.png)

테크 트리에서는 나무 도구를 AutoGPT보다 15.3배 빨리 뚫는다(6회 vs 92회 반복). 돌은 8.5배, 철은 6.4배다.

다이아몬드 도구는 Voyager만 도달했다. 다만 3번 중 1번 성공이다. 이 수치는 표에 그대로 나와 있다.

지도 탐사에서는 여러 지형을 넘나들며 베이스라인의 2.3배 거리를 이동했다. 베이스라인은 좁은 지역에 갇히는 경향이 있다고 한다.

![이동 범위 조감도 (논문 Figure 7)](https://momozzing.github.io/assets/images/voyager/fig7-map.png)

주황색 원이 Voyager의 이동 범위다.

-> 커리큘럼이 새 자원을 요구하니 새 지형으로 가야 하고, 이동 스킬이 쌓이니 멀리 갈 수 있게 되는 식으로 도는 것 같다.

새 월드 일반화 실험은 인벤토리를 비우고 처음 보는 월드에서 다이아몬드 곡괭이, 황금 검, 용암 양동이, 나침반을 만들게 한다.

![새로운 월드에서의 zero-shot 일반화 (논문 Table 2, Figure 8)](https://momozzing.github.io/assets/images/voyager/fig8-zeroshot.png)

Voyager는 4개 전부 3/3 성공이고, 베이스라인은 전부 0/3이다.

그리고 Voyager의 스킬 라이브러리를 AutoGPT에 붙이면 AutoGPT도 일부 과제를 풀기 시작한다.

-> 쌓아둔 스킬이 특정 에이전트 구조에 묶이지 않고 다른 데로 옮겨 쓸 수 있다는 얘기다.

### **3.4 Ablation Studies**

설계 선택 6가지(자동 커리큘럼, 스킬 라이브러리, 환경 피드백, 실행 에러, self-verification, GPT-4)를 하나씩 빼거나 바꿔서 탐험 성능이 어떻게 되는지 본다.

![Ablation 결과 (논문 Figure 9)](https://momozzing.github.io/assets/images/voyager/fig9-ablation.png)

1. 커리큘럼을 랜덤 순서로 바꾸면 발견 아이템이 93% 줄어든다. 순서가 안 맞으면 너무 어려운 과제를 먼저 만나서 그렇다고 한다. 사람이 직접 짠 커리큘럼도 자동 커리큘럼보다 못한데, 에이전트의 지금 상황을 반영하지 못해서다.
2. 스킬 라이브러리를 빼면 후반에 성장이 멈춘다. 왼쪽 그래프의 파란 선이 중반 이후 평평해진다. 새 스킬이 이전 스킬 위에 쌓이는 구조가 끊겨서다.
3. 피드백 3종 중에서는 self-verification이 제일 중요하다고 한다. 빼면 발견 아이템이 73% 줄어든다. 성공인지 판단해야 다음 과제로 갈지 다시 할지를 정할 수 있다.
4. 코드 생성을 GPT-4에서 GPT-3.5로 바꾸면 고유 아이템이 5.7배 차이 난다.

-> 3번은 Reflexion 리뷰의 Table 3("근거 없는 반성은 해롭다")이랑 같은 방향이다. 루프에서 판정하는 부분이 빠지면 나머지가 있어도 무너진다.

-> 4번을 보면 구조가 좋아도 결국 코드 품질이 받쳐줘야 하는 것 같다.

### **3.5 Multimodal Feedback from Humans**

사람이 self-verification(비평)과 커리큘럼(단계 나누기) 역할을 대신하는 실험도 있다.

사람 피드백을 받으면 네더 포탈이나 집 같은 3D 건축까지 한다고 한다.

-> 모듈을 사람으로 바꿔 끼울 수 있을 만큼 역할이 잘 나뉘어 있다는 뜻 같다.

![사람 피드백으로 3D 건축 (논문 Figure 10)](https://momozzing.github.io/assets/images/voyager/fig10-human-feedback.png)

왼쪽에서 오른쪽으로 피드백을 반영하면서 건물이 완성된다.

Voyager는 화면을 못 봐서, 공간 구조가 틀린 건 사람이 눈으로 보고 비평해줘야 한다. 아래 Limitations의 시각 없음이 이런 얘기다.

## **4. Limitations**

1. 비용 : GPT-4 API가 GPT-3.5의 15배다. 그런데 ablation에서 본 것처럼 GPT-4 없이는 안 된다
2. 환각 : 커리큘럼이 게임에 없는 아이템(구리 검)을 과제로 내거나, 코드가 연료로 못 쓰는 조약돌을 연료로 넣거나 없는 함수를 부른다
3. 자기 검증 실패 : 거미를 잡았다는 신호인 거미줄을 성공으로 알아보지 못하는 경우가 있다
4. 시각 없음 : 당시 GPT-4 API가 텍스트만 받아서, 화면이 아니라 봇 API의 텍스트 상태만 읽는다

## **5. 지금 관점: 검증된 절차를 코드로 저장한다**

이 시리즈에서 본 에이전트들은 기억 형태가 달랐다.

- Reflexion : 실패의 교훈을 자연어 반성문으로 저장
- Generative Agents : 경험을 자연어 관찰과 반성으로 저장
- Voyager : 실행 가능한 코드로 저장

자연어 기억은 다음 판단의 참고 자료인데, 코드 스킬은 그대로 다시 실행되고 다른 스킬의 부품이 된다.

검증을 통과한 절차를 스킬로 저장하고, 필요할 때 찾아서 조합하는 설계는 지금 코딩 에이전트들이 도구랑 스킬을 쌓는 방식으로 이어졌다.

-> ReAct랑 Reflexion이 전멸한 것도 볼 만하다. 추론 루프와 반성은 과제가 정해져 있을 때 쓰는 도구 같다. 뭘 할지부터 정해야 하는 열린 세계에서는 커리큘럼(방향)과 스킬 축적이 없으면 같은 자리를 맴돈다.

## **6. Conclusion**

스스로 과제를 정하고(automatic curriculum), 성공한 코드를 저장해서 다시 쓰고(skill library), 세 종류의 피드백으로 코드를 다듬는(iterative prompting) 세 요소로, 파인튜닝 없이 마인크래프트 테크 트리를 다이아몬드까지 올라가는 에이전트를 만들었다.

쌓은 스킬은 새 월드에서도, 다른 에이전트에 옮겨도 동작한다고 한다.

대신 비용이 크고, 환각이 남아 있고, 다이아몬드는 3번 중 1번이다.

여태까지 에이전트가 강해지려면 모델 가중치를 바꿔야 했다면, 이 방법은 검증된 코드를 쌓는 걸로 강해진다.
