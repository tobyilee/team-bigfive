---
name: team-bigfive-orchestrator
description: "Team Science의 Team Big Five 이론(팀 리더십·상호 성과 모니터링·백업 행동·적응성·팀 지향성 + 공유 정신 모델·폐쇄 루프 커뮤니케이션·상호 신뢰)을 에이전트 팀에 적용해 고성과로 과제를 수행하는 오케스트레이터. 여러 에이전트의 협업이 필요한 모든 과제 — 조사·분석·설계·구현·검증·작성 등 — 에 사용. '팀으로', '에이전트 팀', '팀 빅파이브', 'team bigfive', '고성과 팀', '협업으로 처리', '여러 에이전트로' 요청 시 반드시 트리거. 후속 작업: 재실행·다시 실행·업데이트·수정·보완·부분 재실행·이전 결과 개선 요청 시에도 반드시 이 스킬을 사용."
---

# Team Big Five Orchestrator

Team Science의 "팀워크 빅파이브"(Salas, Sims & Burke, 2005)를 에이전트 팀에 적용하는 오케스트레이터. 차별점은 **조율 메커니즘을 명시적 프로토콜로 강제**한다는 것 — 더 똑똑한 에이전트가 아니라 더 나은 팀워크로 성과를 낸다.

이론 상세는 `references/bigfive-theory.md` 참조 (메커니즘의 "왜"가 필요할 때 로드).

## 실행 모드: 에이전트 팀 (기본)

`TeamCreate`로 팀을 구성하고, 공유 작업 목록(`TaskCreate`)·`SendMessage`로 자체 조율한다. 모든 에이전트는 `model: "opus"`.

## 에이전트 구성

| 팀원 | 에이전트 타입 | Big Five 담당 | 사용 스킬 | 출력 |
|------|-------------|--------------|----------|------|
| team-lead (=리더) | custom | Team Leadership + Adaptability | shared-mental-model, closed-loop-comms | `_workspace/SMM.md`, `_workspace/final/` |
| contributor-{n} | custom | Team Orientation + Backup + 부분 Monitoring | shared-mental-model, closed-loop-comms | `_workspace/{phase}_{name}_{artifact}.md` |
| performance-monitor | custom (general-purpose 도구셋) | Mutual Performance Monitoring + Backup 촉발 | mutual-monitoring, closed-loop-comms | `_workspace/monitor/round_{N}_report.md` |

> contributor 수는 과제 분해 결과에 따라 1~5명. specialization(전문 영역)은 리더가 스폰 시 프롬프트로 주입한다.

## 워크플로우

### Phase 0: 컨텍스트 확인 (후속 작업 지원)
1. `_workspace/` 존재 여부 확인
2. 분기:
   - **미존재** → 초기 실행. Phase 1로
   - **존재 + 부분 수정 요청** → 부분 재실행. 해당 contributor만 재호출, 이전 산출물 경로를 프롬프트에 포함해 피드백 반영
   - **존재 + 새 입력** → 새 실행. 기존 `_workspace/`를 `_workspace_{YYYYMMDD_HHMMSS}/`로 이동 후 Phase 1

### Phase 1: 준비 + Shared Mental Model 초기화
1. 사용자 과제를 분석 — 무엇이 목표이고 어떻게 전문 영역으로 분해되는가
2. `_workspace/` 생성
3. **`shared-mental-model` 스킬로 `_workspace/SMM.md` 초기화** — 공유 목표·완료 기준·용어집·작업 맵·알려진 인터페이스를 채운다. 이것이 팀의 단일 진실 원천이다.

### Phase 2: 팀 구성
1. 팀 생성:
   ```
   TeamCreate(team_name: "bigfive-team", members: [
     { name: "performance-monitor", agent_type: "performance-monitor", model: "opus",
       prompt: "mutual-monitoring 스킬을 사용. SMM 기준으로 contributor 산출물을 점진적 교차 검증하고 정체/실패를 리더에 보고하라." },
     { name: "contributor-1", agent_type: "contributor", model: "opus",
       prompt: "specialization: {영역1}. 완료 기준: {...}. SMM을 읽고 시작, 결정 시 갱신, 폐쇄 루프로 통신하라." },
     { name: "contributor-2", agent_type: "contributor", model: "opus",
       prompt: "specialization: {영역2}. ..." }
     // 필요 수만큼
   ])
   ```
   > 리더(team-lead)는 이 오케스트레이터를 실행하는 메인 컨텍스트가 겸한다. 별도 팀원으로 스폰하지 않는다.
2. 작업 등록 — 팀원당 4~6개, 의존성은 `depends_on`:
   ```
   TaskCreate(tasks: [
     { title: "{영역1 작업}", assignee: "contributor-1" },
     { title: "{영역2 작업}", assignee: "contributor-2", depends_on: ["{영역1 작업}"] },
     { title: "round-1 교차 검증", assignee: "performance-monitor", depends_on: [...] }
   ])
   ```

### Phase 3: 협업 실행 (Big Five 작동 구간)
**실행 방식:** 팀원 자체 조율 + 리더 모니터링

1. contributor들이 작업을 claim하고 수행한다. 각자 **작업 전 SMM을 Read**, 타인 영향 결정 시 **SMM 갱신 + 폐쇄 루프 통보**.
2. 경계면 핸드오프는 `closed-loop-comms` 3단 루프(발신→ACK 재진술→검증)로 전달한다.
3. 각 모듈 완료 직후 `performance-monitor`가 **점진적 교차 검증** — 경계면을 SMM 기준으로 대조, OK/DRIFT/BLOCKED 판정.
4. **적응 체크포인트 (Adaptability):** monitor가 DRIFT/BLOCKED를 보고하면 리더가 멈추고 재평가한다. 계획을 바꾸면 **SMM을 먼저 갱신**한 뒤 팀에 브로드캐스트.
5. **백업 (Backup):** 정체/실패 팀원이 감지되면 유휴 팀원이 claim하거나 리더가 재할당 (`mutual-monitoring` Part 2).

**산출물 저장:** 위 에이전트 구성 표의 출력 경로.

**리더 모니터링:** 유휴 알림 수신, `TaskGet`으로 진행률 확인, monitor 보고를 적응 결정 근거로 사용.

### Phase 4: 종합
1. 모든 작업 완료 대기 (`TaskGet`)
2. 최종 monitor round가 OK인지 확인 — DRIFT 잔존 시 해당 contributor에 마지막 수정 요청
3. 리더가 contributor 산출물을 Read로 수집, SMM의 목표·완료 기준 대조하며 통합
4. 최종 산출물: `_workspace/final/{filename}` (+ 사용자 지정 경로)

### Phase 5: 정리 + 진화
1. 팀원 종료 요청 (SendMessage), 팀 정리 (`TeamDelete`)
2. `_workspace/` 보존 (SMM·monitor 보고 = 감사 추적)
3. 결과 요약 보고
4. **피드백 요청 (Phase 7 진화):** "결과·팀 구성·워크플로우에서 바꾸고 싶은 점이 있나요?" 피드백 유형에 따라 스킬/에이전트/오케스트레이터를 갱신하고 CLAUDE.md 변경 이력에 기록.

## 데이터 흐름
```
[리더: SMM 초기화] → TeamCreate → contributor들 ←폐쇄루프→ 서로
        │                              │  (SMM 읽기/갱신)
        │                              ↓
        │                    _workspace/{artifacts}
        │                              ↓
        │              [performance-monitor: 교차 검증]
        │                     │ OK         │ DRIFT/BLOCKED
        │                     ↓            ↓
        │                  통과      [리더: 적응 → SMM 갱신 → 재할당]
        ↓                                  │
   [리더: Read + 통합] ←──────────────────┘
        ↓
   _workspace/final/
```

## 데이터 전달 프로토콜
- **태스크 기반**(조율): `TaskCreate`/`TaskUpdate`
- **메시지 기반**(실시간): `SendMessage` + 폐쇄 루프 ACK
- **파일 기반**(산출물·SMM): `_workspace/`, 컨벤션 `{phase}_{agent}_{artifact}.{ext}`

## 에러 핸들링
| 상황 | 전략 |
|------|------|
| contributor 1명 실패 | 리더가 폐쇄 루프 상태 확인 → 1회 재시작 → 재실패 시 백업 재할당 |
| 과반 실패 | 사용자에 알리고 진행 여부 확인 |
| monitor가 같은 DRIFT 2회 보고 | 구조적 문제 — SMM 자체 결함 의심, 리더가 적응(재계획) |
| 산출물 충돌 | 삭제 금지, SMM 결정 로그에 출처 병기 |
| ACK 무응답 | 재전송 → 무응답 지속 시 백업 트리거 |
| 타임아웃 | 부분 결과 사용, 미완료 팀원 종료, 보고서에 누락 명시 |

## 테스트 시나리오
### 정상 흐름
1. 사용자가 협업 과제 제공 (예: "이 API 설계하고 프론트 연동까지")
2. Phase 1에서 SMM 초기화 (목표·인터페이스 스키마 고정)
3. Phase 2에서 팀 구성 (contributor-api, contributor-ui, performance-monitor)
4. Phase 3에서 contributor-api가 스키마 결정 → SMM 갱신 → contributor-ui에 폐쇄 루프 통보 → ui가 ACK 재진술
5. monitor가 api 출력 ↔ ui 입력 경계면 교차 검증 → OK
6. Phase 4에서 리더가 통합 → `_workspace/final/` 생성
7. Phase 5에서 정리 + 피드백 요청

### 에러 흐름
1. Phase 3에서 contributor-api가 스키마를 SMM에 안 올리고 작업
2. monitor가 경계면 검증 → DRIFT (ui 기대 shape ↔ api 실제 shape 불일치)
3. monitor가 api에 폐쇄 루프 플래그, 리더에 보고
4. 리더가 적응 체크포인트 발동 → SMM 인터페이스 섹션 보강 → 팀 브로드캐스트
5. api가 SMM 기준 수정, monitor 재검증 → OK
6. 정상 통합으로 복귀
