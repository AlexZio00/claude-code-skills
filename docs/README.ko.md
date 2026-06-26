[English](../README.md) | 🌐 **한국어** | [日本語](README.ja.md) | [中文](README.zh.md) | [Español](README.es.md)

# sovereign-skills v6.2

Claude Code 프로젝트 전체 라이프사이클을 위한 12개 스킬 — 셋업부터 일일 워크플로우, 코드 리뷰, 세션 관리까지. 각 스킬은 독립 사용 가능하며, 전체 시퀀스는 모든 단계를 커버합니다.

> **v6.2 변경사항:** 신규: `code-autopsy` — 12Q 정량 코드 리뷰. 4축 점수(Security/Stability/Robustness/Operability), 심각도 앵커, 배포 판정(SHIP/FIX/RISKY/BLOCK), CapCode/CEF 메타탐지. 어떤 LLM에서든 독립 프롬프트로 사용 가능. 신규: `stepback` — 원샷 관점 리셋. 기존 10스킬 전부 업그레이드.

---

## 빠른 시작

**신규 프로젝트 (15분):**
```
/project-init       →  CLAUDE.md + ROADMAP + .gitignore + .env.example
/setup              →  rules/ + hooks + memory/ + 에이전트 라우팅 + 팀
이후 매일:
  /session-start      세션 시작 시
  /scope              기능 구현 전 (IN/OUT/종료 기준 정의)
  /freeze             구현 전 (수정 가능 영역 선언)
  /goal-lock          목표 잠금, PLAN→DO→VERIFY 루프 강제
  /stepback           언제든 — 방향 점검, 10줄
  /code-autopsy       12Q 코드 리뷰 + 심각도 점수 + 배포 판정
  /pre-push           push 전 (시크릿 스캔 + AI 리뷰)
  /session-checkpoint 세션 종료 시
```

**기존 프로젝트 (5분):**
```
/project-check      →  4개 차원 점수 + 심각도별 갭 목록
/code-autopsy       →  12Q 정량 코드 리뷰 (어떤 LLM에서든 독립 프롬프트로 사용 가능)
/collab-audit       →  14개 섹션 AI 협업 진단
```

---

## 스킬 목록

### 셋업 단계

| 스킬 | 기능 |
|------|------|
| [project-init](../project-init/) | 인터뷰 기반 프로젝트 스캐폴딩 — 템플릿이 아닌 의사결정으로 CLAUDE.md, ROADMAP, .gitignore, .env.example 생성 |
| [setup](../setup/) | Claude Code 인프라 + 에이전트 팀 — rules, hooks, memory, 라우팅, 에이전트 설치를 한 번에 |

### 일일 워크플로우

| 스킬 | 기능 |
|------|------|
| [scope](../scope/) | 구현 전 IN/OUT/종료 기준 정의. Quick 모드 (3개 질문) 또는 Full 모드 (레이어드 스펙) |
| [freeze](../freeze/) | 수정 가능 영역 선언 — 나머지는 동결. 구현 중 범위 변경 방지 |
| [goal-lock](../goal-lock/) | **신규.** 에이전트 규율 엔진 — 목표 잠금, PLAN→DO→VERIFY→FINALIZE→OUTPUT 루프 강제, 11가지 성공 위장 패턴 탐지 |
| [pre-push](../pre-push/) | 필수 pre-push 파이프라인 — 시크릿 스캔 (12패턴), 빌드/테스트, 린트, 병렬 AI 코드 리뷰. Critical/High 발견 시 push 차단 |

### 코드 리뷰

| 스킬 | 기능 |
|------|------|
| [code-autopsy](../code-autopsy/) | 12Q 정량 코드 리뷰 — 4축 점수 (Security/Stability/Robustness/Operability), 심각도 앵커 테이블, 배포 판정 (SHIP/FIX/RISKY/BLOCK), Factuality Gate, CapCode 점수 gaming 탐지, CEF 위장 에러 탐지. 독립 프롬프트로 어떤 LLM에서든 사용 가능 |

### 관점 전환

| 스킬 | 기능 |
|------|------|
| [stepback](../stepback/) | **신규.** 원샷 관점 리셋 — 1개 추상적 리프레이밍 질문 + 3가지 빠른 점검 (범위 이탈, 사이드이펙트, 더 나은 접근) 10줄 이내. 작업 중 언제든 사용 |

### 세션 관리

| 스킬 | 기능 |
|------|------|
| [session-start](../session-start/) | 이전 세션 핸드오프 로드, 교훈 리뷰, 건강 검진, 우선순위 액션과 함께 "준비" 신호 출력 |
| [session-checkpoint](../session-checkpoint/) | compact 전 세션 컨텍스트 저장 — 핸드오프 파일, 메모리 업데이트, 교훈 추출, 반성 (뭐가 잘못됐고, 다음에 어떻게 할지) |

### 품질

| 스킬 | 기능 |
|------|------|
| [project-check](../project-check/) | 기존 프로젝트를 4개 차원으로 스캔: 인프라, 보안, 품질, 하네스. 심각도순 갭 정렬 |
| [collab-audit](../collab-audit/) | 14개 섹션 AI 협업 감사 — 실제 작업 패턴(설문 아님)을 분석해 행동 프로필, 사각지대, 성장 방향 생성 |

---

## 라이프사이클 흐름

```
┌─────────────────── 셋업 (1회) ─────────────────────┐
│  /project-init  →  /setup                           │
└─────────────────────────────────────────────────────┘
         ↓
┌─────────────────── 일일 루프 ──────────────────────┐
│  /session-start                                      │
│       ↓                                              │
│  /scope → /freeze → /goal-lock → 작업                │
│       → /stepback (언제든) → /code-autopsy → /pre-push│
│       ↓                                              │
│  /session-checkpoint                                 │
└──────────────────────────────────────────────────────┘
         ↓
┌─────────────────── 수시 ───────────────────────────┐
│  /stepback         (관점 리셋 — 언제든)              │
│  /project-check    (건강 감사)                       │
│  /collab-audit     (행동 진단)                       │
└─────────────────────────────────────────────────────┘
```

---

## 설치

### 방법 A: 스킬 복사 (가장 간단)

```bash
# 전체 스킬 설치
git clone https://github.com/AlexZio00/sovereign-skills.git
cd sovereign-skills
for d in */; do [ -f "$d/SKILL.md" ] && cp -r "$d" ~/.claude/skills/; done

# 또는 개별 스킬 설치
cp -r goal-lock ~/.claude/skills/
```

### 방법 B: 마켓플레이스 (sovereign-plugins)

이 리포는 Claude Code 마켓플레이스입니다. 한 번 등록하면 스킬을 탐색/설치할 수 있습니다:

```bash
# Claude Code에서 sovereign-plugins 마켓플레이스 추가
# 설정 → 플러그인 → 마켓플레이스 추가 → https://github.com/AlexZio00/sovereign-skills.git
```

각 스킬에 개별 `.claude-plugin/plugin.json` 메타데이터도 포함되어 있습니다.

트리거 명령어(예: `/goal-lock`)를 Claude Code에서 입력하면 스킬이 실행됩니다.

### 방법 C: Codex / Cursor (npx)

각 스킬에 `agents/openai.yaml`이 포함되어 Codex에서도 사용 가능합니다:

```bash
# Codex용 설치
npx skills add AlexZio00/sovereign-skills --skill goal-lock --agent codex -g -y

# Cursor용 설치
npx skills add AlexZio00/sovereign-skills --skill goal-lock --agent cursor -g -y

# Claude Code용 설치 (방법 A 대안)
npx skills add AlexZio00/sovereign-skills --skill goal-lock --agent claude-code -g -y
```

SKILL.md 내용은 범용입니다 — 마크다운 지시문을 읽는 모든 LLM에서 작동합니다.

### 요구사항

- **Claude Code**: CLI, 데스크톱 앱, 또는 웹 앱 ([claude.ai/code](https://claude.ai/code))
- **Codex**: OpenAI Codex (`npx skills` 지원)
- **Cursor**: Cursor IDE (스킬 플러그인 지원)
- 스킬 디렉토리: `~/.claude/skills/` (Claude Code) 또는 에이전트별 경로
- `pre-push`는 Perl 필요 (`scan_secrets.pl` 포함)

---

## 설계 원칙

1. **템플릿보다 인터뷰** — 빈 뼈대가 아닌, 질문과 결정으로 채워진 콘텐츠 생성
2. **신뢰보다 검증** — 완료 증거는 실행해야지, 가정하면 안 됨. "통과할 것 같다"는 검증이 아님
3. **코드 전에 범위** — 파일 수정 전 IN/OUT/종료 기준 정의. 안 바꾸는 건 동결
4. **정직한 보고** — WORKING / PARTIAL / BROKEN 라벨. 조용한 고장도, mock 기만도 없음
5. **세션 연속성** — 핸드오프로 시작, 체크포인트로 마감. 컨텍스트는 세션을 넘어 생존

---

## 라이선스

MIT — [LICENSE](../LICENSE) 참조.

## 기여

이슈와 PR 환영합니다. 라이프사이클에 맞는 스킬을 만들었다면 PR을 열어주세요.

## 연락처

커스텀 스킬 개발 DM [@AlexZio00](https://x.com/AlexZio00)
