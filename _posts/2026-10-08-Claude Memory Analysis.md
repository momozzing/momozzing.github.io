---
date: 2026-10-08 01:00:00 +0900
title: "Claude 메모리 분석"
excerpt: "claude.ai 메모리를 공식 문서 기준으로 정리했다. 대화 중에 카테고리별 항목을 바로 쓰고 고치며, 과거 대화 원문은 필요할 때만 검색 tool로 꺼낸다."
categories:
  - Study
tags:
  - Large Language Model
  - Agent
  - Memory
  - Claude
toc: true
toc_sticky: true
field: agent-memory
---

[에이전트 메모리 논문 21편](https://momozzing.github.io/study/Agent-Memory-Retrospective/)을 리뷰하고 나서, 실제 서비스는 메모리를 어떻게 만들어 뒀는지 궁금해졌다.
그래서 Claude(claude.ai)와 ChatGPT의 메모리를 공식 문서 기준으로 조사했다. 조사 시점은 2026년 10월 8일이다.

글은 세 편이다. 이 글은 Claude 편이고, [ChatGPT 편](https://momozzing.github.io/study/ChatGPT-Memory-Analysis/)과 [비교 편](https://momozzing.github.io/study/Claude-ChatGPT-Memory-Comparison/)이 따로 있다.

문장마다 출처 수준을 붙였다.

- [공식] : Anthropic 공식 문서, 블로그, 릴리스 노트, 도움말
- 표시 없음 : 내 계정 화면에서 직접 확인한 것

공식 문서는 "무엇을 하는지"까지만 설명한다. 어떤 규칙으로 저장하고 어떤 형식으로 컨텍스트에 넣는지는 공개하지 않아서, 그 부분은 8절에 모르는 것으로 적었다.

## **1. 전체 구조**

대화 하나가 시작될 때 컨텍스트에 들어가는 건 이렇다.

```
claude.ai 대화 컨텍스트
├─ 시스템 프롬프트
├─ 사용자 기억 (Profile · 주제 · 영역 항목)   ← 대화 중 작성·갱신
├─ 프로젝트 메모리 (프로젝트 안일 때만)        ← 프로젝트마다 별도
├─ 대화 히스토리
└─ 필요할 때: 채팅 검색 tool → 과거 대화 원문 (RAG)
```

사용자 기억은 매번 들어가고, 과거 대화 원문은 들어가지 않는다. 원문이 필요하면 Claude가 검색 tool을 호출해서 꺼낸다.
이 둘을 나눈 게 구조의 뼈대다.

이와 별개로 개발자용 메모리가 둘 더 있다. API memory tool과 Claude Code 메모리인데, 6절에서 따로 본다.

## **2. 사용자 기억**

### **2.1 현재 구조 (2026-07-10 이후)**

- [공식] 기억은 카테고리별 개별 항목이다. Claude가 대화 중에 읽고 갱신한다. 이전에는 하루 단위 요약 하나였다.
- [공식] 대화가 끝난 뒤 요약하지 않는다. 대화하면서 주제(topic)를 메모리에 추가한다. "기억해줘"라고 하지 않아도 다음 대화에 반영된다.
- [공식] Settings > Memory에서 Topics 목록으로 전부 볼 수 있고, 주제별로 읽기·수정·삭제할 수 있다. "기억해줘"라고 직접 말해서 저장할 수도 있다.

내 계정 화면(2026-10-07)에서는 항목이 세 그룹으로 나뉘어 있었다.
업무 프로젝트 이름이 들어 있어서 화면 이미지 대신 구조만 옮긴다.

| 그룹 | 개수 | 내용 | 예시 |
|---|---|---|---|
| Profile | 1 | 사용자가 누구인지 (역할, 소속, 배경) | Profile |
| 주제 | 6 | 개인 생활, 관심사, 개발 환경 | Blog, Dev Environment, Hobbies, Recent Work |
| 영역 | 4 | 업무 프로젝트 단위 | (프로젝트 이름) |

항목마다 제목, 한 줄 설명, 업데이트 날짜가 붙는다. 한 줄 설명은 항목에 따라 영어와 한국어가 섞여 있다.
영역 하나를 열면 본문이 수천 자 분량의 문서다. 항목이라기보다 주제별 문서에 가깝다.
"Recent Work"는 독립 영역이 되기엔 작은 업무를 모아 두는 칸이다.

-> 항목마다 날짜가 붙어 있는 게 눈에 들어왔다. 뭘 언제 알았는지가 보인다.

### **2.2 이전 구조 (2025-09 ~ 2026-07)**

- [공식] 대화 기록을 요약해서 핵심을 합성했고, 이 요약을 24시간마다 갱신해 새 대화마다 넣었다.

ChatGPT가 2026-06에 백그라운드 합성(Dreaming V3) 쪽으로 간 것과 반대로, Claude는 2026-07에 하루 단위 합성 요약을 버리고 대화 중 항목 갱신으로 갔다.

## **3. 프로젝트 메모리**

- [공식] 프로젝트마다 별도 메모리 공간과 프로젝트 요약이 있다. 프로젝트 안의 맥락은 전역 메모리나 다른 프로젝트로 섞이지 않는다.

공개된 내용은 이게 전부다. 격리가 기본이고 선택지가 없다.

## **4. 채팅 검색**

과거 대화 원문을 다루는 부분이다.

- [공식] "지난달에 가격 페이지에 대해 뭐라고 정했지?" 같은 요청이 있을 때 과거 대화를 검색해서 가져온다. RAG 방식이고 대화 중 tool 호출로 보인다. 유료 플랜(Pro, Max, Team, Enterprise)에서 제공한다.
- [공식] 검색 범위는 "프로젝트 밖 전체 대화" 또는 "해당 프로젝트 대화"로 나뉜다.

과거 대화를 미리 요약해서 매번 넣지 않고, 요청이 있을 때 검색한다.

-> [ReFind](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)와 방향이 같다. 원문을 건드리지 않고 검색으로 꺼낸다. 다만 검색이 키워드인지 벡터인지, 몇 건을 어떻게 잘라서 가져오는지는 공개되지 않았다.

## **5. 민감정보와 사용자 통제**

쓰기 규칙은 둘이다.

- [공식, 2026-08-25] 건강, 신념 같은 민감 주제는 기본으로 기억하지 않는다. "Include sensitive topics in memory"를 켜면 저장한다(예: 글루텐 알레르기).
- [공식] 신분증 번호, 범죄 기록, 금융 계좌 번호는 요청해도 저장하지 않는다.

사용자가 통제할 수 있는 건 이렇다.

- [공식] Incognito 채팅: 기억에 반영되지 않고 채팅 기록에도 남지 않는다. 단, 조직 소유자는 데이터 내보내기로 접근할 수 있다.
- [공식] 대화를 지워도 그 대화에서 만들어진 기억 항목은 지워지지 않는다. 기억은 따로 지워야 한다.
- [공식] 가져오기/내보내기: 다른 AI 서비스의 기억을 가져오거나 Claude 기억을 백업·이전할 수 있다. 실험 기능이다.
- [공식] 기본값: Free, Pro, Max는 메모리가 기본으로 켜져 있다. Team, Enterprise는 관리자가 정한다.
- [공식, 2026-08-25] chat과 Cowork이 같은 메모리를 쓴다. 어디서 쓰든 같은 기억에서 시작한다.
- [공식, 2026-07-09] 월간 회고(Monthly recap): Settings > Reflect에서 그달의 주제, 활동 패턴, 업무 관찰을 보여준다. 메모리가 켜져 있어야 한다.

대화 삭제와 기억 삭제가 분리돼 있다. ChatGPT도 같은데, 사용자 입장에서는 헷갈리기 쉬운 부분이다.

## **6. 개발자용 메모리**

claude.ai 메모리와는 별개 제품이다. 같은 회사가 메모리를 어떻게 설계하는지 보여줘서 같이 정리했다.

### **6.1 Claude API memory tool**

[공식]

- 클라이언트 쪽에서 동작한다. Claude가 `/memories` 디렉터리의 파일 조작을 요청하면 애플리케이션이 자기 저장소에서 실행한다.
- 명령은 6개다. `view`, `create`, `str_replace`, `insert`, `delete`, `rename`.
- tool을 켜면 API가 "작업 전에 메모리 디렉터리부터 확인하라"는 지시를 시스템 프롬프트에 자동으로 넣는다.
- 문서 권장 사항: 경로 탈출 방지, 파일 크기 상한, 오래 안 쓴 파일 정리, 민감정보 검증.
- 서버 쪽 대화 요약(compaction)과 함께 쓰는 것을 권한다. compaction은 컨텍스트를 줄이고, memory는 요약에서 살아남아야 하는 정보를 보존한다.

메모리가 파일이고 조작이 파일 명령이다. 저장 구조를 Anthropic이 정하지 않고 모델이 파일로 알아서 짜게 둔다.

### **6.2 Claude Code**

[공식] 두 층이다.

CLAUDE.md는 사람이 쓰는 지시다.
조직(managed policy) → 사용자(`~/.claude/CLAUDE.md`) → 프로젝트 → 로컬(`CLAUDE.local.md`) 순으로 모두 이어 붙여 로드한다. 하위 디렉터리 파일은 필요할 때 로드한다.

Auto memory는 Claude가 스스로 쓰는 메모다.

- 위치: 레포마다 `~/.claude/projects/<project>/memory/`
- `MEMORY.md` 목차(한 줄에 기억 하나)와 주제 파일(기억 하나당 파일 하나)로 구성된다
- 세션 시작 때는 `MEMORY.md`의 앞 200줄 또는 25KB만 로드한다. 주제 파일은 필요할 때 읽는다
- 기억 종류는 `user`, `feedback`, `project`, `reference` 4가지다. 코드나 git 기록에서 알 수 있는 것은 저장하지 않는다

-> claude.ai의 Topics 구조와 닮았다. 목차는 매번 넣고 본문은 필요할 때 연다. claude.ai도 그렇게 하는지는 8절에.

## **7. 연혁**

[공식]

| 날짜 | 변경 |
|---|---|
| 2025-08-11 | 과거 대화 검색 (Max, Team, Enterprise) |
| 2025-09-11 | 메모리 출시 (Team 등). 대화 기록 요약 방식, Incognito 채팅 |
| 2025-09-18 | Enterprise 메모리 |
| 2025-10-23 | Max, 이어서 Pro로 확대 |
| 2026-03-02 | 무료 사용자까지 전체 제공 |
| 2026-07-09 | 월간 회고 (Settings > Reflect) |
| 2026-07-10 | 카테고리별 개별 항목, 대화 중 갱신으로 구조 변경 |
| 2026-08-25 | chat·Cowork 메모리 통합, Topics 편집 화면, 민감 주제 설정 |

출시 열 달 만에 구조를 한 번 갈아엎었다. 하루 단위 합성 요약(2025-09)에서 대화 중 항목 갱신(2026-07)으로.

## **8. 공개되지 않은 것**

- 주제·영역 항목을 매 대화에 전부 넣는지, 목차만 넣고 필요할 때 여는지는 공개되지 않았다. 영역 하나가 수천 자라서 전부 넣으면 양이 꽤 된다. Claude Code auto memory처럼 목차만 넣을 가능성이 있지만 확인하지 못했다.
- 시간 처리에 대한 공개 자료가 없다. "7월에 갈 예정"이 지나면 어떻게 되는지 ChatGPT는 발표문에 적었는데 Claude는 없다.
- 채팅 검색이 키워드 검색인지 벡터 검색인지, 발췌문 길이가 얼마인지도 없다.
- 갱신할 때 옛 항목과 새 항목을 비교하는 절차가 있는지도 없다.
- 기억을 컨텍스트에 어떤 형식으로 넣는지, 답할 때 기억을 어떤 규칙으로 골라 쓰는지도 없다.

## **9. 정리**

Claude 메모리를 한 줄로 쓰면 이렇다. 사용자 기억은 카테고리별 항목으로 대화 중에 바로 쓰고 고치며, 과거 대화 원문은 검색 tool로 필요할 때만 꺼낸다.

[논문 21편 회고](https://momozzing.github.io/study/Agent-Memory-Retrospective/)에서 겹친 결론과 맞춰보면 두 가지가 보인다.

- 원문을 남기고 그 위에 검색을 얹는다. 기억 항목은 추출물이지만 원문 대화는 그대로 있고 tool로 직접 접근한다. ReFind, MemMachine이 간 방향이다.
- 기억을 항목 단위로 쪼개고 날짜를 붙여서 사용자가 하나씩 보고 지울 수 있게 했다. 합성 요약 하나로 두는 것보다 수정과 삭제가 명확하다.

반대로 논문에서 중요하게 본 갱신 검증(옛 버전과 비교)과 시간 경과 처리는 공개 자료가 없다. 하는지 안 하는지 모른다.

## **출처**

| ID | 자료 | 주체 · 날짜 | 수준 | 링크 |
|---|---|---|---|---|
| C1 | Release notes (Claude Help Center) | Anthropic · 상시 갱신 | 공식 | [링크](https://support.claude.com/en/articles/12138966-release-notes) |
| C2 | Use Claude's chat search and memory to build on previous context | Anthropic · 상시 갱신 | 공식 | [링크](https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context) |
| C3 | Claude's memory works everywhere, and you decide what's in it | Anthropic 블로그 · 2026-08-25 | 공식 | [링크](https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it) |
| C4 | Bringing memory to teams | Anthropic 블로그 · 2025-09-11 | 공식 | [링크](https://claude.com/blog/memory) |
| C5 | Memory tool | Claude Platform Docs | 공식 | [링크](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) |
| C6 | How Claude remembers your project | Claude Code Docs | 공식 | [링크](https://code.claude.com/docs/en/memory) |
| C7 | Claude 메모리 설정 화면 | 내 계정 · 2026-10-07 | 1차 관찰 | 개인 정보라 이미지 대신 구조만 실음 |
