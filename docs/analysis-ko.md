# AlphaProof Nexus Results — 전수조사 분석 정리 (한국어)

> 작성일: 2026-09-22
> 대상 저장소(포크): https://github.com/bmshin94/alphaproof-nexus-results
> 원본 저장소(업스트림): https://github.com/google-deepmind/alphaproof-nexus-results

---

## 목차
1. [이 저장소는 무엇인가](#1-이-저장소는-무엇인가)
2. [전수조사 결과](#2-전수조사-결과)
3. [쉬운 설명](#3-쉬운-설명)
4. [설치 및 사용법](#4-설치-및-사용법)
5. [플러그인/스킬/MCP 판별](#5-플러그인스킬mcp-판별)
6. [API 토큰 및 라이선스](#6-api-토큰-및-라이선스)
7. [왜 유명한가](#7-왜-유명한가)
8. [로컬 에이전트 구축에 주는 도움](#8-로컬-에이전트-구축에-주는-도움)
9. [React / PHP 구현 가능성](#9-react--php-구현-가능성)
10. [수익화 아이디어](#10-수익화-아이디어)
11. [참고 링크](#11-참고-링크)

---

## 1. 이 저장소는 무엇인가

구글 딥마인드의 수학 증명 AI 시스템 **AlphaProof Nexus**가 실제로 해결한 미해결 수학 문제들의
**Lean 4 형식 증명 + 사람이 읽는 서술형 증명(PDF)** 아카이브다.

- 성격: 실행 라이브러리가 아니라 **논문 부록 / 재현성 자료(artifact)**
- 관련 논문: arXiv 2605.22763 (2026년 5월 21일 공개)
- 핵심 성과: 에르되시(Erdős) 미해결 문제 9개, OEIS 미해결 추측 44개 해결,
  15년 된 대수기하학 난제 해결, 볼록 최적화 수렴 한계 개선
- 문제당 비용: 수백 달러 수준

### 핵심 아이디어

```
LLM이 증명 생성  →  Lean 컴파일러가 기계적으로 전 단계 검증  →  실패 시 재생성 (문제당 최대 3,000 에피소드)
```

AI의 환각(hallucination) 문제를 **결정론적 검증기(Lean)** 로 원천 차단한 구조다.

---

## 2. 전수조사 결과

### 디렉터리 구조 및 파일 수

| 경로 | 파일 수 | 내용 |
|---|---|---|
| `APNOutputs/OEIS/` | 38 `.lean` | 정수 수열 백과사전(OEIS) 미해결 추측 증명 |
| `APNOutputs/ErdosProblems/` | 9 `.lean` | 에르되시 미해결 문제 증명 |
| `APNOutputs/StacksProject/` | 11 `.lean` | 대수기하학 Stacks Project 정리 |
| `APNOutputs/AICollaborator/AlgebraicGeometry/` | 7 `.lean` | 힐베르트 함수 관련 |
| `APNOutputs/AICollaborator/Graphs/` | 2 `.lean` | 그래프 이론 (Graffiti.pc 추측 등) |
| `APNOutputs/AICollaborator/QuantumOptics/` | 2 `.lean` | 양자광학 (AME 상태 존재성 등) |
| `APNOutputs/AICollaborator/AdditiveCombinatorics/` | 1 `.lean` | 가법 조합론 |
| `APNOutputs/AICollaborator/Optimization/` | 1 `.lean` | 최적화 (마지막 반복 수렴) |
| `NaturalLanguageProofs/` | 12 `.pdf` | 사람이 읽는 서술형 증명 |
| `.github/workflows/lean_action_ci.yml` | 1 | Lean CI (푸시마다 증명 자동 검증 + 문서 생성) |

### 정량 지표

- Lean 파일: **71개 / 총 44,857줄**
- `theorem` / `lemma` 선언: **2,494개**
- `@[category research open]` 태그: **57개** (원래 미해결이던 문제)
- `@[category research solved]` 태그: **25개**
- `EVOLVE-BLOCK` 마커 포함 파일: **66개 / 71개**
- 시도했던 에르되시 문제 전체 목록: `erdos_problems_attempted.txt` (352줄)

### 기술 스택

| 항목 | 값 |
|---|---|
| 언어 | Lean 4 (`leanprover/lean4:v4.27.0`) |
| 빌드 도구 | Lake (`lakefile.toml`) |
| 주요 의존성 | `google-deepmind/formal-conjectures`, `mathlib4`, `plausible`, `LeanSearchClient` |
| CI | `leanprover/lean-action@v1` + `leanprover-community/docgen-action@v1` |
| 라이선스 | 코드 Apache 2.0 / 자료 CC-BY 4.0 (OEIS 유래분은 CC BY-SA 4.0) |

### 코드에서 발견한 특징

**(1) EVOLVE-BLOCK 마커** — 71개 중 66개 파일에 존재

```lean
-- EVOLVE-BLOCK-START
... AI가 진화적으로 생성/변형한 증명 본문 ...
-- EVOLVE-BLOCK-END
```

AI가 수정 가능한 영역을 명시적으로 잠가 두고 그 안만 반복 변이시킨 흔적. 이 저장소에서 얻을 수 있는
가장 실용적인 설계 패턴이다.

**(2) 기계 생성 티가 나는 증명 스타일**

```lean
  show∃_, _∧_∧_=(id _)
  choose _ _ _ using(Set.exists_max_image _) id (BddAbove.finite (by valid)) ...
  obtain ⟨A, B, rfl⟩:=‹∃l,_›
```

공백 없는 압축된 tactic, 익명 참조(`‹...›`) 남용 등 사람이 쓰지 않는 형태. 가독성은 낮지만
컴파일러 검증을 통과했으므로 수학적 정확성은 보장된다.

**(3) 정직한 실패 공개**

`erdos_problems_attempted.txt`에 시도한 353개 문제 전체를 공개해, 성공률의 분모를 숨기지 않았다.

**(4) 출처 표기 철저**

각 파일 헤더에 원 문제 URL, 참고 논문(arXiv/Phys. Rev.), formal-conjectures 이슈 번호가 명시돼 있다.

---

## 3. 쉬운 설명

이 저장소를 비유하면 **"천재 AI가 푼 수학 난제 답안지 + 자동 채점기 세트"** 다.

| 구성요소 | 비유 |
|---|---|
| `erdos_problems_attempted.txt` | 시험 범위 (도전한 353문제) |
| `APNOutputs/*.lean` | 답안지 (실제로 맞힌 71개 풀이) |
| `lake build` | 채점기 (돌리면 답안 검증) |
| `NaturalLanguageProofs/*.pdf` | 사람 말로 쓴 해설지 |

### 동작 방식

```
사람: "이 문제 풀어봐"
  → AI: Lean 코드로 증명 작성
  → Lean 컴파일러: 논리 비약 발견 시 탈락
  → AI: 에러 피드백 받아 수정 후 재제출
  → (문제당 최대 3,000회 반복)
  → 통과 시 이 저장소에 저장
```

**AI가 자기 답을 스스로 채점하지 않는다**는 것이 핵심이다.

### Lean이란

수학을 프로그래밍 언어처럼 기술하는 정리 증명기.

| 일반 수학 | Lean |
|---|---|
| "a(n)은 중심 이항계수의 세제곱" | `def a (n : ℕ) : ℕ := (Nat.choose (2*n) n) ^ 3` |
| "따라서 참이다 ∎" | `theorem target_theorem_0 : ... := by ...` |

일반 프로그램은 컴파일 성공 = 실행 가능이지만, **Lean은 컴파일 성공 = 수학적으로 참임이 증명됨**이다.

---

## 4. 설치 및 사용법

### 설치

```bash
# 1) Lean 4 툴체인 설치 (macOS / Linux)
curl https://elan.lean-lang.org/elan-init.sh -sSf | sh
source ~/.profile

# 2) 클론 및 빌드
git clone https://github.com/google-deepmind/alphaproof-nexus-results
cd alphaproof-nexus-results
lake exe cache get      # Mathlib 사전 빌드 캐시 (필수, 생략 시 수 시간 소요)
lake build              # 전체 증명 검증

# 3) 코어가 많은 머신은 스레드 제한 (파일 핸들 고갈 방지)
LEAN_NUM_THREADS=64 lake build
```

### 요구 사항 / 주의점

- 디스크 10GB 이상, RAM 16GB 이상 권장 (Mathlib 의존성이 수 GB)
- 전체 빌드는 수십 분 ~ 수 시간 소요
- 단순 열람 목적이면 **빌드 불필요** — VS Code + Lean 4 확장으로 읽기만 해도 충분

### 목적별 사용법

| 목적 | 방법 |
|---|---|
| 구경만 | GitHub 웹에서 파일 열람 (설치 불필요) |
| 서술형 증명 읽기 | `NaturalLanguageProofs/*.pdf` |
| 직접 검증 | `lake exe cache get && lake build` |
| 학습/연구 | VS Code + Lean 4 확장, 커서 이동하며 증명 상태 확인 |

---

## 5. 플러그인/스킬/MCP 판별

| 구분 | 해당 여부 | 근거 |
|---|---|---|
| 플러그인 | 아니오 | 호스트 앱/확장 포인트 없음 |
| Claude Skill | 아니오 | `SKILL.md`, 프론트매터 없음 |
| MCP 서버 | 아니오 | 서버 코드, `mcp.json`, 도구 스키마 전무 |
| **연구 결과 아카이브(데이터셋)** | **예** | `.lean` 증명 + `.pdf` + CI 설정만 존재 |

`lakefile.toml`에 `lean_lib`만 있고 `lean_exe`(실행 엔트리포인트)가 없다는 점이 결정적 근거다.

> AlphaProof Nexus 시스템 자체는 비공개이며, 이 저장소는 그 **출력물만** 공개한 것이다.
> 따라서 이것을 설치해 새로운 문제를 풀게 할 수는 없다.

---

## 6. API 토큰 및 라이선스

### API 토큰: 불필요

- 네트워크 호출 코드, API 키 환경변수 없음
- `lake exe cache get`이 쓰는 것은 Mathlib 공개 캐시 다운로드로 인증 불필요
- 비용 0원

### 라이선스

| 대상 | 라이선스 | 의무 |
|---|---|---|
| 소프트웨어 | Apache 2.0 | 출처 표기 |
| 자료 | CC-BY 4.0 | 출처 표기 |
| OEIS 유래 자료 | CC BY-SA 4.0 | 출처 표기 + **동일조건변경허락** |
| Erdős Problems 유래 | erdosproblems.com 조건 확인 필요 | - |

상업적 이용은 가능하나, **OEIS 파생물은 share-alike 의무**가 있으므로 수익화 시 반드시 확인할 것.

---

## 7. 왜 유명한가

### 실제 수치 (2026-09 조회)

| 저장소 | Stars | Forks |
|---|---|---|
| google-deepmind/alphaproof-nexus-results | 299 | 29 |

초대형 인기 저장소는 아니며, **화제성 기반**의 주목이다.

### 주목받는 이유

1. **"AI가 인간 미해결 난제를 실제로 풀었다"는 상징성** — 벤치마크 점수가 아닌 새로운 수학 결과
2. **제3자 검증 가능성** — `lake build`만 돌리면 누구나 검증 가능, 반박 불가
3. **AGI 논쟁의 중심** — 딥마인드가 "아직 AGI는 아니다"라고 선을 그으며 오히려 화제화
4. **비용 충격** — 문제당 수백 달러 vs 인간 수학자 수개월
5. **정직한 실패 공개** — 353개 시도 목록 전체 공개로 연구 신뢰 확보

> 요약: 별 개수는 "실용 라이브러리"가 아니라 "뉴스 가치"에서 나온 것이다.

---

## 8. 로컬 에이전트 구축에 주는 도움

### 코드 재사용: 불가 / 설계 철학: 매우 유용

이 저장소에서 추출한 핵심 패턴은 **Verifier-in-the-Loop**다.

```
1. Proposer (LLM)        → 후보 생성
2. Verifier (컴파일러/테스트) → 결정론적 판정   ← LLM이 아니어야 함
3. Feedback Loop         → 에러 메시지를 컨텍스트로 재투입
4. Evolution             → 성공분만 보존, 지정 블록만 변이 (EVOLVE-BLOCK)
5. Budget Control        → 최대 시도 횟수 / 비용 상한
```

### 일반 개발 에이전트로의 치환

| 이 저장소 | 일반 에이전트 |
|---|---|
| Lean 컴파일러 | TypeScript 컴파일러 / ESLint / Jest |
| 증명 성공 | 테스트 전부 통과 |
| `EVOLVE-BLOCK` 마커 | "AI가 수정 가능한 영역"을 주석으로 명시 |
| 3,000 episodes 예산 | `max_retries` + 비용 상한 |
| `erdos_problems_attempted.txt` | 시도/실패 로그 DB |

### 가져갈 인사이트 3가지

1. **LLM이 LLM을 채점하면 신뢰도가 무너진다** → 결정론적 검증기를 반드시 붙일 것
2. **EVOLVE-BLOCK 방식** → AI가 파일 전체를 헤집지 않고 지정 블록만 수정 → 안정성 급상승
3. **실패 로그도 자산** → 다음 시도의 컨텍스트로 재활용

---

## 9. React / PHP 구현 가능성

### 불가능한 것

- Lean 증명 커널을 React/PHP로 재구현 (의존 타입 이론 구현체, 현실성 없음)
- 브라우저에서 Mathlib 전체 구동 (수 GB 규모)

### 가능한 것

**(1) React — "AI 수학 증명 갤러리" 웹앱**

- 스택: Next.js + Tailwind + KaTeX(수식) + Shiki(Lean 하이라이팅)
- 기능: 71개 증명 카드 브라우징, 태그 필터, Lean ↔ PDF 대조 뷰,
  AI 요약 버튼(Claude API), 난이도/줄 수/tactic 통계 대시보드
- 데이터가 정적이므로 SSG로 빌드 시 서버 비용 거의 0

**(2) PHP(Laravel) — 검증 루프 SaaS 백엔드**

```php
Route::post('/verify-job', function (Request $r) {
    $job = VerifyJob::create(['code' => $r->code, 'lang' => 'lean']);
    dispatch(new RunVerifier($job));   // Docker 컨테이너에서 실제 검증 수행
    return ['job_id' => $job->id];
});
```

- Laravel Queue + Horizon으로 작업 큐 관리
- 무거운 검증은 Docker 컨테이너에 위임, PHP는 오케스트레이션 담당

**(3) 권장 조합**

```
[React 프론트] <-> [PHP/Laravel API] <-> [Docker: Lean/Node/Python 검증기]
                          |
                   [Claude API: 생성 + 설명]
```

기존 React/PHP 역량을 그대로 쓰면서 이 저장소의 "검증 루프 철학"만 얹는 구조.

---

## 10. 수익화 아이디어

> 전제: 저장소 자체는 CC-BY로 무료 배포되므로 "그대로 판매"는 의미가 없다.
> **데이터를 가공**하거나 **방법론을 제품화**해야 수익이 된다.

### 아이디어 1. "AI 코드 검증 루프" SaaS (최우선 추천)

| 항목 | 내용 |
|---|---|
| 문제 | AI 생성 코드가 그럴듯하지만 틀려 리뷰 비용이 폭증 |
| 해법 | 생성 → 결정론적 검증(타입체크+테스트+린트) → 피드백 → 재생성 루프 자동화 |
| 차별점 | 경쟁사는 "AI가 리뷰", 본 제품은 "컴파일러가 판정" |
| 가격 | Free 50회/월 / Pro $29 / Team $99·석 / Enterprise 온프레미스 |
| 스택 | React + Laravel + Docker 샌드박스 + Claude API |
| 난이도 | 중 (MVP 2~3주) |

### 아이디어 2. EVOLVE-BLOCK 오픈소스 → 컨설팅 전환

```javascript
// @evolve-start   ← AI 수정 허용 시작
function optimizeQuery(sql) { ... }
// @evolve-end     ← 이 밖은 수정 금지
```

1. npm 패키지 무료 공개 → 인지도 확보
2. VS Code 확장 무료 배포 → 사용자 확보
3. Pro(진화 히스토리/롤백/팀 정책) 유료화
4. 기업 도입 컨설팅으로 고단가 전환

### 아이디어 3. 교육 콘텐츠 / 강의 (현금화 속도 1위)

| 상품 | 가격대 |
|---|---|
| 유튜브/블로그 시리즈 | 무료 (유입 채널) |
| 온라인 강의 "검증 가능한 AI 에이전트 만들기" | 5~15만원 |
| 유료 뉴스레터 | 월 1만원 |
| 기업 세미나 | 회당 200~500만원 |

CC-BY 라이선스라 출처 표기만 하면 강의 자료로 자유롭게 사용 가능.

### 아이디어 4. AI 생성 코드 데이터셋 가공

- 원재료: Lean 44,857줄, 정리 2,494개, 실패 목록 353개
- 가공: (문제 → 실패 시도 → 에러 → 성공 코드) 구조화
- 주의: OEIS 유래분은 CC BY-SA → 파생물 공개 의무.
  **데이터는 무료 공개하고 "가공 도구"를 판매**하는 우회 전략 권장

### 아이디어 5. 증명 갤러리 미디어 사이트

React SSG + AI 해설 + 수식 렌더링. 애드센스/스폰서/프리미엄 해설 구독.
서버 비용 최소, 다만 트래픽 확보가 관건.

### 아이디어 6. 형식 검증 B2B 컨설팅 (수익성 최고)

- 타겟: 금융, 의료, 항공, 블록체인 스마트컨트랙트
- 단가: 프로젝트당 3,000만 ~ 2억원
- 난이도 최상, 아이디어 3으로 권위를 쌓은 뒤 인바운드 유도

### 비교표

| 아이디어 | 수익성 | 난이도 | 실현 속도 | 종합 추천 |
|---|---|---|---|---|
| 1. 검증 SaaS | 높음 | 중 | 중 | ★★★★★ |
| 2. EVOLVE-BLOCK OSS | 중상 | 낮음 | 중 | ★★★★ |
| 3. 교육 콘텐츠 | 중 | 낮음 | 빠름 | ★★★★ |
| 4. 데이터셋 | 중 | 중 | 중 | ★★★ |
| 5. 갤러리 사이트 | 낮음 | 낮음 | 빠름 | ★★★ |
| 6. B2B 컨설팅 | 최상 | 최상 | 느림 | ★★★★★ |

### 권장 로드맵

```
1~2개월  : 교육 콘텐츠     → 현금흐름 + 인지도
3~4개월  : 오픈소스 공개   → 개발자 신뢰
5~8개월  : SaaS 출시       → 구독 수익 본진
9개월~   : B2B 컨설팅      → 고단가 수확
```

**핵심 원칙: 저장소를 파는 것이 아니라 "검증 가능한 AI"라는 개념을 판다.**

---

## 11. 참고 링크

### 저장소

- 포크(작업 대상): https://github.com/bmshin94/alphaproof-nexus-results
- 원본(업스트림): https://github.com/google-deepmind/alphaproof-nexus-results
- 관련: https://github.com/google-deepmind/formal-conjectures
- OEIS 전체 문제셋: https://github.com/google-deepmind/formal-conjectures/tree/auto_oeis/FormalConjectures/OEIS/Auto
- OEIS 정리명 매핑: https://github.com/google-deepmind/formal-conjectures/blob/auto_oeis/FormalConjectures/OEIS/Auto/THEOREM_MAPPING.txt
- Erdős 형식화: https://github.com/google-deepmind/formal-conjectures/tree/main/FormalConjectures/ErdosProblems

### 도구 / 출처

- Lean 4 설치 가이드: https://lean-lang.org/lean4/doc/quickstart.html
- OEIS: https://oeis.org
- Erdős Problems: https://www.erdosproblems.com/
- Open Quantum Problems: https://oqp.iqoqi.oeaw.ac.at/

### 기사 / 해설

- https://winbuzzer.com/2026/05/26/google-deepmind-says-alphaproof-nexus-is-still-not-agi-xcxwbn/
- https://www.kucoin.com/news/flash/google-deepmind-s-alphaproof-nexus-solves-9-erd-s-problems-and-44-oeis-conjectures
- https://dev.to/monuminu/how-deepmind-alphaproof-nexus-cracks-56-year-old-math-agentic-llm-loops-and-lean-formal-45ei
- https://aichats.substack.com/p/deepminds-alphaproof-nexus
- https://medium.com/@beatwad/alphaproof-nexus-how-deepminds-ai-is-cracking-mathematical-problems-that-have-stumped-humans-for-eccdd2433b84

---

*본 문서는 저장소 전수조사 결과를 바탕으로 작성된 한국어 분석 정리본입니다.*
