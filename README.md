# Team Big Five Harness

Team Science의 **"팀워크 빅파이브"**(Salas, Sims & Burke, 2005)를 Claude Code의 에이전트 팀에 적용한 하네스. 더 똑똑한 에이전트가 아니라 **더 나은 팀워크**로 협업 과제의 결과 품질을 높인다.

> 한 줄 가설: 에이전트 팀의 성과 병목은 개별 지능이 아니라 *조율의 질*이다. 조율 메커니즘을 명시적 프로토콜로 강제하면 같은 모델로도 결과가 좋아진다.

> **v2 (2026-05-29):** 코드 전용 가정을 벗고 **개발·일상·창작 전 영역**으로 일반화하고, 실행 게이트·킥오프/디브리핑·보정된 신뢰·심리적 안전 채널을 추가했다. 무엇보다 v1의 오케스트레이터가 *존재하지 않는 도구 시그니처*(`TeamCreate(members:[...])` 등)를 호출하던 치명 결함을 고쳐 **실제로 실행되게** 만들었다. 전체 변경은 [`CHANGELOG.md`](CHANGELOG.md).

---

## 설치 (Claude Code 플러그인)

이 리포지토리는 설치형 Claude Code 플러그인이자 마켓플레이스다. 한 번 설치하면 어느 프로젝트에서든 트리거 표현으로 팀이 작동한다.

**1) 마켓플레이스 추가 + 설치:**
```
/plugin marketplace add tobyilee/team-bigfive
/plugin install team-bigfive@team-bigfive-marketplace
```

**2) 또는 대화형 UI:**
```
/plugin marketplace add tobyilee/team-bigfive
/plugin            # 메뉴에서 team-bigfive 선택 → Install
```

로컬 경로(클론한 디렉터리)로도 추가 가능:
```
/plugin marketplace add /path/to/team-bigfive
```

설치 후 제공되는 것: 에이전트 3(`team-lead`/`contributor`/`performance-monitor`) + 스킬 4(`team-bigfive-orchestrator` 외 3개 조율 메커니즘). 확인은 `/plugin` 메뉴 또는 스킬 목록.

> 플러그인 컴포넌트는 리포 루트의 `agents/`·`skills/`에 있고, 패키징은 `.claude-plugin/`(plugin.json·marketplace.json)에 있다.

---

## 빠른 시작

협업이 필요한 과제를 줄 때 트리거 표현을 쓰면 오케스트레이터가 작동한다:

```
"이 API 설계하고 프론트 연동까지 팀으로 해줘"
"에이전트 팀으로 이 리서치 종합해줘"
"이 단편 세계관·플롯·1~3장 팀 빅파이브로 써줘"
```

트리거 키워드: `팀으로`, `에이전트 팀`, `팀 빅파이브`, `고성과 팀`, `여러 에이전트로`, `협업으로 처리`
→ `team-bigfive-orchestrator` 스킬이 실행된다.

**언제 쓰나:** 산출물이 서로 맞물리는 과제(경계면이 있는 과제 — API↔프론트, 세계관↔플롯, 분석↔검증).
**언제 안 쓰나:** 단일 작업, 단순 질문, 경계면 없는 독립 병렬 작업(이건 일반 서브 에이전트 팬아웃이 낫다). 오케스트레이터의 **Phase 0.5 triage**가 이 판단을 자동으로 한다.

---

## 원리 — Team Big Five

팀 성과는 개인 역량의 합이 아니다. **팀워크 행동의 질**에서 나온다. Salas 2005는 이를 5개 행동(Big Five) + 그것을 가능하게 하는 3개 조율 메커니즘으로 정리했다.

```
조율 메커니즘 (토대)          Big Five 행동              결과
  · 공유 정신 모델     ──→    · 팀 리더십          ──→
  · 폐쇄 루프 통신            · 상호 성과 모니터링        팀 성과 ↑
  · 상호 신뢰                · 백업 행동                (과제 상호의존성
                            · 적응성                    높을수록 효과 큼)
                            · 팀 지향성
```

메커니즘이 없으면 행동도 안 나온다 — 신뢰 없으면 백업 없고, 공유 정신 모델 없으면 적응이 엇나간다.

### 이론 → 하네스 매핑

| Team Big Five 요소 | 인간 팀 | 이 하네스 |
|---|---|---|
| **Team Leadership** | 팀장 | `team-lead` (오케스트레이터 메인) — 목표·배분·종합·적응 |
| **Mutual Performance Monitoring** | 동료 작업 관찰 | `performance-monitor` — 경계면 교차 검증 + (코드) 실행 게이트 |
| **Backup Behavior** | 바쁜 동료 거들기 | 유휴 팀원 작업 claim / 리더 재할당 (idle ≠ 정체) |
| **Adaptability** | 전략 수정 | 적응 체크포인트 + 위험 가정 사전 감시 |
| **Team Orientation** | 팀 우선 태도 | 에이전트 정의 내재 원칙 + 능동 정보 공유 |
| *Shared Mental Models* | 머릿속 공유 이해 | `_workspace/SMM.md` 단일 진실 원천 (과제 유형 적응) |
| *Closed-Loop Communication* | 복창 | `SendMessage` + ACK 재진술 + FLAG-UNCERTAIN/DISSENT |
| *Mutual Trust* | 동료 신뢰 | **보정된 신뢰** — 경계면별 verify 등급, 계약 버전 묶음 |

**핵심 설계 결정:**
1. **공유 정신 모델을 파일로 외재화** (`_workspace/SMM.md`) — 에이전트 컨텍스트는 휘발하므로 단일 진실 원천 파일이 필요. 모든 팀원이 작업 단위마다 읽고 갱신하며, 킥오프 read-back ACK로 *공유*를 보장.
2. **모니터링을 전담 에이전트로 분리** — 에이전트는 자기 작업에 매몰돼 경계면을 놓침. `performance-monitor`가 교차 검증(코드면 실제 빌드/테스트 실행)을 보장.
3. **과제 유형 다형성** — 같은 메커니즘, 다른 계약. 코드는 스키마·빌드 게이트, 산문은 보이스/canon/사실 원장·rubric 게이트.

자세한 이론 해설은 **[`docs/TEAM-SCIENCE.md`](docs/TEAM-SCIENCE.md)** 참조.

---

## 과제 유형 적응

| 과제 유형 | 경계면 계약 | 완료 기준 | 모니터링 대상 |
|----------|-----------|----------|-------------|
| 코드/설계 | API/타입 스키마, 파일 소유권 | 빌드·테스트·린트 통과 | 실행 게이트 + shape 대조 |
| 리서치/분석 | 사실 원장(주장→출처), 용어 | 모든 주장 출처 有, rubric | 무근거 주장·인용 정확성 |
| 문서/보고서 | 섹션 개요, 용어, 독자 | rubric 수용 기준 | 용어 일관·중복/공백 |
| 창작(소설 등) | 보이스 시트, canon, 타임라인 | rubric 수용 기준 | 보이스 드리프트·canon 모순 |
| 마케팅/카피 | 포지셔닝, 브랜드 보이스 | rubric 수용 기준 | 메시지·톤·CTA 일관 |

---

## 구조

```
.claude-plugin/                      # 플러그인 패키징
│   ├── plugin.json                  # 플러그인 매니페스트 (v2.0.0)
│   └── marketplace.json             # 마켓플레이스 정의
agents/                              # 누가 (역할 정의, tools·model frontmatter 포함)
│   ├── team-lead.md                 # 리더십 + 적응성 (메인 컨텍스트가 채택, opus)
│   ├── contributor.md               # 팀 지향 + 백업 + 신뢰 (N명 스폰, sonnet 기본)
│   └── performance-monitor.md       # 상호 모니터링 + 실행 게이트 + 스코어카드 (sonnet)
skills/                              # 어떻게 (절차)
│   ├── team-bigfive-orchestrator/   # 8개 메커니즘을 워크플로우로 조율
│   │   ├── SKILL.md
│   │   └── references/bigfive-theory.md
│   ├── shared-mental-model/         # 조율 메커니즘 1 (과제 유형 적응 SMM)
│   ├── closed-loop-comms/           # 조율 메커니즘 2 (+ 심리적 안전 채널)
│   └── mutual-monitoring/           # Big Five 2·3 (실행 게이트 + 스코어카드)
docs/TEAM-SCIENCE.md                 # 이론 상세 해설
CHANGELOG.md                         # 버전별 변경 이력
CLAUDE.md                            # 하네스 포인터 + 변경 이력
```

에이전트 = "누가", 스킬 = "어떻게". 상호 신뢰·팀 지향성은 절차가 아니라 *태도*라서 에이전트 정의의 원칙으로 내재화했다.

---

## 동작 흐름

```
Phase 0    컨텍스트 확인     초기/새/부분 재실행 판별 (_workspace/ 존재 여부)
Phase 0.5  상호의존성 triage 독립→팬아웃 권고 / 낮음→경량 / 높음→풀 Big Five
Phase 1    준비 + SMM 초기화 과제 유형 판별, 목표·rubric·계약·리스크 고정, (코드)baseline
Phase 2    팀 구성 + 킥오프   TeamCreate + Agent로 멤버 스폰 + read-back ACK로 SMM 공유
Phase 3    협업 실행 ★       claim 루프, 폐쇄루프 핸드오프, monitor 교차검증(코드=실행), 적응/백업
Phase 4    종합 + 게이트     코드=빌드/테스트 green·회귀 없음 / 산문=rubric 각 항목 OK → final/
Phase 5    디브리핑 + 정리    AAR(debrief.md) + scorecard, shutdown→TeamDelete, 진화 반영
```

★ Big Five가 실제로 작동하는 구간:
- **핸드오프**는 폐쇄 루프 3단(발신→ACK 재진술→검증), 가능하면 peer-to-peer — 단방향·데드락 방지
- **각 모듈 완료 직후** monitor가 교차 검증 — 코드면 실제 빌드/테스트 실행, 산문이면 보이스/canon/사실 대조
- **DRIFT/정체/위험가정 붕괴** → 리더 적응(SMM 갱신 후 브로드캐스트) 또는 백업(재할당)
- **불확실/이견**은 FLAG-UNCERTAIN/DISSENT로 가시화 — 추측으로 덮지 않음

---

## 산출물 위치

| 경로 | 내용 |
|------|------|
| `_workspace/SMM.md` | 공유 정신 모델 (단일 진실 원천, 공동 소유) |
| `_workspace/contracts/` | 코드 과제의 인터페이스 계약 스키마 파일 |
| `_workspace/{phase}_{agent}_{artifact}.md` | contributor 중간 산출물 (코드는 워크트리/소스) |
| `_workspace/monitor/round_{N}_report.md` | 교차 검증 보고 (OK/DRIFT/BLOCKED) |
| `_workspace/monitor/scorecard.md` | 팀 성과 스코어카드 (진화 신호) |
| `_workspace/debrief.md` | 사후 검토 (AAR 4문항) |
| `_workspace/final/` | 최종 산출물 |

`_workspace/`는 감사 추적용으로 보존된다(`.gitignore` 처리 — 커밋 안 됨).

---

## 한계와 튜닝

- **독립 과제엔 과하다** — 경계면 없으면 팀워크 오버헤드가 이득을 넘음. Phase 0.5 triage가 팬아웃을 권고한다.
- **강도를 상호의존성에 맞춰라** — 낮은 상호의존엔 경량 모드(전담 monitor 생략), 높을 때만 풀 프로토콜.
- **폐쇄 루프는 선택적** — 모든 메시지에 ACK 요구하면 팀 마비. 결과 좌우 메시지에만.
- **모니터링은 신뢰 보존** — 전부 재검증하면 상호 신뢰 붕괴. 신뢰 등급에 따라 비싼 경계면·변경분에만 집중.
- **적응은 비싸다** — 사소한 변동마다 재계획하면 진척 없음. 목표 위협 시에만 (단 위험 가정 붕괴는 사전 신호).
- **창의는 풀어줘라** — [soft] 제약(보이스·플롯 탐색)은 드래프트 단계에서 막지 않고, [hard] 제약(canon·사실·계약·빌드)만 차단.

---

## 진화

하네스는 고정물이 아니다. 매 실행 후 **디브리핑(AAR)과 스코어카드**를 근거로 에이전트·스킬·오케스트레이터를 갱신하고, 변경은 `CHANGELOG.md`와 `CLAUDE.md` 변경 이력에 기록한다. 같은 DRIFT가 2회 반복되거나 특정 에이전트가 반복 실패하면 구조 수정을 검토한다. (v2 자체가 이 진화 과정의 산물 — 4개 렌즈의 에이전트 팀 분석 → 개선 반영 → 독립 평가.)

---

## 참고

- 이론 해설: [`docs/TEAM-SCIENCE.md`](docs/TEAM-SCIENCE.md)
- 변경 이력: [`CHANGELOG.md`](CHANGELOG.md)
- 핵심 논문: Salas, E., Sims, D. E., & Burke, C. S. (2005). *Is there a "Big Five" in Teamwork?* Small Group Research, 36(5), 555–599.
