# Changelog

본 하네스(Team Big Five)의 변경 이력. 형식은 [Keep a Changelog](https://keepachangelog.com/) 약식.

## [2.0.0] — 2026-05-29

에이전트 팀으로 하네스를 심층 분석(4개 렌즈: 개발·일상/창작·하네스 메커니즘·팀과학 충실도)한 뒤, 발견된 개선점을 반영한 대규모 개정. 독립 평가 결과 **18 PASS / 1 PARTIAL / 0 FAIL** (PARTIAL 및 HIGH 플래그는 검증 후 오탐으로 확인 — `TaskGet`은 실재 도구).

### Fixed — 정확성 (이게 없으면 하네스가 실행조차 안 됨)
- **오케스트레이터 Phase 2가 존재하지 않는 도구 시그니처를 호출하던 치명 결함 수정.**
  - `TeamCreate(members:[...])` → 실제 시그니처(`team_name`/`agent_type`/`description`)만 사용. 멤버는 **Agent 도구**로 `team_name`+`name`+`subagent_type` 주어 개별 스폰.
  - `TaskCreate(assignee/depends_on)` → 실제로는 없는 인자. 배정은 `TaskUpdate(owner)`, 의존은 `TaskUpdate(addBlockedBy)`.
  - 종료: `SendMessage({type:"shutdown_request"})` → 전원 `shutdown_response` 수신 → `TeamDelete` (활성 멤버 잔존 시 실패) 순서 명시.
- **구체적 조율 루프** 명문화 — `TaskList` → 최저 ID unblocked claim → `TaskUpdate` 완료 → 재조회.
- **idle ≠ 정체 구분** — idle은 매 턴 정상. 정체 = task가 N주기 무변화. 건강한 유휴를 실패로 오인해 중복 백업하던 위험 제거.
- **데드락 회피** — 폐쇄 루프 ACK는 1회 재전송 후 에스컬레이션 상한, peer-to-peer 직접 통신·능동 SendMessage 깨우기.
- **에이전트 `tools:` frontmatter 추가** (3개 전부). `performance-monitor`를 general-purpose가 아닌 커스텀 타입으로 스폰 + Bash 부여. `team-lead`은 "메인 컨텍스트가 채택, 별도 스폰 안 함" 명시.
- **모델 라우팅** — 전원 opus 폐지. lead=opus, contributor=sonnet 기본(고난도만 opus), monitor=sonnet. 비용·지연 절감.
- **폴백 경로** — 팀 도구 미가용 세션에서 Agent 팬아웃 + 리더 중재 핸드오프로 강등.

### Added — 과제 유형 다형성 (개발 외 일상·창작 지원)
- **과제 유형 적응 표** (오케스트레이터 + SMM) — 코드/리서치/문서/창작/마케팅별 경계면 계약·완료기준·모니터링 대상 매핑.
- **SMM 신규 섹션** — §6 스타일/보이스(산문 계약), §8 사실 원장/Canon(주장→출처·플롯 사실), §7 파일 소유권(코드), §9 리스크/대응.
- **모니터링 체크리스트의 과제별 적응** — "shape 대조" 일변도에서 보이스 드리프트·canon 모순·무근거 주장·용어 일관까지.
- **완료 기준 = 수용 rubric** (체크 가능한 예/아니오 3~6개) — 주관 산출물도 게이트 가능. Phase 4가 rubric으로 종합 게이팅.
- **하드 vs 소프트 제약** — canon·사실·계약은 [hard](차단), 보이스·플롯 탐색은 [soft](권고)로 창의 보존.
- **창작/리서치 워크드 예시** 추가.

### Added — 개발 엄밀성
- **실행 게이트** (mutual-monitoring Part 0) — 코드 과제는 텍스트 대조 전 빌드·타입체크·테스트·린트를 실제 실행, exit code 기록. "markdown 정합 ≠ 컴파일"의 빈틈 제거.
- **baseline 회귀 가드** — Phase 1에서 기존 통과 상태 캡처, 통과하던 테스트가 깨지면 차단.
- **코드 계약을 스키마 파일로** (`_workspace/contracts/`) + **버전 묶음 신뢰 무효화** — 계약 버전이 오르면 하위 "검증됨" 자동 무효 → 재검증.

### Added — 팀 과학 충실도 (IMOI/에피소드 사이클 도입)
- **킥오프 브리핑** (Phase 2) — 각 멤버가 자기 슬라이스를 read-back ACK → SMM이 *작성*에서 *공유*로 수렴.
- **디브리핑/AAR** (Phase 5) — `debrief.md` 4문항 사후 검토. 하네스 진화의 학습 신호.
- **팀 스코어카드** — monitor가 `scorecard.md`에 DRIFT·백업·적응·라운드·rubric 충족을 계량. "측정 없이 진화 없다."
- **보정된 신뢰** — 이진 "재작업 금지"에서 경계면별 신뢰 등급(verify-once/on-change/each-round)으로. 결과 비용에 비례한 검증.
- **심리적 안전 채널** — FLAG-UNCERTAIN(불확실성)·DISSENT(이견). 추측으로 덮기(confabulation) 방지.
- **사전 적응** — SMM §9 위험 가정을 monitor가 능동 감시, 붕괴 조짐 시 사전 경고.
- **상호의존성 triage** (Phase 0.5) — 독립/낮음/높음 등급에 따라 팀 미구성·경량·풀 Big Five로 강도 조절. 과적용 방지.
- **결정권(decision rights)** — SMM 작업 맵에 영역별 권한 명시, 리더 병목 완화.

### Changed
- 데이터 흐름·에러 핸들링 표를 위 메커니즘에 맞게 갱신(빌드 실패·머지 충돌·팀 도구 미가용 행 추가).

## [1.0.0] — 2026-05-26
- 초기 구성: 에이전트 3(team-lead/contributor/performance-monitor) + 스킬 4(orchestrator/SMM/closed-loop/mutual-monitoring) + 이론 해설 문서(TEAM-SCIENCE.md).
- 이후 README 추가, 설치형 Claude Code 플러그인으로 변환(`plugin` 브랜치).
