# AI_Project — 소스코드 분석 프로젝트 루트 지침

## 폴더 구조

```
AI_Project/
├── CLAUDE.md                     이 파일
├── {프로젝트명}/                  분석 대상 소스코드
└── {프로젝트명}_analysis_report_YYYY-MM-DD.md    분석 보고서 (마크다운)
    {프로젝트명}_analysis_report_YYYY-MM-DD.html   분석 보고서 (HTML)
    work-log-YYYY-MM-DD-{프로젝트명}-analysis.md   작업일지
```

---

## 소스코드 분석 워크플로우

### 1. 분석 준비

- 분석 전 프로젝트 파일 수, 총 라인 수, 주요 파일 크기를 먼저 파악한다
- 전체 파일 토큰 추정값이 80,000 초과 시 배치 분할 전략을 사용자에게 먼저 안내한다
- `.env`, `node_modules`, `.git`, `dist`, `__pycache__` 등은 분석에서 제외한다

### 2. Multi-Agent 협업 분석

이 폴더의 프로젝트는 **ai_team.py 소스코드 분석 모드**를 사용한다.

```
Agent A (분석) → Agent B (검증, 코드+A분석 함께 참조) → A 재검토 → 인간 결정
```

Agent A/B 선택은 사용자가 실행 시 결정한다.  
현재 ai_team.py 경로: `C:\Users\acepe\OneDrive\바탕 화면\Claude\ai_team\ai_team.py`

### 3. 보고서 작성 기준

보고서는 **마크다운 + HTML** 두 가지 형식으로 생성한다.  
HTML은 `C:\Users\acepe\Downloads\2026-05-31_orion_report.html` 템플릿 스타일 (A4 세로)을 따른다.

---

## 보고서 작성 원칙

### ✅ 이슈 서술 방식

이슈를 서술할 때 **결과 목록만 나열하지 않는다.**  
반드시 "왜 문제인지 구체적인 시나리오"를 함께 포함한다.

**나쁜 예 (추상적 결과 나열):**
```
결과:
- 변경 영향 예측 불가
- 회귀 위험 증가
- 테스트 불가능
```

**좋은 예 (구체적 시나리오 포함):**
```
실제 발생하는 문제:
→ 버그 수정 하나 하려고 파일 열면 20,000줄을 스크롤해야 함
→ PayPal 코드를 건드렸는데 Git 동기화가 깨지는 사이드이펙트 발생
→ 테스트 작성 불가 — 개별 기능을 격리할 수 없어 단위 테스트 구성 자체가 어려움
→ 팀원 추가 불가 — "네가 이 부분 맡아"라고 할 경계 자체가 없음

결과:
- 변경 영향 예측 불가
- 회귀 위험 증가
- 테스트 불가능
```

대화 중 구체적인 설명이 나왔다면, 그 내용을 보고서에 반영한다.

---

### ✅ 강점 평가 시 주의

"보안 수준 높음" 같은 평가는 세분화한다:
- **보안 인식(Security Awareness)**: 개별 헬퍼 함수, 검증 로직의 존재
- **보안 아키텍처(Security Architecture)**: 시스템 경계, 신뢰 모델의 일관성

두 가지는 다르다. 인식이 있어도 아키텍처가 없을 수 있다.

---

### ✅ 보고서 이슈 심각도 기준

| 심각도 | 기준 |
|--------|------|
| Critical | 아키텍처 전체에 영향, 지속 불가능한 구조 |
| High | 보안 취약점, 운영 장애 유발 가능, 중요 기능 누락 |
| Medium | 유지보수 부담, 잠재적 버그, 기술 부채 |
| Low | 코드 품질, 컨벤션, 사소한 개선 |

---

## 산출물 저장 규칙

| 파일 | 위치 |
|------|------|
| 분석 보고서 (MD) | `AI_Project/{프로젝트명}_analysis_report_YYYY-MM-DD.md` |
| 분석 보고서 (HTML) | `AI_Project/{프로젝트명}_analysis_report_YYYY-MM-DD.html` |
| 작업일지 | `AI_Project/work-log-YYYY-MM-DD-{프로젝트명}-analysis.md` |
| 노션 등록 | Journal DB (date 속성 반드시 설정할 것) |

---

## 참고

- connect-ai 분석 보고서 (첫 번째 프로젝트): `connect-ai_analysis_report_2026-06-01.md`
- ai_team.py 매뉴얼: `C:\Users\acepe\OneDrive\바탕 화면\Claude\ai_team\README.md`
- Orion 보고서 HTML 템플릿: `C:\Users\acepe\Downloads\2026-05-31_orion_report.html`
