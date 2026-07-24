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
---

Voyager: An Open-Ended Embodied Agent with Large Language Models

[https://arxiv.org/abs/2305.16291](https://arxiv.org/abs/2305.16291) / [데모 사이트](https://voyager.minedojo.org/)

Voyager는 NVIDIA + Caltech 등에서 나온 논문이다. (2023)

[Generative Agents](https://momozzing.github.io/paper%20review/Generative-Agents-Paper-review/)가 에이전트의 경험을 자연어 기억으로 쌓았다면, Voyager는 **성공한 행동을 실행 가능한 코드로 쌓는다.** 환경은 마인크래프트고, 목표는 정해진 과제 풀기가 아니라 게임이 끝나지 않는 세계에서 계속 탐험하며 강해지는 lifelong learning(평생 학습)이다.

그리고 베이스라인으로 우리가 앞서 리뷰한 ReAct와 Reflexion이 등장한다. 결과를 먼저 말하면, 둘 다 나무 도구조차 만들지 못했다.

![탐험 성능 비교 (논문 Figure 1)](https://momozzing.github.io/assets/images/voyager/fig1-exploration.png)

주황색이 Voyager다. 160번의 반복 동안 고유 아이템 63종을 발견하며 다이아몬드 도구까지 도달하는 동안, ReAct와 Reflexion은 바닥의 수평선이다. 이 격차가 어디서 오는지가 이 논문의 내용이다.

## **Introduction**

마인크래프트는 정해진 엔딩이 없다. 나무를 캐고, 도구를 만들고, 더 좋은 도구로 더 깊이 내려가는 테크 트리를 스스로 올라가야 한다. 논문은 이런 세계에서 사람 개입 없이 계속 배우는 에이전트의 조건을 세 가지로 정리한다.

- 자신의 실력과 세계 상태에 맞는 **다음 과제를 스스로 제안**할 것 (사막에 있으면 철보다 모래·선인장부터)
- 환경 피드백으로 스킬을 다듬고, **완성된 스킬을 저장해 재사용**할 것
- 새로운 과제를 찾아 **탐험을 계속**할 것

Voyager는 이를 세 가지 구성 요소로 구현한다. GPT-4를 블랙박스 API로만 호출하며, 파인튜닝이나 gradient 업데이트는 없다.

![Voyager 구성 요소 (논문 Figure 2)](https://momozzing.github.io/assets/images/voyager/fig2-overview.png)

## **Automatic Curriculum**

무엇을 배울지부터 모델이 정한다. "최대한 다양한 것을 발견하라"는 상위 목표 아래, GPT-4가 에이전트의 현재 상태(인벤토리, 장비, 주변 블록, 바이옴, 체력)와 지금까지의 성공/실패 과제 목록을 보고 다음 과제를 제안한다. "너무 어려운 과제는 내지 마라. 아직 자원과 스킬이 부족할 수 있다"는 지시가 프롬프트에 들어 있어서, 난이도가 실력을 따라 올라가는 커리큘럼이 된다.

![자동 커리큘럼이 제안하는 과제들 (논문 Figure 3)](https://momozzing.github.io/assets/images/voyager/fig3-curriculum.png)

제안이 상황을 따라간다는 것이 보인다. 나무 곡괭이와 돌이 있으면 돌 곡괭이 업그레이드를, 강 바이옴에서 낚싯대가 있으면 낚시를, 배고픔이 0인데 근처에 돼지가 있으면 사냥을, 밤에 돌검과 방패를 들고 있으면 좀비 사냥을 한다. 같은 목표("다양한 것을 발견하라")라도 상태가 다르면 다른 과제가 나온다.

논문은 이를 in-context 형태의 novelty search(새로움 탐색)라고 부른다. 탐험 자체를 보상으로 삼는 강화학습의 오래된 아이디어를, 학습 없이 프롬프트로 구현한 것이다.

## **Skill Library**

과제를 풀어낸 코드는 버리지 않고 스킬로 저장한다. 스킬 하나는 Mineflayer API를 호출하는 JavaScript 함수다. `combatZombie(bot)` 같은 함수가 무기를 챙기고, 없으면 만들고, 좀비와 싸우는 절차 전체를 담는다.

![스킬 저장과 검색 (논문 Figure 4)](https://momozzing.github.io/assets/images/voyager/fig4-skill-library.png)

저장할 때는 GPT-3.5가 코드 설명을 생성하고, 그 설명의 임베딩을 key로, 코드를 value로 벡터 DB에 넣는다. 새 과제를 받으면 해결 계획과 환경 피드백을 쿼리로 관련 스킬 top-5를 검색해서 코드 생성 프롬프트에 넣어준다.

이 설계의 효과는 두 가지다. 복잡한 스킬이 단순한 스킬을 호출하며 **조합**되므로 능력이 누적되고, 완성된 코드는 변하지 않으므로 **catastrophic forgetting(파괴적 망각)이 없다.**

## **Iterative Prompting Mechanism**

코드가 한 번에 완성되지는 않는다. 세 종류의 피드백으로 고쳐 나간다.

![환경 피드백과 실행 에러 (논문 Figure 5)](https://momozzing.github.io/assets/images/voyager/fig5-feedback.png)

1. **환경 피드백**: "막대기를 못 만든다. 판자 2개가 더 필요하다" 같은 게임 내 중간 결과
2. **실행 에러**: 인터프리터가 뱉는 에러. 존재하지 않는 아이템(acacia axe)을 만들려던 코드가 에러를 보고 wooden axe로 고쳐진다
3. **Self-verification(자기 검증)**: 별도의 GPT-4 에이전트가 현재 상태와 과제를 보고 성공 여부를 판정하고, 실패면 비평을 남긴다

생성 → 실행 → 피드백 반영을 반복하다가 자기 검증이 성공을 확인하면 스킬 라이브러리에 등록하고, 계속 실패하면 커리큘럼이 다른 과제를 낸다. 논문은 자기 검증이 Reflexion의 self-reflection보다 포괄적이라고 명시한다. 실패 후 반성만 하는 것이 아니라 성공 판정까지 하기 때문이다.

## **Experiments**

베이스라인은 ReAct, Reflexion, AutoGPT다. 전부 같은 GPT-4와 같은 Mineflayer API를 쓴다.

**탐험**: 서두의 Figure 1 그대로다. 고유 아이템 63종으로 베이스라인의 3.3배다. 열린 목표("다양한 것을 발견하라")는 추론 루프만으로는 실행 계획으로 옮겨지지 않는다는 것이 논문의 분석이다.

![테크 트리 결과 (논문 Table 1)](https://momozzing.github.io/assets/images/voyager/table1-techtree.png)

**테크 트리**: 나무 도구를 AutoGPT보다 15.3배 빨리 뚫고(6회 vs 92회 반복), 돌 8.5배, 철 6.4배다. 다이아몬드 도구는 Voyager만 도달했다. 다만 3번 중 1번 성공이다. 이 수치는 표에 그대로 있고 논문도 숨기지 않는다.

**지도 탐사**: 다양한 지형을 넘나들며 베이스라인의 2.3배 거리를 이동했다. 베이스라인은 좁은 지역에 갇히는 경향을 보인다.

![이동 범위 조감도 (논문 Figure 7)](https://momozzing.github.io/assets/images/voyager/fig7-map.png)

주황색 원이 Voyager의 이동 범위다. 커리큘럼이 새 자원을 요구하니 새 지형으로 갈 수밖에 없고, 이동 스킬이 쌓이니 멀리 갈 수 있게 되는 순환이다.

![새로운 월드에서의 zero-shot 일반화 (논문 Table 2, Figure 8)](https://momozzing.github.io/assets/images/voyager/fig8-zeroshot.png)

**새 월드 일반화**: 인벤토리를 비우고 처음 보는 월드에서 다이아몬드 곡괭이, 황금 검, 용암 양동이, 나침반 제작을 시킨다. Voyager는 4개 전부 3/3 성공, 베이스라인은 전부 0/3이다. 흥미로운 것은 Voyager의 스킬 라이브러리를 AutoGPT에 이식하면 AutoGPT도 일부 과제를 풀기 시작한다는 점이다. 축적된 스킬이 특정 에이전트 구조에 종속되지 않는 이식 가능한 자산이라는 뜻이다.

## **Ablation Studies**

설계 선택 6가지(자동 커리큘럼, 스킬 라이브러리, 환경 피드백, 실행 에러, self-verification, GPT-4)를 각각 빼거나 바꿔가며 탐험 성능에 미치는 영향을 잰다.

![Ablation 결과 (논문 Figure 9)](https://momozzing.github.io/assets/images/voyager/fig9-ablation.png)

- **커리큘럼을 랜덤 순서로 바꾸면 발견 아이템이 93% 줄어든다.** 순서가 안 맞으면 너무 어려운 과제를 먼저 만나기 때문이다. 사람이 손으로 설계한 커리큘럼도 자동 커리큘럼에 못 미치는데, 에이전트의 실시간 상황을 반영하지 못해서다
- **스킬 라이브러리를 빼면 후반부에 성장이 정체된다.** 왼쪽 그래프의 파란 선이 중반 이후 평평해진다. 새 스킬이 이전 스킬 위에 쌓이는 복리 구조가 끊기기 때문이다
- **피드백 3종 중에서는 self-verification이 가장 중요하다.** 빼면 발견 아이템이 73% 줄어든다. 성공을 판정해야 "다음 과제로 넘어갈지, 재도전할지"를 결정할 수 있는데, 이 판단이 없으면 루프 전체가 방향을 잃는다
- **코드 생성을 GPT-4에서 GPT-3.5로 바꾸면 고유 아이템이 5.7배 차이 난다.** 구조가 아무리 좋아도 코드 품질이 토대라는 결과다

Reflexion 리뷰의 Table 3("근거 없는 반성은 해롭다")과 같은 방향의 결과다. 에이전트 루프에서 판정 장치가 빠지면 나머지 구조가 있어도 성능이 무너진다.

덧붙여 논문 3.5절에서는 사람이 self-verification(비평)과 커리큘럼(단계 분해)의 역할을 대신하는 실험도 보인다. 사람 피드백을 받으면 네더 포탈이나 집 같은 3D 건축까지 가능하다. 모듈 구조가 사람으로도 대체 가능할 만큼 역할 분리가 명확하다는 뜻이다.

![사람 피드백으로 3D 건축 (논문 Figure 10)](https://momozzing.github.io/assets/images/voyager/fig10-human-feedback.png)

왼쪽에서 오른쪽으로 피드백을 반영하며 건물이 완성되어 간다. Voyager가 화면을 못 보기 때문에 공간 구조의 오류는 사람이 눈으로 보고 비평해줘야 하는데, 이것이 아래 Limitations에 나오는 시각 없음이 실제로 의미하는 바다.

## **Limitations**

- **비용**: GPT-4 API가 GPT-3.5의 15배다. 그런데 ablation이 보여주듯 GPT-4 없이는 성립하지 않는다
- **환각**: 커리큘럼이 게임에 없는 아이템(구리 검)을 과제로 내거나, 코드가 없는 재료를 쓰려고 한다
- **자기 검증 실패**: 거미를 잡았다는 신호인 거미줄을 성공으로 인식하지 못하는 사례가 있다
- **시각 없음**: 당시 GPT-4 API가 텍스트 전용이라 화면이 아니라 봇 API의 텍스트 상태만 읽는다

## **지금 관점: 검증된 절차를 코드로 저장한다**

이 시리즈에서 본 에이전트들의 기억은 형태가 달랐다. Reflexion은 실패의 교훈을 자연어 반성문으로, Generative Agents는 경험을 자연어 관찰과 반성으로 저장했다. Voyager의 기억은 **실행 가능한 코드**다. 자연어 기억은 다음 판단의 참고 자료지만, 코드 스킬은 그대로 다시 실행되고 다른 스킬의 부품이 된다.

"검증을 통과한 절차를 재사용 가능한 스킬로 저장하고, 필요할 때 검색해서 조합한다"는 설계는 지금의 코딩 에이전트들이 도구와 스킬을 축적하는 방식으로 이어졌다. 에이전트가 만든 코드가 곧 에이전트의 능력이 되는 구조, 코드 실행이 곧 행동인 에이전트의 원형이 이 논문에 있다.

ReAct와 Reflexion의 전멸도 배울 점이다. 추론 루프와 반성은 과제가 명확할 때의 도구다. 무엇을 할지부터 정해야 하는 열린 세계에서는 커리큘럼(방향)과 스킬 축적(복리)이 없으면 같은 자리를 맴돈다.

## **Conclusion**

스스로 과제를 정하고(automatic curriculum), 성공한 코드를 저장해 재사용하고(skill library), 세 종류의 피드백으로 코드를 다듬는(iterative prompting) 세 요소로, 파인튜닝 없이 마인크래프트 테크 트리를 다이아몬드까지 올라가는 에이전트를 만들었다. 축적된 스킬은 새 월드에서도, 다른 에이전트에 이식해도 동작한다.

한계도 명확하다. 비용이 크고, 환각이 남아 있고, 다이아몬드는 3번 중 1번이다. 그래도 "에이전트의 성장을 모델 가중치가 아니라 검증된 코드의 축적으로 만든다"는 설계는 이후 에이전트들이 가져다 쓰는 기본기가 됐다.
