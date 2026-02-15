---
title: "[Claude Code] 해커톤 우승자의 70가지 파워 팁 요약"
date: 2026-02-11
draft: false
category: [ai]
subcategories: [claude-code]
tags: [ai, claude-code, productivity, tips]
---

최근 Manus AI에서 발행한 "Claude Code 완전 가이드: 해커톤 우승자의 70가지 파워 팁" PDF를 읽었다. Anthropic 해커톤 우승자 ykdojo의 43가지 팁과 Anthropic DevRel Ado Kukic의 31가지 팁을 집대성한 가이드인데, 핵심 내용을 정리해 보았다.

<!--more-->

## 1. 에이전틱 개발자의 사고방식

Claude Code를 잘 활용하기 위해 가장 먼저 필요한 것은 **사고방식의 전환**이다.

### 큰 문제를 분해하라

"로그인 페이지를 만들어줘" 같은 막연한 요청 대신, 작업을 잘게 쪼개서 지시해야 한다. 예를 들어 DB 스키마 설계 -> ORM 마이그레이션 -> UI 컴포넌트 -> API 로직 -> 테스트 순서로 단계별로 요청하는 것이 훨씬 정확한 결과를 얻을 수 있다.

실제로 테스트해 보았을 때, `이 코드를 마이그레이션 해줘`보다는 아래처럼 구체적으로 지시하는 것이 훨씬 정확도가 높았다.

```plain
XX 경로의 소스코드들을 읽고 아래와 같이 마이그레이션 해줘.
1. 전체 흐름은 아래와 같아.
  - QQ에서 데이터 읽기 (기존 코드의 WW 함수 참고)
  - 해당 데이터를 참고하여 json 데이터 만들기 (기존 코드의 NN 줄 참고)
2. 단, 이때 데이터 중 ZZ는 N:N 방식이므로 여러 데이터를 포함하기
```

이처럼 태스크를 쪼개고, 참고할 함수나 라인 번호, 주의할 점을 명시하면 오류 확률이 현저히 줄어든다.

### 계획 모드 vs YOLO 모드

- **계획 모드 (Plan Mode)**: `Shift+Tab`을 두 번 누르면 진입. Claude가 코드를 분석하고 계획을 세우지만, 승인 전까지 아무것도 편집하지 않는다. 복잡한 작업, 대규모 리팩토링에 적합하다.
- **YOLO 모드**: `claude --dangerously-skip-permissions`로 실행. 모든 작업을 자동 승인한다. 반드시 컨테이너 안에서만 사용할 것.

YOLO 모드는 회사 업무에는 부적합하다. 회사 업무라면 Plan Mode를 사용할 것을 적극 권장한다.

### 컨텍스트 관리가 핵심

ykdojo의 비유로 "AI 컨텍스트는 우유와 같다." 신선하고 압축된 상태를 유지해야 한다. 주요 전략은 다음과 같다.

- **단일 목적 대화**: 하나의 세션에서는 하나의 작업에만 집중
- **HANDOFF.md**: 대화가 길어지면, 작업 내용을 정리한 파일을 생성하고 `/clear` 후 새 세션에서 이어가기
- **/context**: 컨텍스트 윈도우의 사용 현황을 X-Ray처럼 확인

대화가 너무 길어지거나 여러 주제를 한 세션에서 다루면 결과물이 꼬이는 경우가 자주 발생한다. 프롬프트를 매번 새로 입력하는 것이 번거롭긴 하지만, 최상의 결과물을 얻기 위해 컨텍스트 관리는 필수다.

## 2. 환경 설정과 필수 명령어

### CLAUDE.md 활용

프로젝트 루트에 `CLAUDE.md`를 작성하면 Claude가 프로젝트의 기술 스택, 코딩 스타일, 금지 사항 등을 파악한다. `/init` 명령어로 자동 생성도 가능하다. 단, 간결하게 유지하는 것이 중요하다.

### 터미널 별칭 설정

```plain
alias c='claude'
alias cc='claude --continue'
alias cr='claude --resume'
alias ch='claude --chrome'
```

### 세션 관리

- `claude --continue`: 마지막 대화 이어가기
- `/rename auth-system-refactor`: 세션에 이름 붙이기
- `claude --resume auth-system-refactor`: 이름으로 세션 재개
- `/export`: 대화 내역을 마크다운으로 내보내기

## 3. 생산성 극대화 기술

### 음성 코딩

타이핑(분당 40단어)보다 말하기(분당 150단어)가 3.75배 빠르다. superwhisper 같은 도구를 활용하면 음성으로 Claude에게 지시할 수 있다.

### 필수 키보드 단축키

| 단축키 | 기능 |
|--------|------|
| `Esc Esc` | 대화/코드 되감기 (Undo) |
| `Ctrl+R` | 역방향 검색 (명령어 히스토리) |
| `Ctrl+S` | 프롬프트 임시 저장 |
| `Shift+Tab` (x2) | Plan 모드 토글 |
| `Ctrl+B` | 백그라운드로 보내기 |
| `Ctrl+G` | 외부 에디터로 편집 |
| `!command` | Bash 명령어 즉시 실행 |

`!` prefix를 사용하면 Claude의 처리 없이 즉시 셸 명령을 실행하고 결과를 컨텍스트에 주입할 수 있다. 토큰 낭비를 방지하는 데 유용하다.

## 4. Git과 GitHub 워크플로우

### 자동 커밋 & PR 생성

Claude에게 "변경 사항을 분석하고 커밋해줘", "draft PR을 만들어줘"와 같은 요청이 가능하다.

### Git Worktrees

여러 브랜치에서 동시에 작업해야 할 때 `git worktree`를 사용하면 브랜치를 전환하지 않고 병렬로 작업할 수 있다. 각 worktree를 별도의 터미널 탭에서 독립적인 Claude 세션으로 관리하면 효율적이다.

### 승인된 명령어 감사 (cc-safe)

ykdojo가 만든 `cc-safe` 도구로 위험한 승인 명령어를 감지할 수 있다. `sudo`, `rm -rf`, `git push --force` 등 위험한 패턴을 자동으로 스캔해 준다.

## 5. 고급 기능

### MCP (Model Context Protocol)

Claude가 외부 서비스와 직접 통신할 수 있게 해주는 프로토콜이다.

- **Playwright MCP**: 브라우저 자동화, E2E 테스트
- **Supabase MCP**: 데이터베이스 직접 쿼리
- **Firecrawl MCP**: 웹 크롤링

단, 활성화된 MCP가 많으면 컨텍스트를 많이 차지하므로 10개 미만의 MCP, 80개 미만의 활성 도구를 권장한다.

### Hooks

특정 이벤트 발생 시 자동으로 실행되는 셸 명령어. AI의 확률적 행동에 결정론적 제어를 추가할 수 있다. 위험한 명령어 차단, 로그 기록 등에 활용 가능하다.

### Skills & Agents

- **Skills**: 재사용 가능한 지식 조각. Claude가 필요 시 자동 호출하거나 수동 호출 가능
- **Agents (서브에이전트)**: 각자 독립적인 200k 컨텍스트를 가진 전문화된 AI 프로세스. 병렬 실행 가능

## 6. 컨테이너와 샌드박스

YOLO 모드는 반드시 Docker 컨테이너 안에서만 사용해야 한다. ykdojo는 "보호되지 않은 성관계"에 비유하며, 호스트 시스템에서 직접 사용하지 말 것을 강력히 권장한다.

고급 활용으로는 메인 Claude가 tmux를 통해 컨테이너 안의 "워커" Claude에게 작업을 위임하는 멀티 에이전트 오케스트레이션이 가능하다.

## 7. 실전 활용 사례

### 작성-테스트 사이클

코드 작성 후 테스트까지 한 번에 요청하면, Claude가 테스트 실패 시 코드를 자동 수정하고 모든 테스트가 통과할 때까지 반복한다.

### DevOps 엔지니어로 활용

GitHub Actions 실패 분석, Docker 이미지 최적화, CI/CD 디버깅 등을 Claude에게 맡길 수 있다.

### 범용 인터페이스

ffmpeg를 이용한 비디오 편집, Whisper를 이용한 오디오 전사, 디스크 공간 정리 등 컴퓨터에서 하고 싶은 거의 모든 작업을 Claude Code를 통해 수행할 수 있다.

### 출력 검증

AI가 생성한 코드를 맹목적으로 신뢰하면 안 된다. 테스트 코드 작성, GitHub Desktop으로 시각적 검토, Draft PR 생성, Claude에게 자기 검증 요청 등 여러 방법을 병행해야 한다.

## 8. 시스템 최적화

### 시스템 프롬프트 슬림화

ykdojo는 58개의 패치를 통해 시스템 프롬프트를 19k 토큰에서 10k 이하로 줄였다. 이를 통해 더 많은 코드와 대화를 컨텍스트에 담을 수 있고, 응답 속도도 향상된다.

### 자동화의 자동화

ykdojo의 자동화 여정은 인상적이다.

1. ChatGPT에서 코드 복사/붙여넣기
2. Claude Code로 터미널 통합
3. 음성 전사 시스템으로 타이핑 자동화
4. CLAUDE.md로 반복 지시 자동화
5. Skills로 Claude의 자동 판단 자동화
6. Hooks로 규칙 강제 자동화

**"같은 작업을 3번 이상 반복한다면, 자동화할 방법을 찾으세요. 그리고 그 자동화 과정 자체도 자동화하세요."**

## 9. 학습 로드맵

### 초급 (1-3개월)
- 기본 명령어 익히기, CLAUDE.md 작성
- 컨텍스트 관리 기초, Git 통합
- 터미널 별칭, 키보드 단축키 설정

### 중급 (3-12개월)
- MCP 서버 연동, Hooks 설정
- Skills/Slash Commands 제작
- 컨테이너 워크플로우, 서브에이전트 활용

### 고급 (1년 이상)
- 시스템 프롬프트 패치, 맞춤형 MCP 서버 개발
- 멀티 에이전트 오케스트레이션
- Claude Agent SDK 활용, CI/CD 통합

## 핵심 요약 5가지

1. **큰 문제를 작은 단위로 분해하라** - AI는 명확한 지시에서 최고 성능을 발휘한다
2. **컨텍스트는 우유와 같다** - 신선하고 압축된 상태를 유지하라
3. **올바른 추상화 수준을 선택하라** - Vibe Coding과 Deep Dive를 상황에 맞게
4. **자동화의 자동화** - 반복 작업은 자동화하고, 자동화 과정도 자동화하라
5. **사용이 최고의 학습** - 직접 사용하는 것이 가장 좋은 배움이다

자세한 내용은 [GeekNews](https://news.hada.io/topic?id=26526)에서 확인할 수 있다.

## 참고 자료

- [ykdojo claude-code-tips](https://github.com/ykdojo/claude-code-tips)
- [Ado의 Advent of Claude 2025](https://adocomplete.com/advent-of-claude-2025/)
- [Anthropic 공식 문서](https://code.claude.com/docs/en/overview)
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
