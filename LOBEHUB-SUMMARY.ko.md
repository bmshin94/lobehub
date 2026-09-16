# LobeHub 분석 정리 (한국어)

> LobeHub 레포지토리를 직접 분석하고 정리한 문서입니다.
> 무엇을 하는 프로젝트인지, 어떻게 설치·활용하는지, 그리고 수익화 가능성까지 다룹니다.

---

## 🔗 링크 모음

| 구분 | 주소 |
| --- | --- |
| **내 포크(fork)** | https://github.com/bmshin94/lobehub |
| **원본 저장소(upstream)** | https://github.com/lobehub/lobehub |
| 공식 사이트 | https://lobehub.com |
| 공식 문서 | https://lobehub.com/docs |
| 이슈 / 피드백 | https://github.com/lobehub/lobehub/issues |
| 상업 라이선스 문의 | hello@lobehub.com |

### 생태계 패키지 (별도 저장소)

| 패키지 | 저장소 | 설명 |
| --- | --- | --- |
| `@lobehub/ui` | https://github.com/lobehub/lobe-ui | AIGC 웹앱용 UI 컴포넌트 라이브러리 |
| `@lobehub/icons` | https://github.com/lobehub/lobe-icons | AI / LLM 브랜드 SVG 아이콘 모음 |
| `@lobehub/tts` | https://github.com/lobehub/lobe-tts | TTS / STT React Hooks |
| `@lobehub/lint` | https://github.com/lobehub/lobe-lint | ESLint·Prettier 등 설정 모음 |

---

## 1. 이게 뭐 하는 프로젝트야?

한 줄로: **"내가 직접 설치해서 쓰는 ChatGPT + AI 에이전트 팀 관리 플랫폼"**

원래는 `LobeChat`이라는 ChatGPT 클론이었는데, 지금은 **"AI 직원들을 고용하고 스케줄을 돌리는 사무실"** 컨셉으로 진화했습니다.

현재 버전: `v2.2.17`

### 핵심 기능 4가지

| 기능 | 설명 |
| --- | --- |
| 🧑‍💼 **Operator** | 에이전트들을 한곳에서 관리 + 메신저로도 호출 |
| 🛠 **Create** | 말로 설명하면 에이전트 자동 생성(Agent Builder), 스킬/MCP 플러그인 연동 |
| 👥 **Collaborate** | 에이전트 팀 협업, 문서 공동 작성(Pages), 예약 실행(Schedule) |
| 🧠 **Evolve** | 개인 메모리 — 사용자를 학습하되, 내가 직접 열어서 수정 가능(화이트박스) |

---

## 2. 폴더 구조

```
lobehub/
├── apps/            앱 6개
│   ├── desktop/     Electron 데스크탑 앱 (Mac/Win)
│   ├── cli/         터미널 명령어 (lh / lobe / lobehub)
│   ├── server/      백엔드 서버 (Hono)
│   ├── auth/        로그인 화면
│   ├── share/       대화 공유 페이지
│   └── workbench/   작업 공간
│
├── packages/        패키지 99개
│   ├── builtin-tool-*        내장 도구 30개+ (브라우저, 계산기, 이미지생성,
│   │                         메모리, 지식베이스, 노트북, 스킬스토어 등)
│   ├── chat-adapter-*        메신저 연동 (feishu / imessage / line / qq / wechat)
│   ├── agent-runtime/        에이전트 실행 엔진 (핵심!)
│   ├── heterogeneous-agents/ 외부 CLI 에이전트 연동 (Claude Code, Codex, Cursor 등)
│   ├── model-bank/           AI 모델 정보 (프로바이더 87개)
│   ├── database/             Drizzle ORM + PostgreSQL
│   └── locales/              다국어 18개 (ko-KR 포함)
│
├── src/             프론트엔드 (features 폴더 158개)
├── e2e/             E2E 테스트 (Cucumber + Playwright)
├── docker-compose/  도커 배포 설정
└── .agents/skills/  AI 코딩 에이전트용 가이드 48개
```

**규모:** 파일 16,748개 / `.ts` 10,069개 + `.tsx` 3,660개

---

## 3. 기술 스택

- **Next.js 16 + React 19 + TypeScript** (프론트는 Vite SPA로 별도 빌드)
- **zustand** (상태관리) + **SWR** (데이터 패칭) + **tRPC** (타입 안전 API)
- **Drizzle ORM + PostgreSQL**, 테스트는 **Vitest**
- UI: `@lobehub/ui` + antd + antd-style
- 패키지 매니저 `pnpm`, 스크립트 실행 `bun`

---

## 4. 설치 및 사용법

### A. 그냥 웹으로 쓰기
설치 없이 https://lobehub.com 에서 가입. 구경용으로 제일 빠릅니다.

### B. 도커로 설치 (추천)

**맛보기 버전 (DB 없이):**
```bash
docker run -d -p 3210:3210 \
  -e OPENAI_API_KEY=sk-xxxxx \
  lobehub/lobehub
```
→ `localhost:3210` 접속

**제대로 된 버전 (DB + 파일저장 + 검색):**
```bash
mkdir lobehub-db && cd lobehub-db
bash <(curl -fsSL https://lobe.li/setup.sh)
docker compose up -d
```

| 구성요소 | 역할 | 필수 여부 |
| --- | --- | --- |
| PostgreSQL | 대화/에이전트/파일 저장 | 필수 |
| Redis | 세션·캐시 | 선택 |
| RustFS / MinIO (S3) | 파일 업로드, 지식베이스 | 선택 |
| Searxng | 프라이버시 보호형 웹 검색 | 선택 |

### C. 개발자 모드

```bash
pnpm install       # 의존성 설치
bun run dev:spa    # 프론트만 (가볍고 빠름)
bun run dev        # 풀스택
bun run check      # 커밋 전 검사 (lint + test)
```

### D. 기타 배포 플랫폼
`docs/self-hosting/platform/` 안에 **Vercel, Docker, Docker Compose, Zeabur, Sealos, Dokploy, RepoCloud** 가이드가 모두 포함되어 있습니다.

### 브랜치 전략
- `canary` = 개발 브랜치 (클라우드 프로덕션)
- `main` = 릴리즈 브랜치
- 새 작업은 `canary`에서 분기, PR도 `canary`로

---

## 5. 플러그인? 스킬? MCP? → 전부 지원

LobeHub는 "꽂는 쪽"이 아니라 **"꽂히는 본체(호스트)"** 입니다.

| 종류 | 설명 | 코드 위치 |
| --- | --- | --- |
| **내장 도구** | 처음부터 포함된 기본 기능 30개+ | `packages/builtin-tool-*` |
| **스킬** | 에이전트용 매뉴얼 묶음 (agent-browser, artifacts, lobehub, task) | `packages/builtin-skills/` |
| **플러그인** | 외부 개발자용 확장. 전용 SDK + 마켓 존재 | 별도 저장소 |
| **MCP** | Model Context Protocol 표준 지원 | `packages/types/src/plugins/mcp.ts` 등 |

### MCP 지원 근거 (실제 코드 확인)
- `@modelcontextprotocol/sdk ^1.30.0` 의존성 포함 (웹 + 데스크탑 양쪽)
- DB에 `user_connectors` 테이블 → MCP 서버 연결 정보 + OAuth/OIDC 인증 저장
- `packages/heterogeneous-agents/src/builtinMcp/LobeBuiltinMcpServer.ts`
  → **LobeHub 자신이 MCP 서버 역할도 수행** (외부 에이전트가 붙을 수 있음)

---

## 6. API 토큰이 필요한가?

**필요합니다.** 모델 제공자의 API 키가 있어야 AI와 대화할 수 있습니다.

```bash
OPENAI_API_KEY=sk-xxxxxxxxx     # OpenAI
ANTHROPIC_API_KEY=sk-ant-xxx    # Claude
GOOGLE_API_KEY=xxx              # Gemini
```

`.env.example`에 설정 가능한 환경변수가 **110개** 있습니다 (쓰는 것만 채우면 됨).

### 무료로 쓰는 방법 — Ollama (로컬 모델)

```bash
docker run -d -p 3210:3210 \
  -e OLLAMA_PROXY_URL=http://host.docker.internal:11434 \
  lobehub/lobehub
```
→ API 키 0원, 토큰 비용 0원. 단, PC 사양이 좋아야 하고 성능은 상용 모델보다 아쉽습니다.

### 그 외 선택적으로 필요한 키
- **S3 키** — 파일 업로드 기능
- **Clerk 키** — 로그인 기능
- 혼자 쓸 거면 둘 다 생략 가능

---

## 7. 왜 GitHub에서 유명할까?

### 확인된 인기 지표
- **Trendshift** 뱃지 + **ProductHunt 일간 1위** 뱃지
- **Vercel OSS 프로그램** 공식 선정
- 18개 언어 지원 / 87개 AI 프로바이더 지원
- Docker 이미지 다운로드·기여자·포크 뱃지 다수

### 인기 이유
1. **UI 완성도** — 자체 디자인 시스템(`@lobehub/ui`) + 디자인 철학 문서(`DESIGN.md`: 자연/의미감/확실성/성장)
2. **설치가 쉬움** — 버튼 한 번으로 배포. 비개발자도 가능
3. **모델 통합** — 여러 AI를 한 화면에서. 신모델 대응도 빠름
4. **데이터 소유권** — 자체 호스팅이라 기업 도입 명분이 생김
5. **생태계** — lobe-ui / lobe-icons / lobe-tts / lobe-lint 각각이 독립적으로 유명
6. **개발 속도** — CHANGELOG가 84,000자, 거의 매일 업데이트

---

## 8. 로컬 에이전트 구축에 도움이 될까? → 매우 도움됨

### 보물 1: `packages/heterogeneous-agents`
외부 CLI 에이전트를 하나의 UI로 조종하는 **완성된 실전 예제**입니다.

```
claudeCodeDirectEnv.ts    Claude Code 연동
codex/                    OpenAI Codex 연동
cursorAcpSession.ts       Cursor 연동
devinAcpSession.ts        Devin 연동
grokAcpSession.ts         Grok 연동
droidAcpSession.ts        Droid 연동
traeAcpSession.ts         Trae 연동
builtinMcp/               자체 MCP 서버
subagentCoordinator/      서브에이전트 관리
```

### 보물 2: `packages/agent-runtime`
```
core/                 에이전트 핵심 엔진
loop/                 에이전트 루프 (생각 → 도구실행 → 반복)
executors/            도구 실행기
groupOrchestration/   멀티 에이전트 협업
audit/                실행 기록
```
에이전트 개발에서 가장 어려운 **루프 설계**의 검증된 구현체가 통째로 있습니다.

### 보물 3: 안전장치
- `builtin-tool-local-system/` — 로컬 셸 실행 + `interventionAudit`(위험 명령 가로채기)
- `agent-signal` — 에이전트 트리거 시스템
- `agent-tracing` — 실행 추적
- `device-sandbox`, `python-interpreter` — 격리 실행

### 보물 4: CLI
`apps/cli`에 `lh` / `lobe` / `lobehub` 명령어. 터미널에서 에이전트 생성, 봇 실행, 평가(eval)까지 가능.

### 현실적인 접근법
파일이 16,748개라 통째로 쓰려면 압도당합니다. **참고서로 활용**하는 것을 권장:
1. `agent-runtime/loop` → 에이전트 루프 개념 습득
2. `heterogeneous-agents/adapters` → 외부 에이전트 연동 방식 학습
3. 그 구조만 참고해서 내 작은 프로젝트로 새로 구현

---

## 9. ⚠️ 라이선스 (수익화 전 필독)

`package.json`에는 MIT라고 쓰여 있지만, 실제 `LICENSE` 파일은 **"LobeHub Community License"** (Apache 2.0 기반 + 추가 조건)입니다.

| 하는 일 | 가능 여부 | 근거 |
| --- | --- | --- |
| 소스 **수정 없이** 상업 서비스로 운영 | ✅ 가능 | *"may be utilized commercially... **without modifying the source code**"* |
| 소스 **수정해서** 파생 제품 배포·판매 | ❌ 상업 라이선스 필요 | *"a commercial license must be obtained... **derivative work**"* |
| 로고·이름 바꿔서 자체 제품으로 판매 | ❌ 위험 | 파생 저작물에 해당 |
| **플러그인 / MCP 서버 / 스킬** 별도 제작·판매 | ✅ 완전 자유 | 별도 저작물 |
| 설치·구축·컨설팅 용역 | ✅ 가능 | 코드 배포가 아닌 용역 |

> **핵심: "코드를 팔면 ❌, 코드 주변에서 벌면 ✅"**

상업 라이선스 문의: `hello@lobehub.com`

---

## 10. 수익화 아이디어 8가지

### TIER 1 — 즉시 시작 가능

#### ① 사내 AI 구축 대행
- **타겟:** 중소기업, 병원, 학원, 법무법인
- **배경:** "ChatGPT 사내 사용 금지" 공지는 많은데 직원들은 AI를 쓰고 싶어함 → 자체 호스팅 수요
- **작업:** 서버 준비 → docker compose 설치 → API 키·사용량 한도 세팅 → 부서별 에이전트 구성 → 직원 교육
- **수익 구조:** 구축비(일회성) + **유지보수 월정액(핵심)**
- **라이선스:** ✅ 안전 (코드 수정 0줄)
- **시작법:** 지인 회사 1곳에 무료 설치 → 사례 확보 → 영업

#### ② 매니지드 호스팅
- "설치가 귀찮은 분들 대신 운영해드립니다" 월 구독
- 서버 1대에 도커 컨테이너로 다수 고객 분리 → 원가 절감
- **차별점:** 한국 서버(속도 + 국내 데이터 보관), 한국어 지원, API 키 대행 발급

---

### TIER 2 — 1~2개월 준비

#### ③ 🇰🇷 카카오톡 어댑터 (최대 기회)

**발견한 사실:** 메신저 어댑터가 `feishu / imessage / line / qq / wechat` 5개인데
**카카오톡이 없습니다.** 코드 전체에서 `kakao` 검색 결과 **0건**.

**왜 기회인가:**
- 한국 메신저 = 카카오톡인데 미지원 → 명확한 시장 공백
- LINE 어댑터 전체가 **1,288줄** → 혼자 2~4주면 구현 가능한 규모
- 파일 구성도 단순: `adapter.ts` / `api.ts` / `format-converter.ts` / `types.ts` / `index.ts`
- ①번 구축 대행 시 **"카톡으로 사내 AI 호출"** 이라는 킬러 세일즈 포인트 확보

**수익 경로:**
1. 오픈소스 공개 → 본체에 PR 기여 → "LobeHub 컨트리뷰터" 타이틀 → 영업력 강화
2. 비공개 유지 → 내 구축 서비스만의 독점 기능
3. 카톡 챗봇 구축 자체를 상품화

**라이선스:** ✅ 안전 (별도 패키지, 본체 수정 최소화)

#### ④ MCP 서버 개발·판매 (범용성 최고)
- **장점:** MCP는 표준이라 LobeHub뿐 아니라 **Claude, Cursor, Windsurf 등 어디든** 연결 → 시장이 훨씬 넓음
- **라이선스 제약 없음**

**한국 특화 아이디어:**

| MCP 아이디어 | 타겟 고객 |
| --- | --- |
| 국세청 홈택스 연동 | 세무사, 소상공인 |
| 부동산 실거래가 조회 | 공인중개사, 투자자 |
| 배민/쿠팡이츠 매출 분석 | 자영업자 |
| 대법원 판례 검색 | 법무법인 |
| 나라장터 입찰공고 알림 | 조달 참여 기업 |
| 건강보험·병원 청구 | 의원, 병원 |

- **유통:** 코드에서 확인된 **LobeHub Market**(검색·설치수·별점 정렬 지원)에 등록 → 노출 확보
- **실제 매출:** B2B 커스텀 개발

#### ⑤ 에이전트 템플릿 판매
- 코드에 `AgentMarketSubmission`(마켓 제출 기능) 존재
- 예: 쇼핑몰 CS 자동응답, 블로그 SEO 작성, 영수증 정리
- **솔직한 평가:** 단독으로는 수익성이 낮음 → **①번의 부가 상품으로 묶어 파는 것이 정답**

---

### TIER 3 — 본격 사업 (3개월+)

#### ⑥ 업종 특화 패키지
①②③④를 묶어 "○○업계 전용 AI 솔루션"으로 브랜딩.

예시 — **병원 전용 AI 패키지**
```
LobeHub 설치 (원본 그대로)
  + 카톡 어댑터 (환자 문의 자동응답)
  + 의료 MCP (보험청구 코드 조회)
  + 에이전트 템플릿 (진료 요약, 차트 정리)
  + 월 유지보수
```
업종을 좁힐수록 경쟁이 줄고 단가가 오릅니다. 본체는 그대로 두고 주변 부품만 내 것으로 → ✅ 라이선스 안전.

#### ⑦ 교육·콘텐츠
- 강의: "LobeHub로 사내 AI 구축하기"
- 블로그/유튜브: 설치 가이드, MCP 제작 튜토리얼
- **진짜 목적은 강의료가 아니라 ①번 영업 유입**

#### ⑧ 정식 상업 라이선스 취득 후 제품화
`hello@lobehub.com` 계약 후 개조 판매 가능. 단, ①~⑥으로 매출을 먼저 검증한 뒤 진행하는 것이 순서.

---

## 11. 추천 조합 및 액션플랜

### 1순위 조합
```
③ 카카오톡 어댑터 개발  →  ① 사내 AI 구축 대행
      (기술 차별화)           (실제 매출)
```

| 이유 | 설명 |
| --- | --- |
| 경쟁자 없음 | 카톡 어댑터 구현체가 현재 없음 |
| 적당한 규모 | 1,288줄 수준 → 혼자 2~4주 |
| 스택 일치 | TypeScript — React/TS 경험 그대로 활용 |
| 명함 효과 | "LobeHub에 카톡 지원을 넣은 사람" |
| 라이선스 안전 | 별도 패키지 + 오픈소스 기여 |
| 매출 직결 | 카톡은 한국 기업 필수 채널 |

### 첫 4주 계획

| 주차 | 할 일 |
| --- | --- |
| 1주차 | 도커로 설치해 직접 사용해보며 감 잡기 |
| 2주차 | `packages/chat-adapter-line/` 통째로 분석 (파일 7개) |
| 3~4주차 | `chat-adapter-kakao` 구현 (카카오 비즈니스 채널 API) |
| 동시 진행 | 지인 회사 1곳 무료 설치 → 첫 포트폴리오 확보 |

---

## 12. ⚠️ 리스크 3가지

### ① 라이선스 함정
"로고만 바꿔서 판매"는 명백한 위반입니다. 코드에 `isCustomBranding` 같은 기능이 있어 **기술적으로는 가능하지만 법적으로는 불가**합니다.

### ② 본체 업데이트 속도
CHANGELOG 84,000자 — 거의 매일 업데이트됩니다. 내가 만든 어댑터가 깨질 수 있습니다.
**대응:** 본체 코드 수정을 최소화하고 내 코드는 별도 패키지로 분리.

### ③ 무료 대체재 반론
"그냥 ChatGPT 쓰면 되지 않나?"
**대응 논리:** ① 데이터가 외부로 나가지 않음 ② 카톡으로 바로 사용 ③ 부서별 전용 AI ④ 구독제보다 저렴

---

## 부록: 자주 쓰는 명령어

```bash
# 개발
pnpm install                 # 의존성 설치
bun run dev:spa              # 프론트만 실행
bun run dev                  # 풀스택 실행
bun run check                # lint + 관련 테스트
bun run check --type         # 타입 체크 (전체 레포)

# 도커
docker compose up -d         # 전체 스택 실행
docker compose pull          # 업데이트

# 개별 패키지 테스트
cd packages/database && bunx vitest run --silent='passed-only' '[file-path]'
```

> ⚠️ `bun run test`는 전체 테스트를 돌리므로 사용하지 말 것.
