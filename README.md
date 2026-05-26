# Team Big Five Harness

Team Science의 **"팀워크 빅파이브"**(Salas, Sims & Burke, 2005)를 Claude Code의 에이전트 팀에 적용한 하네스. 더 똑똑한 에이전트가 아니라 **더 나은 팀워크**로 협업 과제의 결과 품질을 높인다.

> 한 줄 가설: 에이전트 팀의 성과 병목은 개별 지능이 아니라 *조율의 질*이다. 조율 메커니즘을 명시적 프로토콜로 강제하면 같은 모델로도 결과가 좋아진다.

---

## 설치 (Claude Code 플러그인)

이 저장소는 Claude Code 플러그인이다. 로컬 설치 두 가지 방법:

### 방법 A — 로컬 마켓플레이스 (권장)
```bash
# 1. 이 저장소를 클론
git clone https://github.com/tobyilee/team-bigfive.git
cd team-bigfive

# 2. 로컬 마켓플레이스 등록 (저장소 루트 경로)
claude plugin marketplace add "$(pwd)"

# 3. 플러그인 설치
claude plugin install team-bigfive@team-bigfive-marketplace
```

### 방법 B — 개발용 직접 로드 (설치 없이)
```bash
claude --plugin-dir /path/to/team-bigfive
```

설치 후 스킬·에이전트는 `team-bigfive:` 네임스페이스로 노출된다 (예: 스킬 `team-bigfive:team-bigfive-orchestrator`, 에이전트 `team-bigfive:team-lead`).

**업데이트/제거:**
```bash
claude plugin marketplace update team-bigfive-marketplace
claude plugin uninstall team-bigfive@team-bigfive-marketplace
```

---

## 빠른 시작

협업이 필요한 과제를 줄 때 트리거 표현을 쓰면 오케스트레이터가 작동한다:

```
"이 API 설계하고 프론트 연동까지 팀으로 해줘"
"에이전트 팀으로 이 리서치 종합해줘"
"팀 빅파이브로 이 기능 구현해줘"
```

트리거 키워드: `팀으로`, `에이전트 팀`, `팀 빅파이브`, `고성과 팀`, `여러 에이전트로`, `협업으로 처리`
→ `team-bigfive-orchestrator` 스킬이 실행된다.

**언제 쓰나:** 산출물이 서로 맞물리는 과제(경계면이 있는 과제 — API↔프론트, 세계관↔플롯, 분석↔검증).
**언제 안 쓰나:** 단일 작업, 단순 질문, 경계면 없는 독립 병렬 작업(이건 일반 서브 에이전트 팬아웃이 낫다).

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
| **Team Leadership** | 팀장 | `team-lead` (오케스트레이터 메인) — 목표·배분·종합 |
| **Mutual Performance Monitoring** | 동료 작업 관찰 | `performance-monitor` — 경계면 교차 검증 |
| **Backup Behavior** | 바쁜 동료 거들기 | 유휴 팀원 작업 claim / 리더 재할당 |
| **Adaptability** | 전략 수정 | 오케스트레이터 적응 체크포인트 |
| **Team Orientation** | 팀 우선 태도 | 에이전트 정의 내재 원칙 |
| *Shared Mental Models* | 머릿속 공유 이해 | `_workspace/SMM.md` 단일 진실 원천 |
| *Closed-Loop Communication* | 복창 | `SendMessage` + ACK 재진술 |
| *Mutual Trust* | 동료 신뢰 | "검증된 산출물 재작업 금지" 원칙 |

**핵심 설계 결정 2개:**
1. **공유 정신 모델을 파일로 외재화** (`_workspace/SMM.md`) — 에이전트 컨텍스트는 휘발하므로 단일 진실 원천 파일이 필요. 모든 팀원이 작업 단위마다 읽고 갱신.
2. **모니터링을 전담 에이전트로 분리** — 에이전트는 자기 작업에 매몰돼 경계면을 놓침. `performance-monitor`가 교차 검증을 보장.

자세한 이론 해설은 **[`docs/TEAM-SCIENCE.md`](docs/TEAM-SCIENCE.md)** 참조.

---

## 구조

```
.claude-plugin/
├── plugin.json                      # 플러그인 매니페스트
└── marketplace.json                 # 로컬 마켓플레이스 카탈로그
agents/                              # 누가 (역할 정의)
├── team-lead.md                     # 리더십 + 적응성
├── contributor.md                   # 팀 지향 + 백업 + 신뢰 (N명 스폰)
└── performance-monitor.md           # 상호 모니터링 + 백업 촉발
skills/                              # 어떻게 (절차)
├── team-bigfive-orchestrator/       # 8개 메커니즘을 워크플로우로 조율
│   ├── SKILL.md
│   └── references/bigfive-theory.md
├── shared-mental-model/             # 조율 메커니즘 1
├── closed-loop-comms/               # 조율 메커니즘 2
└── mutual-monitoring/               # Big Five 2·3 (모니터링+백업)
docs/TEAM-SCIENCE.md                 # 이론 상세 해설
CLAUDE.md                            # 하네스 포인터 + 변경 이력
```

에이전트 = "누가", 스킬 = "어떻게". 상호 신뢰·팀 지향성은 절차가 아니라 *태도*라서 별도 스킬이 아닌 에이전트 정의의 원칙으로 내재화했다.

---

## 동작 흐름

```
Phase 0  컨텍스트 확인     초기/새/부분 재실행 판별 (_workspace/ 존재 여부)
Phase 1  준비 + SMM 초기화  목표·완료기준·용어·인터페이스를 SMM.md에 고정
Phase 2  팀 구성           TeamCreate(monitor + contributor N) + TaskCreate
Phase 3  협업 실행 ★       contributor가 SMM 읽고/갱신, 폐쇄루프 핸드오프,
                          monitor가 경계면 교차검증, DRIFT시 리더 적응
Phase 4  종합             리더가 산출물 수집 → SMM 기준 통합 → final/
Phase 5  정리 + 진화       팀 해체, _workspace 보존(감사추적), 피드백 수집
```

★ Big Five가 실제로 작동하는 구간:
- **핸드오프**는 폐쇄 루프 3단(발신→ACK 재진술→검증)으로 — 단방향 전달의 조용한 실패 방지
- **각 모듈 완료 직후** monitor가 경계면 교차 검증 — "존재 확인"이 아니라 "두 산출물 shape 대조"
- **DRIFT/정체 감지** → 리더 적응(SMM 갱신 후 브로드캐스트) 또는 백업(재할당)

---

## 산출물 위치

| 경로 | 내용 |
|------|------|
| `_workspace/SMM.md` | 공유 정신 모델 (단일 진실 원천, 공동 소유) |
| `_workspace/{phase}_{agent}_{artifact}.md` | contributor 중간 산출물 |
| `_workspace/monitor/round_{N}_report.md` | 교차 검증 보고 (OK/DRIFT/BLOCKED) |
| `_workspace/final/` | 최종 산출물 |

`_workspace/`는 감사 추적용으로 보존된다(`.gitignore` 처리 — 커밋 안 됨).

---

## 한계와 튜닝

- **독립 과제엔 과하다** — 경계면 없으면 팀워크 오버헤드가 이득을 넘음. 서브 에이전트 팬아웃을 써라.
- **폐쇄 루프는 선택적** — 모든 메시지에 ACK 요구하면 팀 마비. 결과 좌우 메시지에만.
- **모니터링은 신뢰 보존** — 전부 재검증하면 상호 신뢰 붕괴. 변경분·경계면에만 집중.
- **적응은 비싸다** — 사소한 변동마다 재계획하면 진척 없음. 목표 위협 시에만.

---

## 진화

하네스는 고정물이 아니다. 매 실행 후 피드백을 반영해 에이전트·스킬·오케스트레이터를 갱신하고, 변경은 `CLAUDE.md` 변경 이력에 기록한다. 같은 피드백이 2회 반복되거나 특정 에이전트가 반복 실패하면 구조 수정을 검토한다.

---

## 참고

- 이론 해설: [`docs/TEAM-SCIENCE.md`](docs/TEAM-SCIENCE.md)
- 핵심 논문: Salas, E., Sims, D. E., & Burke, C. S. (2005). *Is there a "Big Five" in Teamwork?* Small Group Research, 36(5), 555–599.
