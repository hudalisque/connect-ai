## 작업 메모 — 2026-06-01 connect-ai 소스코드 분석

### 작업 내용
- connect-ai VS Code 확장 프로젝트 소스코드 분석 보고서 작성
- ai_team 분석 모드 설계 및 구현 (ai_team.py 수정)
- Multi-Agent 협업 워크플로우 실행: Claude(A) 초기 분석 → GPT-4o(B) 검증 → Claude(A) 재검토 → 인간 결정
- 최종 A-2안 통합 보고서 생성

### 관련 파일
- `AI_Project/connect-ai_analysis_report_2026-06-01.md` — 최종 분석 보고서 (A-2안)
- `Claude/ai_team/ai_team.py` — 소스코드 분석 모드 추가, Claude 백엔드 manual/api 전환 구조 추가
- `Claude/ai_team/.env` — CLAUDE_BACKEND=manual 추가

### 결정 사항
- **A-2안 채택**: Claude A안 + GPT B안 검증 의견 통합
- **Claude API 호출 구조**: CLAUDE_BACKEND 환경변수로 manual/api 전환 가능하게 설계 (현재 manual — 구독 활용)
- **모델 변경**: ai_team.py의 Claude 모델 Haiku → Sonnet 4.6으로 업그레이드

### 문제 / 리스크
- extension.ts 20,391줄 전체를 단일 API 호출로 분석 불가 (토큰 한도 초과) → 스냅샷 기반 분석으로 진행
- GPT는 전체 코드 미열람 상태로 검증 → "스냅샷 기준 peer review"임을 명시

### 다음 액션
- connect-ai 개발자에게 분석 보고서 공유 (필요시)
- 4.2~4.5 High 이슈 중 HTTP 서버 바인딩/인증 실제 코드 확인 (extension.ts 4825 포트 섹션 직접 검토)
- Secret 관리 방식 확인 (VS Code SecretStorage 사용 여부)

---

## Multi-Agent 협업 회의록

**세션 정보**
- 분석 대상: connect-ai (VS Code Extension)
- Agent A (분석): Claude Sonnet 4.6
- Agent B (검증): GPT-4o
- 인간 의사결정자: 원종석
- 워크플로우: A 분석 → B 검증(코드+A분석 함께 참조) → A 재검토 → 인간 최종 결정

---

### Round 1 — Agent A 초기 분석 (A안)

**분석 범위**: ARCHITECTURE.md, package.json, agents.ts, paths.ts, extension.ts 구조(함수 목록 + 핵심 섹션 샘플링)

**주요 발견:**

강점:
- 보안 헬퍼 함수들이 초기부터 구현됨 (경로 트래버설 방지, Git URL 검증 등)
- agents.ts, paths.ts로 의도적 모듈 분리 진행 중
- 크로스플랫폼 대응 수준이 높음

이슈:
- [Critical] extension.ts 20,391줄 단일 파일 (ARCHITECTURE.md는 여전히 "2600+ lines" 기재)
- [High] runCommandCaptured(shell:true) — AI 생성 커맨드 쉘 실행
- [Medium] 버전 불일치: _CONNECT_AI_VERSION='2.89.156' vs package.json '2.89.157'
- [Medium] 글로벌 뮤터블 상태 산재
- [Medium] 테스트 코드 없음
- [Low] 자동 사이클 15분마다 LLM 호출 (저사양 PC cold start 실패)

리팩토링 제안: git-sync.ts, llm-client.ts, tool-runner.ts, http-bridge.ts, integrations/, panels/ 분리

---

### Round 2 — Agent B 검증 의견 (B안)

**GPT-4o가 동의한 항목:**
- extension.ts monolith — Critical 맞음 (변경 영향 예측 불가, 책임 경계 소멸)
- shell:true — High, 오히려 Critical 가능성 제기
- 버전 불일치 — 운영 비용이 크다고 동의
- 테스트 없음 — 리팩토링 시 안전망 없다고 동의

**GPT-4o가 이견을 제시한 항목:**

1. "보안 수준 높음" → 과도하게 낙관적
   - GPT 주장: "보안 인식"과 "보안 아키텍처"는 다름
   - HTTP 서버·쉘 실행·웹뷰·Python 실행이 경계 없이 공존 = 보안 아키텍처 부재

2. 자동 사이클 Low → Medium-High 상향 권고
   - 단순 성능 문제가 아닌 CPU·메모리·토큰·네트워크 사용자 인지 없이 소비
   - activate()에 묶여 격리 불가

3. "모듈 분리 진행 중" → 절반만 사실
   - agents.ts/paths.ts 존재는 인정하나, 의존성 방향이 바뀌었는지 증거 없음
   - 파일이 생겨도 extension.ts → 모든 것 구조면 여전히 모놀리스

**GPT-4o가 A에서 놓쳤다고 지적한 5가지:**

1. activate() Lifecycle 과부하 — "가장 중요한 누락"
   - VS Code 시작 시 LLM 자동화·텔레그램·매출 모니터링·브리핑이 동시 시동
   - 레이지 활성화, 명시적 opt-in 필요

2. HTTP 서버(포트 4825) 보안 미분석
   - 바인딩: 127.0.0.1 vs 0.0.0.0?
   - 인증: API 키/토큰/서명 요청?
   - 스키마 검증 여부?

3. 웹뷰 ↔ Extension Host 경계 미분석
   - 메시지 라우팅 방식, 화이트리스트, 도구 실행 트리거 가능성

4. Secret 관리 방식 미확인
   - Telegram·Calendar·PayPal 크레덴셜이 SecretStorage인지 평문인지

5. 도메인 경계(Brain/Chat/Automation/Business/Integration) 미분류

**GPT-4o의 리팩토링 우선순위 (A와 다른 점):**
- Phase 1: SchedulerService 분리 (런타임 오케스트레이션 먼저)
- Phase 2: 핵심 서비스 경계 (BrainRepository, GitSyncService, LLMClient, CommandRunner)
- Phase 3: integrations/ 격리
- Phase 4: 신뢰 경계 강화
- Phase 5: 테스트 추가

---

### Round 3 — Agent A 재검토 (B 의견 수신 후)

**동의하여 A-2안에 반영한 항목:**

| B 의견 | A의 대응 |
|--------|---------|
| "보안 인식 ≠ 보안 아키텍처" | 강점 평가 수정. "인식은 있으나 아키텍처 부재"로 변경 |
| 자동 사이클 Low→Medium | Medium으로 상향 반영 |
| activate() 과부하 | High 이슈로 신규 추가 |
| HTTP 서버 보안 | High 이슈로 신규 추가, 검증 항목 명시 |
| 웹뷰 경계 | High 이슈로 신규 추가 |
| Secret 관리 | Medium 이슈로 신규 추가 |
| 리팩토링 순서 | GPT Phase 1 채택 (런타임 격리 우선) |

**부분 반론 (A-2안에서 유지):**
- "모듈 분리는 절반만 사실" — GPT 기준(의존성 방향)은 맞지만 agents.ts/paths.ts 분리는 실질적 개선
- "방향은 맞으나 미완성"이 더 정확한 평가. GPT가 과소평가함.

---

### 인간 의사결정자 최종 결정

**채택: A-2안**

사유:
- B의 누락 항목(activate 과부하, HTTP 보안, 웹뷰 경계, Secret 관리)이 타당하여 반영
- severity 수정(보안 평가, 자동 사이클)도 동의
- 리팩토링 순서는 GPT 제안 채택 (런타임 격리 → 파일 분리)

**이슈 최종 현황:**

| 심각도 | 이슈 | 출처 |
|--------|------|------|
| Critical | extension.ts 20,391줄 monolith | A안 |
| High | activate() Lifecycle 과부하 | B 추가 |
| High | runCommandCaptured(shell:true) | A안 |
| High | HTTP 서버(4825) 보안 미검증 | B 추가 |
| High | 웹뷰 ↔ Extension Host 경계 | B 추가 |
| Medium | Secret 관리 방식 미확인 | B 추가 |
| Medium | 버전 불일치 | A안 |
| Medium | 글로벌 뮤터블 상태 | A안 |
| Medium | 테스트 없음 | A안 |
| Medium | 자동 사이클 (A: Low → B: Medium) | 상향 조정 |
