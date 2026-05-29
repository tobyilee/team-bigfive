---
name: team-bigfive-orchestrator
description: "Team Science의 Team Big Five 이론(팀 리더십·상호 성과 모니터링·백업 행동·적응성·팀 지향성 + 공유 정신 모델·폐쇄 루프 커뮤니케이션·상호 신뢰)을 에이전트 팀에 적용해 고성과로 과제를 수행하는 오케스트레이터. 여러 에이전트의 협업이 필요한 모든 과제 — 조사·분석·설계·구현·검증·작성 등 — 에 사용. '팀으로', '에이전트 팀', '팀 빅파이브', 'team bigfive', '고성과 팀', '협업으로 처리', '여러 에이전트로' 요청 시 반드시 트리거. 후속 작업: 재실행·다시 실행·업데이트·수정·보완·부분 재실행·이전 결과 개선 요청 시에도 반드시 이 스킬을 사용."
---

# Team Big Five Orchestrator

Team Science의 "팀워크 빅파이브"(Salas, Sims & Burke, 2005)를 에이전트 팀에 적용하는 오케스트레이터. 차별점은 **조율 메커니즘을 명시적 프로토콜로 강제**한다는 것 — 더 똑똑한 에이전트가 아니라 더 나은 팀워크로 성과를 낸다.

이론 상세는 `references/bigfive-theory.md` 참조 (메커니즘의 "왜"가 필요할 때 로드).

## 과제 유형 적응 (Task-type adaptation)
이 하네스는 코드 전용이 아니다. 메커니즘은 동일하고 *계약·완료기준·모니터링 대상*만 과제 유형에 맞춰 바뀐다. 리더는 Phase 1에서 유형을 먼저 판별한다.

| 과제 유형 | 경계면 계약 | 완료 기준 | 모니터링 대상 |
|----------|-----------|----------|-------------|
| 코드/설계 | API/타입 스키마, 파일 소유권 | 빌드·테스트·린트 통과 | 실행 게이트 + shape 대조 |
| 리서치/분석 | 사실 원장(주장→출처), 용어 | 모든 주장 출처 有, rubric | 무근거 주장·인용 정확성 |
| 문서/보고서 | 섹션 개요, 용어, 독자 | rubric 수용 기준 | 용어 일관·중복/공백 |
| 창작(소설 등) | 보이스 시트, canon, 타임라인 | rubric 수용 기준 | 보이스 드리프트·canon 모순 |
| 마케팅/카피 | 포지셔닝, 브랜드 보이스 | rubric 수용 기준 | 메시지·톤·CTA 일관 |

상세는 `shared-mental-model` 스킬의 "과제 유형별 적응" 표.

## 실행 모드: 에이전트 팀 (기본)
`TeamCreate`로 팀을 만들고, 멤버는 **Agent 도구로 스폰**하며, 공유 작업 목록(`TaskCreate`/`TaskUpdate`)·`SendMessage`로 자체 조율한다.

> **도구 사실 (정확히 준수):** `TeamCreate`는 `team_name`/`agent_type`/`description`만 받는다 — `members` 배열은 없다. 멤버는 **Agent 도구**로 `team_name`+`name`+`subagent_type`을 주어 따로 스폰한다. 작업 의존은 `TaskCreate`의 인자가 아니라 `TaskUpdate(addBlockedBy/addBlocks)`로 건다. 담당 배정은 `TaskUpdate(owner)`. 종료는 `SendMessage({type:"shutdown_request"})` → 각 멤버의 `shutdown_response` 수신 후 `TeamDelete`.

## 워크플로우

### Phase 0: 컨텍스트 확인 (후속 작업 지원)
1. `_workspace/` 존재 여부 확인
2. 분기:
   - **미존재** → 초기 실행. Phase 0.5로
   - **존재 + 부분 수정 요청** → 부분 재실행. 해당 contributor만 재호출, 이전 산출물 경로를 프롬프트에 포함해 피드백 반영
   - **존재 + 새 입력** → 새 실행. 기존 `_workspace/`를 `_workspace_{YYYYMMDD_HHMMSS}/`로 이동 후 Phase 0.5

### Phase 0.5: 상호의존성 triage (팀이 필요한가 / 얼마나 무겁게)
Big Five 효과는 **과제 상호의존성**에 비례한다(논문 핵심). 무겁게 갈지 결정:

| 상호의존성 | 신호 | 실행 모드 |
|-----------|------|----------|
| 독립(병렬) | 산출물이 안 맞물림 | **팀 미구성** — 일반 서브 에이전트 팬아웃 권고하고 종료 |
| 낮음(pooled) | 약한 공유, 경계면 적음 | **경량** — SMM + 리더만, 전담 monitor 생략, 리더가 종합 시 1회 교차 검증 |
| 높음(순차/상호) | 강한 경계면(API↔UI, canon↔플롯) | **풀 Big Five** — SMM + contributor N + 전담 monitor + 폐쇄 루프 |

강도(폐쇄 루프 범위, monitor 주기, 적응 민감도)를 이 등급에 맞춰 조절한다. 낮은 상호의존 과제에 풀 프로토콜을 쓰면 오버헤드가 이득을 넘는다.

### Phase 1: 준비 + Shared Mental Model 초기화
1. 과제 유형 판별 + 분해 — 무엇이 목표이고 어떻게 전문 영역으로 나뉘는가
2. `_workspace/` 생성
3. **`shared-mental-model` 스킬로 `_workspace/SMM.md` 초기화** — 공유 목표·**완료 기준(rubric/실행 게이트)**·용어집·작업 맵·**결정권(decision rights)**·알려진 계약·**리스크/대응(§9)**을 채운다. 이것이 단일 진실 원천이다.
4. **코드 과제:** 기존 빌드/테스트 **baseline**을 캡처(현재 통과 상태) — 회귀 가드 기준선.

### Phase 2: 팀 구성 + 킥오프 브리핑
1. 팀 생성 (lead = 현재 메인 컨텍스트):
   ```
   TeamCreate(team_name: "bigfive-team", agent_type: "team-lead",
              description: "{과제 한 줄}")
   ```
2. 멤버 스폰 — **Agent 도구로 각각** (model은 아래 라우팅):
   ```
   Agent(subagent_type: "performance-monitor", team_name: "bigfive-team",
         name: "performance-monitor", model: "sonnet",
         prompt: "mutual-monitoring 스킬 사용. SMM 기준 교차 검증, 코드면 빌드/테스트 실행, 정체/실패 리더 보고, 종료 전 scorecard 작성.")
   Agent(subagent_type: "contributor", team_name: "bigfive-team",
         name: "contributor-1", model: "{sonnet 기본 / 고난도면 opus}",
         prompt: "specialization: {영역1}. 완료 기준: {...}. SMM Read 후 시작, 결정 시 갱신, 폐쇄 루프 통신, 불확실하면 FLAG-UNCERTAIN.")
   // 필요 수만큼 contributor-2 …
   ```
   > 리더는 이 오케스트레이터를 실행하는 메인 컨텍스트가 겸한다 — **별도 스폰하지 않는다.** `agent_type:"team-lead"`은 현재 컨텍스트의 페르소나.
3. 작업 등록 + 의존:
   ```
   TaskCreate(subject:"{영역1 작업}", description:"...")        // → id 1
   TaskCreate(subject:"{영역2 작업}", description:"...")        // → id 2
   TaskUpdate(taskId:"2", addBlockedBy:["1"])                  // 의존
   TaskUpdate(taskId:"1", owner:"contributor-1")               // 배정(또는 미배정→claim)
   ```
4. **킥오프 브리핑 (SMM을 *공유*로 만드는 단계):** 리더가 SMM 목표 + 각 멤버의 인터페이스 의무를 `SendMessage`로 전달, 각 멤버가 **자기 슬라이스를 read-back ACK**(`closed-loop-comms`). 이 1라운드 폐쇄 루프로 SMM 수렴을 보장한 뒤 작업 claim 시작.

### Phase 3: 협업 실행 (Big Five 작동 구간)
**조율 루프 (구체):**
1. 스폰된 contributor는 `TaskList` → 가장 낮은 ID의 unblocked·미배정 작업을 `TaskUpdate(owner=self)`로 claim → 수행. 작업 전 **SMM Read**, 타인 영향 결정 시 **SMM 갱신 + 폐쇄 루프 통보**. 완료 시 `TaskUpdate(status:completed)` → 다시 `TaskList`.
2. 경계면 핸드오프는 `closed-loop-comms` 3단(발신→ACK 재진술→검증). 가능하면 **peer-to-peer** 직접 통신(리더 릴레이 데드락 회피).
3. 각 모듈 완료 직후 `performance-monitor`가 **점진적 교차 검증** — 코드면 **빌드/테스트 실행(Part 0)** 후 경계면 대조, 산문이면 SMM의 스타일/사실/연속성 대조. OK/DRIFT/BLOCKED 판정 → `round_{N}_report.md`.
4. **적응 (Adaptability):** monitor가 DRIFT/BLOCKED 또는 SMM §9 위험 가정 붕괴를 보고하면 리더가 멈추고 재평가. 계획 변경 시 **SMM 먼저 갱신** 후 브로드캐스트. 계약 버전 증가 → 하위 "검증됨" 무효화.
5. **백업 (Backup):** 정체(idle ≠ 정체; task 무변화 정의) 감지 시 유휴 팀원 claim 또는 리더 재할당 (`mutual-monitoring` Part 2).
6. **관찰성:** 리더는 monitor 라운드/웨이브 완료마다 사용자에게 진척 1줄 보고(완료/전체, 최신 판정). 긴 실행은 `_workspace/SMM.md`·`monitor/` 가 사용자 점검 surface.

### Phase 4: 종합 + 게이트
1. 모든 작업 완료 대기 (`TaskList`/`TaskGet`)
2. **완료 게이트:**
   - 코드: 통합 트리에서 **빌드·테스트·린트 실행 → 전부 green**, baseline 회귀 없음. 실패 시 책임 모듈 소유자에 반환, Phase 3 복귀.
   - 산문: monitor가 SMM 완료-기준 **rubric의 각 항목을 OK/DRIFT 판정**, 잔존 DRIFT는 해당 contributor에 마지막 수정 요청.
3. 리더가 contributor 산출물을 Read로 수집, SMM 목표·rubric 대조하며 통합. (코드의 "산출물"은 워크트리/소스 + diff이지 markdown 요약이 아니다.)
4. 최종 산출물: `_workspace/final/{filename}` (+ 사용자 지정 경로)

### Phase 5: 디브리핑 + 정리 + 진화
1. **디브리핑 (After-Action Review)** — 해체 전 `_workspace/debrief.md` 작성, monitor `scorecard.md`를 근거로 4문항: (1) 목표가 무엇이었나 (2) 실제 무슨 일이 있었나(DRIFT/BLOCKED/적응 이벤트) (3) 격차 원인 (4) 다음에 유지/변경할 것.
2. 팀 종료: 각 멤버에 `SendMessage({type:"shutdown_request"})` → 모든 `shutdown_response`(approve) 수신 → `TeamDelete`. (활성 멤버가 남으면 TeamDelete 실패.)
3. `_workspace/` 보존 (SMM·monitor 보고·debrief = 감사 추적).
4. 결과 요약 보고.
5. **진화:** debrief의 "변경할 것" + 반복 DRIFT 패턴을 스킬/에이전트/오케스트레이터 개선으로 반영하고 CLAUDE.md 변경 이력에 기록. (사용자에 추가 피드백도 요청.)

## 폴백: 팀 도구 미가용 시
`TeamCreate`/`SendMessage` 등 팀 도구가 세션에 없으면 (deferred·옵션) 하드 실패하지 말고 **Agent 도구 팬아웃**으로 강등:
- 메인 컨텍스트가 리더 겸 통합자. contributor를 일반 서브 에이전트로 스폰(공유 task list 없음).
- 조율은 파일 기반 SMM + **리더 중재 핸드오프** — 리더가 각 산출물을 Read하고 인터페이스를 다음 contributor에 전달(서브 에이전트끼리는 직접 통신 불가).
- 폐쇄 루프 ACK는 리더 매개로 수행. monitor 역할은 리더가 별도 검증 서브 에이전트로 스폰하거나 직접 수행.

## 모델 라우팅 (전원 opus 금지)
- **team-lead(메인):** opus — 설계·종합·적응.
- **contributor:** 기본 sonnet, 고난도/설계 중심 specialization만 opus.
- **performance-monitor:** sonnet (구조 대조). 복잡한 실행 검증 해석이 필요하면 리더가 opus로 상향.

전원 opus는 비용·지연을 3~6배로 올리며 일상 레인엔 이득이 작다. 리더가 스폰 시 per-agent로 결정.

## 데이터 흐름
```
[리더: 유형판별·SMM초기화·triage] → TeamCreate → (Agent로 멤버 스폰) → 킥오프 브리핑(read-back ACK)
        │                                          contributor들 ←폐쇄루프(peer)→ 서로
        │                                                 │  (SMM 읽기/갱신)
        │                                                 ↓
        │                                       _workspace/{artifacts}, contracts/
        │                                                 ↓
        │                 [performance-monitor: 실행검증(코드)+경계면 교차검증]
        │                        │ OK            │ DRIFT/BLOCKED
        │                        ↓               ↓
        │                     통과         [리더: 적응 → SMM 갱신 → 재할당/백업]
        ↓                                         │
   [리더: 게이트(빌드/테스트 or rubric) → 통합] ←─┘
        ↓
   _workspace/final/  +  debrief.md + monitor/scorecard.md
```

## 데이터 전달 프로토콜
- **태스크 기반**(조율): `TaskCreate` + `TaskUpdate`(owner/addBlockedBy)
- **메시지 기반**(실시간): `SendMessage` + 폐쇄 루프 ACK / FLAG-UNCERTAIN / DISSENT
- **파일 기반**(산출물·SMM): `_workspace/`. 코드의 실제 산출물은 워크트리/소스 + diff이고 `_workspace/`엔 SMM·계약(`contracts/`)·monitor 보고·debrief를 둔다. 컨벤션 `{phase}_{agent}_{artifact}.{ext}`

## 에러 핸들링
| 상황 | 전략 |
|------|------|
| contributor 1명 실패 | 리더가 폐쇄 루프 상태 확인 → 1회 재시작 → 재실패 시 백업 재할당 |
| 과반 실패 | 사용자에 알리고 진행 여부 확인 |
| **빌드 실패(코드)** | 책임 모듈 소유자에 폐쇄 루프 + 1회 수정, 재실패 시 적응 |
| **테스트 실패/회귀(코드)** | 원인 모듈 식별, 해당 contributor에 반환, baseline 회귀는 차단 사유 |
| **머지 충돌(코드)** | 직렬화 재시도 + SMM 파일 소유권 재정의 |
| monitor가 같은 DRIFT 2회 보고 | 구조적 문제 — SMM 자체 결함 의심, 리더가 적응(재계획) |
| 산출물 충돌 | 삭제 금지, SMM 결정 로그/사실 원장에 출처 병기 |
| ACK 무응답 | 1회 재전송 → 무응답 지속 시 리더 에스컬레이션(백업). idle ≠ 정체 |
| 팀 도구 미가용 | 폴백(Agent 팬아웃)으로 강등 |
| 타임아웃 | 부분 결과 사용, 미완료 팀원 종료, 보고서에 누락 명시 |

## 테스트 시나리오

### 정상 흐름 (코드)
1. "이 API 설계하고 프론트 연동까지" → triage: 높은 상호의존 → 풀 Big Five
2. Phase 1: SMM 초기화(인터페이스 스키마를 `contracts/user.schema.json`으로 고정, baseline 캡처)
3. Phase 2: TeamCreate + Agent로 contributor-api/contributor-ui/performance-monitor 스폰, 킥오프 read-back ACK
4. Phase 3: api가 스키마 결정 → SMM+contracts 갱신 → ui에 폐쇄 루프 → ui ACK. monitor가 **빌드/테스트 실행** + 경계면 대조 → OK
5. Phase 4: 통합 트리 빌드/테스트 green → `final/`
6. Phase 5: debrief + scorecard + 정리

### 정상 흐름 (창작/리서치)
1. "이 단편 세계관·플롯·1~3장 팀으로" → triage: 높은 상호의존(canon↔플롯) → 풀 Big Five
2. Phase 1: SMM에 §6 보이스 시트·§8 canon/타임라인을 [hard]로 고정, rubric 완료기준 설정
3. Phase 2: contributor-world/contributor-plot/contributor-draft + monitor, 킥오프 ACK
4. Phase 3: world가 canon 결정 → SMM §8 갱신 → 폐쇄 루프. monitor가 보이스 드리프트·canon 모순 대조(소프트 위반은 FYI, canon 위반은 차단 DRIFT)
5. Phase 4: rubric 각 항목 OK → 통합 → `final/`
6. Phase 5: debrief + scorecard

### 에러 흐름
1. contributor-api가 스키마를 SMM에 안 올리고 작업
2. monitor가 빌드 실행 → ui 컴파일 실패 + 경계면 shape 불일치 → DRIFT
3. monitor가 api에 폐쇄 루프 플래그(구체 근거), 리더에 보고
4. 리더 적응 → SMM/contracts 인터페이스 보강(버전↑) → 브로드캐스트, ui "검증됨" 무효화
5. api 수정, ui 재검증, monitor 재실행 → green/OK
6. 정상 통합으로 복귀
