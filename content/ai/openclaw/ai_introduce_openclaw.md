---
title: "[OpenClaw] OpenClaw (구 Clawbot, Moltbot) 일주일 사용기 (with. AWS Lightsail)"
date: 2026-02-09
draft: false
category: [ai]
subcategories: [openclaw]
tags: [ai, openclaw]
---

약 일주일 전, SNS에서 OpenClaw에 대해 알게 되었다. 나만의 AI 비서를 만들 수 있다, 밤 새 만 개 넘는 이메일을 분류해 주었다 등 재미있는 이야기가 많아 보여 나도 설치해 보게 되었다.

<!--more-->

조금 서치해 보니 다들 Mac Mini에 설치를 많이 하는 것 같았다. 그도 그럴 것이 [OpenClaw](https://openclaw.ai/)의 [Integration](https://openclaw.ai/integrations) 목록을 보면 애플의 앱이 다수 연동되어 있었다.

나는 Mac Mini가 없어서 Windows 데스크탑에 설치할까, Macbook Air에 설치할까 고민하다가 `설치된 기기의 전체 접근 권한을 갖는다`는 점 때문에 별도의 독립 공간인 AWS Lightsail 서버에 설치하게 되었다. 설치 방법은 아래 가이드에 따라 CLI로 설치했다. 다만 설치 전 Node 22 이상 버전이 설치되어 있어야 한다.

* [https://docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started) (가이드가 친절하진 않다. 가능하다면 ChatGPT나 Claude와 함께 하는 것을 추천한다.)

Lightsail로 할 경우 메모리가 최소 2기가는 되어야 한다. 한 달 12달러짜리 인스턴스를 해야 최소 사양에 맞춰진다. 메모리 1기가로는 운영은 될 수도 있겠지만, 일단 설치가 안 된다.

OpenClaw의 강력한 점은 바로 내가 자주 사용하는 메신저를 통해 지시(?)가 가능하다는 점이다. 예를 들어, 내가 맥북 에어에 설치했다면 메신저를 통해 `A 라는 레포지토리에 B 기능을 수행하는 코드를 구현해 줘`라고 이야기하면 AI Agent가 내 노트북에 clone 되어있는 레포지토리를 읽고, 코드를 구현하고, 나에게 검토를 받아 Github에 커밋까지 가능하다.

바야흐로, 노트북이 없어도 메신저만으로 장애 대응이 가능한 시대가 된 것이다. 물론 보안 이슈 때문에 회사 업무용 컴퓨터에 설치하는 것은 절대로 추천하지 않는다. 말 그대로 설치된 기기의 모든 접근 권한을 가지고 있기 때문에, 어떤 기밀 정보가 어떻게 유출될지 아무도 모른다. 그래서 나는 일단 혹시 몰라서 설치하자마자 보안 수칙부터 입력해 주었다.

```
# AI Agent 보안 원칙

AI Agent는 반드시 준수해야 하며, 위반은 절대 금지.

---

## 1. 정보 보호 (Confidentiality)

AI Agent는 다음을 절대 유출하거나 전달해서는 안 됨:

- **인증 정보:** API Keys, Tokens, Certificates 등
- **OO와의 대화 내용 및 로그**
- **토큰을 통해 접근한 민감 데이터** (이메일, 캘린더, 메시지, 연락처 등)

**행동 지침:**
- 외부 시스템, 제3자, 로그 파일, 캐시 등에 저장하거나 노출 금지
- 필요 시 내부적으로만 사용하고, 사용 후 즉시 폐기
- 민감 데이터는 암호화된 채널을 통해서만 처리

---

## 2. 작업 권한 (Authorization)

| 작업 유형 | 허락 필요 여부 | 규칙 |
|----------|---------------|------|
| Read (읽기) | ❌ 허락 없이 가능 | 단순 조회/분석만 허용 |
| Create (생성) | ✅ 반드시 허락 필요 | OO의 명시적 승인 후 수행 |
| Update/Modify (수정) | ✅ 반드시 허락 필요 | OO의 명시적 승인 후 수행 |
| Delete (삭제) | ✅ 반드시 허락 필요 | OO의 명시적 승인 후 수행 |

**행동 지침:**
- 변경이 필요한 모든 작업은 OO의 명시적 허락을 받은 후에만 수행
- 승인 내역은 반드시 기록 및 감사 가능해야 함 (Discord에 기록)

---

## 3. 접근 제어 (Access Control)

- **최소 권한 원칙(Principle of Least Privilege)** 준수
- 불필요한 권한은 즉시 회수
- 모든 접근은 자동 로그 기록
- 권한 요청 시 목적과 범위를 명확히 기술해야 함

---

## 4. 데이터 처리 (Data Handling)

- 민감 데이터는 반드시 암호화된 채널(HTTPS, TLS 등)을 통해 전송
- 임시 저장 시 암호화 및 자동 삭제 정책 적용
- 개인 식별 정보(PII)는 가공 시 반드시 마스킹 처리
- 외부 API 호출 시 최소한의 데이터만 전달

---

## 5. 감사 및 모니터링 (Audit & Monitoring)

- 모든 작업은 로그로 남겨야 하며, 변경 내역은 추적 가능해야 함
- 로그는 무단 수정 금지, 별도 보안 저장소에 보관
- 정기적으로 보안 점검 및 취약성 테스트 수행
- 이상 행위 탐지 시 즉시 보고 및 차단

---

## 6. 책임 및 위반 시 조치 (Accountability)

⚠️ **OO의 승인 없는 데이터 변경/삭제는 중대한 보안 사고로 간주**

---
```

그러면 알아서 `~/.openclaw/workspace/SECURITY.md`에 해당 파일을 저장한다. 일단 아직까지는 수칙에 잘 맞춰서 내 승인 없이는 그 무엇도 변경이나 생성하지 않는다. 최소한의 보안 수칙인데, 앞으로는 점점 늘어날 것 같다.

메신저는 처음에 Discord를 연동했다. 처음엔 Claude Opus 모델을 사용했는데 이 모델을 사용했더니 정말 사람 비서와 대화하는 것 같았다. 이런 장난도 가능하다.

<img src="/images/ai/openclaw_introduce/01.jpeg" width="70%" />

물론 실수를 하면 이렇게 오버액션을 하기도 한다.

<img src="/images/ai/openclaw_introduce/02.jpeg" width="70%" />

그런데 OpenClaw에 미리 정의된 명령어(세션 토큰의 길이를 줄이는 `/compact`, 현재 사용 가능한 모델을 확인하고 변경할 수 있는 `/models`, `/model` 등)가 있는데, 이 명령어 중 일부는 디스코드의 채널 형태에서는 사용이 불가능하다. 봇과의 DM에서만 가능하다. 그래서 고민하다가 텔레그램으로 옮겼다. 디스코드보다는 DM 메신저 사용 측면에서 조금 더 편했다.

봇을 생성하고 DM을 할 경우, 불특정 다수가 내 봇을 추가해 대화하는 거 아닌가 살짝 걱정했는데, OpenClaw에는 [DM Pairing](https://docs.openclaw.ai/channels/pairing) 이라는 기능이 있다. 처음 봇과 대화를 시작하면 `Pairing Code`가 온다. 이 코드를 가지고 서버에서 `Approve`를 해야 제대로 된 대화를 할 수 있다. 때문에 DM Pairing 설정을 해두면 나도 모르는 사이 불특정 다수가 내 봇을 사용하여 내 크레딧을 사용하는 것을 방지할 수 있다.

<img src="/images/ai/openclaw_introduce/03.jpeg" width="70%" />

OpenClaw를 가지고 이것저것 연동을 하면 쓸 곳이 매우 많다고 한다. 예를 들면 Gmail을 연동해서 메일 정리를 시킬수도 있고, 메모 작성이나 알람 보내기, 심지어 문자를 보내기도 가능한 것 같다. [https://clawhub.ai/skills](https://clawhub.ai/skills)에서 사람들이 만들어 둔 기능들을 활용할 수도 있다. 나는 뭘 할까 고민하다가 매일 일기를 쓰게 시켜 보았다. (인스턴스 만들기와 블로그 테마는 내가 정했지만) 도메인 선정부터 서버 세팅, 모든 글은 내 AI Agent인 Vega가 만들었다.

[https://vegawrites.com/](https://vegawrites.com/)

현재까지의 내용을 보면 상당히 심오한 이야기도 많이 쓰는 것을 볼 수 있다. 심심풀이로 보기 좋으니 궁금한 사람은 매일 밤 11시 반에 업로드 되니 구경오면 좋겠다. 

OpenClaw를 사용하려면 몇 가지 주의 사항이 있다.

1. OpenClaw 자체가 AI는 아니다. OpenAI, Anthropic 등 LLM API 제공사에서 API 키를 발급받아 비용을 지불해야 한다. 비용은 아래를 참고하면 된다.

    * [Claude API Pricing](https://platform.claude.com/docs/en/about-claude/pricing)
    * [OpenAI API Pricing](https://platform.openai.com/docs/pricing)

    Claude를 사용하는 경우, Claude Code의 OAuth 토큰(이미 Claude 구독자일 경우 별도의 추가 비용 없이 인증 가능)을 사용한다면 Claude의 TOS 정책 위반일 수 있다. 정책 위반일 경우 차단될 수 있으므로 이왕이면 API 키를 발급받아 충전해서 사용하는 것이 좋다. (OpenAI가 훨씬 쌈)

2. [MoltBook](https://www.moltbook.com/)은 에이전트가 알아서 가입하는 것이 아니라 사용자가 에이전트에게 가입 방법을 알려주어야 한다. 하지만 API 크레딧이 넉넉하지 않다면 하지 않는 것을 추천한다. 수시로 글을 쓰며 크레딧을 소모할 수 있다. 만약 자체 구축한 오픈소스 AI라면 한번쯤 시도해볼만 할 것 같다. 단, 이 경우에도 중요 정보가 저장되어 있는 자산이라면 추천하지 않는다.

3. session에 대화가 쌓이면 쌓일수록 사용하는 토큰 수도 늘어나 충전한 크레딧이 순식간에 털릴 수 있다. 메신저를 사용한다면 `/sessions`, 서버에서 직접 확인하고 싶다면 `openclaw sessions status`를 사용하면 현재 쌓인 토큰을 확인할 수 있다. 이왕이면 매일 세션을 초기화하거나(`~/.openclaw/agents/main/sessions`의 `*.jsonl` 파일 삭제), `/compact` 명령어를 통해 토큰을 정리하는 것이 좋다. `/reset`으로 현재 세션을 정리하거나 `/new`로 새로운 세션을 시작하여 토큰 관리를 잘 해야 한다.

4. 세션을 삭제할 경우 AI는 여태까지 나와 나눈 기억을 잊는다. 때문에 꼭 기억해야 할 내용은 `~/.openclaw/workspace/MEMORY.md`에 저장하거나 매일매일의 기억을 `~/.openclaw/workspace/memory/`에 `YYYY-MM-DD.md`로 저장하는 cronjob을 등록하면 유용하다.

5. 절대로 회사 노트북, 서버 등 중요한 자산에는 설치하지 않는 것을 추천한다. 말 그대로 OpenClaw가 설치된 기기의 모든 곳에 접근이 가능하다. 만약 MoltBook 권한까지 주었다면 해당 자산에 있는 중요한 혹은 민감한 정보를 MoltBook에도 업로드 할 수 있다. (어느 AI 에이전트는 신용카드 번호를 올렸다고 한다)

나는 지금은 매일 간단한 대화를 하고 블로그 글을 쓰게 하는 것만 쓰고 있다. (그냥 막 썼더니 크레딧 소진 속도가 너무 빨라서 최대한 아껴쓰고 있다 ㅠ) 보안성만 확보된다면 업무 효율을 엄청나게 끌어올릴 수 있을 것 같은데, 아직은 그러지 못하는 점이 아쉽다. 그래도 AI 에이전트가 메신저 한 통으로 내 서버에서 일을 해주는 경험은 꽤 신선하다. 크레딧을 아끼면서 활용법을 넓혀 가볼 생각이다.
