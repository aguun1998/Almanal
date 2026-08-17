# 알만알 리치마작 승률 행동 알고리즘 v1.23 — HUMAN-RT Ω-H23 Versioned Closed Runtime

> **Status:** Improvement Candidate / **NOT CANON**  
> **Parent:** v1.22 / HUMAN-RT Ω-H22
> **Parent v1.22 SHA-256:** `4eb753adcd2e8cede664a9449d6da865121fa9f1a83e5d2a348e00a67e4b7c74`
>
> **Historical Canon Root:** v1.11 / HUMAN-RT Ω-H11  
> **Parent SHA-256:** `24580d6e59628eae682856c5fb34430dbc0a4de49f8b0da6bf22798588120e69`  
> **Intermediate:** v1.12 Compact / `e78c073e58bcbab607dafe2b017e9cda2b0bd28c926b0e55d1895ae9f15f3be6`
>
> 알만알 13.3 Compact Kernel 방식으로 H1~H11의 현재 유효 규칙을 다시 컴파일했다.
> 이 문서는 과거 버전 누적본이 아니라 **현재 실행 정본 후보 하나**다.
>
> 목표:
>
> ```text
> 같은 핵심 능력 이상
> + 더 적은 활성 규칙
> + 4마/3마 규칙정합성
> + hard fact / heuristic / query prior 분리
> + offline calibration 가능
> ```

---

# 0. 한 줄 결론

```text
종료순위 목표
→ 룰프로필
→ 점수경로
→ 살아 있는 손구조
→ 위협/즉시론 가능성
→ 실제 행동 전수검사
→ 역할대표 최대 3개
→ 공수 controller
→ 필요할 때만 SOURCE/SIGNAL
→ 합법행동 1개
```

인간 실행 중에는:
- 벽패 배열 열거 금지,
- 상대 정확손/대기 확정 금지,
- 방총률/화료율 수치 계산 금지,
- 임의 가중합 금지.

대국 밖에서는 로그·시뮬레이터로 휴리스틱을 검증하고 다음 버전에 compile한다.

---

# 1. Goal Contract

## 1.1 deterministic terminal order

기본 terminal preference:

```text
1위 ≻ 2위 ≻ 3위 ≻ 4위    // 4P
1위 ≻ 2위 ≻ 3위           // 3P
```

동점은 `RuleProfile`의 공식 정산순서를 따른다.

```text
TerminalOrder ≠ unique lottery utility
```

즉 순위 사이의 cardinal utility gap은 자동으로 만들지 않는다.

## 1.2 Lottery incompleteness

확률분포 `A`, `B`가 교차하면 순위의 서수순서만으로 유일한 선호가 나오지 않는다.

4P:

```text
RankCDF(a) =
  <P(rank≤1), P(rank≤2), P(rank≤3)>
```

3P:

```text
RankCDF(a) =
  <P(rank≤1), P(rank≤2)>
```

```text
RankDom(a,b) ⇔
  모든 CDF축에서 a≥b
  ∧ 하나 이상 strict >
```

`RankDom`이면 모든 strictly monotone rank utility와 정합적인 공통우위다.

```text
crossing distributions
→ INCOMPARABLE_UNDER_TERMINAL_ORDER
```

별도 플랫폼 순위점·우마·오카·사용자 risk preference를 최적화하려면:

```text
LotteryProfile
```

을 명시해야 한다.

## 1.3 Human runtime

인간은 rank probability를 계산하지 않는다.

```text
GoalCard G =
  <UP, DOWN, HandRole,
   MinimumResult, SufficientResult, ForbiddenResult,
   RouteMode>
```

로 terminal order를 질적으로 projection한다.

---

# 2. Rule Algebra — 4P/3P 공통 커널

## 2.1 RuleProfile

```text
RuleProfile R = <
  PlayerCount,
  TileCapacity,
  RedTileFlags,
  ChiAllowed,
  PonAllowed,
  KanRules,
  KitaRules,
  RiichiRules,
  FuritenRules,
  RonResolution,
  TsumoPayment,
  Honba,
  Kyotaku,
  DrawPayments,
  AbortiveDraws,
  Nagashi,
  Tobi,
  RoundSequence,
  DealerContinuation,
  Termination,
  TieBreak,
  ScoreTable,
  Version
>
```

플레이어:

```text
P_R = {0,...,PlayerCount_R-1}
```

패종 우주 `T_R`는 34종 표기를 유지할 수 있지만 룰에서 제거된 패는 capacity 0이다.

```text
Cap_R(p) ∈ {0,1,2,3,4,...}
```

일반 리치 4P는 보통 4지만 **커널에 4를 하드코딩하지 않는다.**

## 2.2 physical upper bound — unique-instance accounting

`upper`의 입력 장수는 **서로 겹치지 않는 물리 인스턴스**로 센다.

현재 자기에게 이미 알려진 패:

```text
KnownToSelf_t(p) =
  SelfConcealed_t(p)
  + PublicUnique_t(p)
```

- `SelfConcealed`: 현재 내 비공개 손 안의 physical copies.
- `PublicUnique`: 현재 공개되어 식별 가능한 physical copies의 합집합.
  - 모든 river의 현재 공개패,
  - 모든 공개 meld / kan,
  - dora indicators,
  - 공개 nuki/kita,
  - 그 밖의 rule-visible tile.

중요:

```text
event appeared twice
≠ two physical tiles
```

예를 들어 상대가 버린 `5m`을 다른 상대가 PON하면
그 physical `5m`은:
- discard event history에도 있고,
- meld history에도 있지만

`PublicUnique`에는 **한 번만** 센다.

```text
VisibleInstanceLedger
```

는 event provenance와 physical identity를 분리한다.

구현에서 physical ID를 직접 가지면 ID 집합을 쓰고,
인간/단순 엔진에서는 current board transition으로 중복을 제거한다.

따라서:

```text
upper_t^R(p)
  = Cap_R(p)
  - KnownToSelf_t(p)
```

```text
0 ≤ OpponentCount_i(p) ≤ upper_t^R(p)
```

여기서 `upper`는:
- 상대 손,
- live wall,
- dead wall

등 **내게 아직 위치가 안 보이는 전체**의 상한이다.

적5 등 물리플래그는 별도 capacity/instance flag로 관리한다.

```text
PublicUnique ∩ SelfConcealed = ∅
```

을 representation invariant로 둔다.

이 규칙은:
- 4P,
- removed-tile sanma,
- 적5 개수 차이,
- 특수 tile profile

을 같은 물리법칙으로 처리한다.

## 2.2.1 Rule-derived transition functions

RuleProfile은 단순 데이터표가 아니라 다음 결정함수의 source다.

```text
ScoreTransition_R(state, concrete_legal_outcome)
  → next score/kyoku state
```

```text
rank_R(score_state, tie/termination rules)
  → current/terminal rank
```

```text
Termination_R(state)
  → {TERMINAL, NONTERMINAL}
```

구체 outcome에는 필요한 경우:
- winner/loser,
- ron/tsumo,
- dealer,
- han/fu or rule score class,
- honba/kyotaku,
- multi-ron resolution,
- abortive/draw transition

을 포함한다.

```text
ScoreTransition_R
≠ expected score transition
```

미래 확률을 넣는 함수가 아니라
**구체적 룰 결과의 정확한 상태전이**다.

## 2.3 legal action

```text
Legal_R(o_t)
```

만 행동후보다.

가능한 action class:

```text
WIN
PASS
DISCARD(x)
RIICHI(x)
CHI(m,x)          if R.ChiAllowed
PON(m,x)
DAIMINKAN
ANKAN
KAKAN
KITA             if R.KitaRules.enabled
ABORTIVE_DRAW     if legal
```

치/퐁은 직후 강제타패까지 compound action으로 평가한다.

```text
CALL = <meld, mandatory_discard>
```

대명깡·KITA처럼 replacement draw가 끼는 행동은
보지 않은 draw를 현재 가치로 선반영하지 않고,
행동 후 실제 draw에서 새 decision을 시작한다.

## 2.4 Legality basis / profile completeness

합법성 총성을 주장하려면 최소 하나가 필요하다.

```text
LegalityBasis ∈ {
  COMPLETE_RULE_PROFILE,
  AUTHORITATIVE_LEGAL_MASK
}
```

### normal strategic mode

```text
NormalMode
requires Complete(R)
```

여기서 `Complete(R)`은 현재 room의:
- legal actions,
- score transition,
- termination,
- furiten,
- special-action rules

를 결정하는 필드와 `Version`이 모두 채워졌다는 뜻이다.

### emergency legality-only rescue

플랫폼/client가 현재 행동기회의 정확한 legal mask를 주는 경우:

```text
AuthoritativeLegalMask
```

는 룰모델 일부가 손상돼도 **현재 행동의 합법성**만 보존할 수 있다.

그러나:

```text
LegalMask
⇏ correct score/goal/value evaluation
```

이므로 이 경우 정상 전략판정은 중지하고
`EMERGENCY / RULE_MODEL_INCOMPLETE`로만 반환한다.

### no basis

```text
¬Complete(R)
∧ ¬AuthoritativeLegalMask
→ LEGALITY_UNRESOLVED
```

이 상태에서는 "반드시 합법행동을 반환한다"고 주장하지 않는다.

실전 human profile은 대국 시작 전에 room `RuleProfile`을 확정해야 하며,
client-integrated profile은 legal mask를 추가 안전망으로 사용할 수 있다.


---

# 3. StrategyProfile — 룰과 전략을 분리

```text
StrategyProfile Σ_R = <
  ThreatPolicy,
  RiichiPolicy,
  CallPolicy,
  KanPolicy,
  KitaPolicy,
  AbortivePolicy,
  ControlCallPolicy,
  SpecialPolicy,
  ShapeFallback,
  DefenseFallback,
  SourcePolicy,
  SignalPolicy,
  CalibrationVersion,
  Version
>
```

```text
Rule legality ≠ strategy heuristic
```

```text
R.Version
```

은 rule semantics/configuration version이고,

```text
Σ.Version
```

은 현재 strategy-policy bundle version이다.

`CalibrationVersion`은 그 strategy bundle을 만든
offline calibration artifact의 provenance이며
`Σ.Version`과 같은 개념이 아니다.

```text
RuleVersionChange
→ all rule-conditional cache/Auth invalid

StrategyVersionChange
→ policy outputs / route-policy assumptions / special order invalid
→ hard observed facts remain
```


특히:

```text
Σ_4P ≠ Σ_3P
```

이며 4P 휴리스틱을 "공격성 +α" 식으로 3P에 이식하지 않는다.

공유할 수 있는 것은:
- 합법성,
- 점수전이,
- 샹텐/역의 구조,
- hard tile capacity,
- 현물/후리텐,
- 후보생성 protocol

이고,

별도 검증할 것은:
- open-hand threat threshold,
- riichi/dama,
- defense fallback,
- call/kan/kita policy,
- source/signal heuristic.

### 3.1 policy subcontracts

`ThreatPolicy`는 세 함수를 묶는다.

```text
ThreatPolicy = <
  ActiveRule,
  KeyPriority,
  HeavyRule
>
```

```text
ActiveRule(Opp_i,G,state) → {ACTIVE,INACTIVE}
KeyPriority(W,G,public)    → KeyThreat
HeavyRule(W,G,public)      → {TRUE,FALSE}
```

이 셋은 factual tenpai detector가 아니라
**공수 controller용 scoped policy**다.

`ShapeFallback`:

```text
ShapeFallback = <
  ApplicabilityScope,
  FeatureOrder,
  IncomparableFallback
>
```

입력 feature가 hard-derived여도 `FeatureOrder` 자체는 policy다.

`DefenseFallback`:

```text
DefenseFallback = <
  SafetyGate,
  NonFactOrder,
  EmergencyOrder,
  ApplicabilityScope
>
```

```text
SafetyGate(i,a) → {PASS,FAIL,UNKNOWN}
```

- FactSafe는 언제나 PASS의 hard witness가 될 수 있다.
- non-FactSafe PASS는 해당 profile의 calibrated scope에서만 허용한다.
- scope 밖은 UNKNOWN.

```text
UNKNOWN ≠ PASS
```

`SpecialPolicy`는 새 전략축이 아니라
이미 존재하는 special class들의 **bounded resolution contract**다.

```text
SpecialPolicy = <
  ClassOrder,              // COMMIT / TRANSFORM / CONTROL
  TransformOrder,          // eligible KAN/KITA reps
  ControlOrder,            // CONTROL_CALL / ABORTIVE reps
  IncomparableFallback
>
```

모든 순서는 outcome 전에 profile version에 봉인한다.

```text
SpecialPolicy
≠ new evidence
```

---

# 4. Observed State / Information Admission

```text
ObservedState o_t = <
  R,
  score vector,
  kyoku/honba/kyotaku/dealer/termination state,
  self hand/open meld/riichi/furiten,
  all public discards with order and source if platform supports,
  calls/kans/kita,
  dora indicators,
  remaining live-wall count if public,
  current legal opportunity
>
```

상태는 사건이력을 보존한다.

하지만 인간 active memory에는 현재 행동을 바꿀 수 있는 것만 올린다.

```text
Admit(e,q) ⇔
  Observable(e)
  ∧ Fresh(e)
  ∧ can_change(e, {Legal,Goal,Hand,Threat,Safety,Choice})
```

바로 버릴 것:
- "아직 안 버림"만으로 보유/대기 추측,
- 감정·장고 한 번,
- 정확 벽배치,
- 정확 화료/방총 확률,
- 행동을 바꾸지 않는 오래된 가설,
- SOURCE heuristic 하나만으로 hard safety를 바꾸는 시도.

---

# 5. Goal / Route Kernel

## 5.1 boundary

4P:

```text
Boundary =
  <UP, UpGap, DOWN, DownBuffer, Leap>
```

3P도 같은 구조지만 상대 수가 2다.

모든 gap은 `ScoreTransition_R`로 계산한다.

## 5.2 route

```text
RouteMode ∈ {
  HOLD,
  CROSS_NOW,
  BUILD,
  FORCE_VALUE
}
```

```text
ResultNeed =
  <MinimumResult, SufficientResult, ForbiddenResult>
```

`BUILD`는 미래 배패를 상상하는 것이 아니라,
이번 국 뒤 점수상태가 다음 통상 점수대의 상방경로를 남기는지 본다.

## 5.3 HandRole

```text
PRESERVE  if HOLD
VALUE     if FORCE_VALUE
SPEED     if CROSS_NOW and smallest legal self-win already sufficient
BUILD     otherwise
```

Goal은 손패 결과를 보고 임의로 낮추지 않는다.

객관적 score/termination condition이 바뀌면 차분 갱신한다.

## 5.4 bounded route set / MustAct^ℛ

현재부터 종료까지 모든 미래를 전수탐색하지 않는다.
결정에 필요한 **검증된 질적 경로 최대 3개**만 유지한다.

```text
RouteCert ρ = <
  FirstActionRef,
  concrete/score-band result,
  leaf rank,
  terminal flag,
  assumptions,
  expiry
>

FirstActionRef =
  exact current legal compound action identity

```

```text
VerifyRoute_R(o_t,ρ) = TRUE
```

iff all are true:

```text
1. FirstActionRef(ρ) ∈ Legal_R(o_t)
2. every concrete current/kyoku transition in ρ
   equals ScoreTransition_R
3. leaf rank equals rank_R(leaf state)
4. terminal flag equals Termination_R(leaf state)
5. every future-dependent premise is listed in assumptions
6. no heuristic/source hypothesis is relabeled as RULE/HARD fact
7. any score/rank band is a sound enclosure of its represented concrete results
8. RuleProfile/version and all required premises are unexpired
```

`VerifyRoute_R`는:

```text
route is internally rule/assumption consistent
```

를 인증할 뿐:

```text
route will occur
```

를 인증하지 않는다.

```text
ℛ_t = {ρ | VerifyRoute_R(o_t,ρ)},   |ℛ_t|≤3
```

`leaf rank`가 곧 반장 최종순위라는 뜻은 아니다.
terminal이 아니면 국 종료 후 다시 계획한다.

현재 **정확한 compound action** `a`로 시작하는 상방경로:

```text
P_t^+(a) =
  {ρ∈ℛ_t |
    route improves reachable terminal-rank frontier
    ∧ FirstActionRef(ρ)=ActionID(a)
  }
```

```text
same action class
≠ same FirstActionRef
```

예를 들어 `DISCARD(3m)`과 `DISCARD(7p)`를
둘 다 `DISCARD`라는 이유로 같은 route first action으로 합치지 않는다.

행동 의도:

```text
Intent(a) ∈ {ATTACK, BALANCED, FOLD}
```

- `ATTACK`: 안전을 희생할 수 있는 상방진행.
- `BALANCED`: Goal/hand progression을 유지하면서 profile safety gate 통과.
- `FOLD`: 상방진행보다 현재 손실회피를 우선.

```text
MustAct_t^ℛ ⇔
   (∃a: Intent(a)=ATTACK ∧ P_t^+(a)≠∅)
   ∧
   (∀b: Intent(b)∈{BALANCED,FOLD},
        P_t^+(b)=∅)
```

의미는 오직:

> **현재 생성·검증한 ℛ 안에서는 더 높은 순위 경로가 공격 쪽에만 남는다.**

이다.

```text
MustAct^ℛ
⇏ actual globally optimal policy must attack
```

생성하지 않은 경로가 반례일 수 있다.

```text
current rank=1
→ no upward-rank route
→ MustAct^ℛ=false
```

단 현 순위 방어를 위한 진행행동은 별도 Goal/Policy에서 여전히 가능하다.

### 5.5 CounterRouteProbe — MustAct self-challenge

`MustAct^ℛ`는 생성경로 누락에 취약하므로
위협 아래 공격강제에 사용하기 전 **반대경로를 한 번 찾는다.**

```text
CounterRouteProbe:
  1. RefreshPathSet if a path trigger is dirty
  2. inspect aS / SafeProgress candidate
  3. inspect legal PASS/control/renchan-preserving route when applicable
  4. try to construct at most one upward/preserve RouteCert
     whose first action is non-ATTACK
```

출력:

```text
MustActGate ∈ {
  CONFIRMED_SCOPED,
  FALSE,
  UNRESOLVED
}
```

```text
CONFIRMED_SCOPED if
  MustAct^ℛ
  ∧ probe completed
  ∧ no non-ATTACK upward route was found

FALSE if
  ¬MustAct^ℛ
  or a valid counter-route was found

UNRESOLVED if
  budget/representation prevents the probe from completing
```

```text
CONFIRMED_SCOPED
⇏ globally optimal attack
```

여전히 현재 bounded route search에 상대적인 명제다.

`UNRESOLVED`를 `CONFIRMED`로 승격하지 않는다.
일반 controller는 ordinary route/threat policy로 돌아가고,
deadline이면 ReliabilityFallback이 residual을 남긴다.


---

# 6. Hand Kernel — Structural + Availability

## 6.1 active paths

인간은 최대 두 경로만 유지한다.

```text
PathSet ⊆ {
  STANDARD,
  CHIITOI,
  KOKUSHI,
  GOAL_YAKU
}
|PathSet|≤2
```

우선:
1. 현재 가장 가까운 경로.
2. VALUE에서 목표타점을 만들 수 있고 지나치게 멀지 않은 경로 하나.

후로하면 닫힌 특수경로를 합법성에 맞게 제거한다.

### PathCatalog / recovery

```text
PathCatalog_R =
  currently legal members of {
    STANDARD,
    CHIITOI,
    KOKUSHI,
    GOAL_YAKU
  }
```

`PathSet≤2`는 기억상한이지
처음 고른 두 경로만 영구고정한다는 뜻이 아니다.

```text
PathRefreshTrigger if any:
  SelfHand changed
  hard visible-count change kills an active path
  call/kan/kita changes legal path space
  RuleProfile changed
  PathWeak
  VALUE_ROUTE_BROKEN
  DEAD_TENPAI
```

trigger 때:

```text
RefreshPathSet:
  1. scan the finite PathCatalog_R
  2. discard rule-illegal / hard-dead paths
  3. choose nearest hard-feasible path
  4. if VALUE role:
       choose one additional target-capable path if useful
     else:
       choose one distinct viable alternative only when decision-relevant
  5. replace stale paths; never keep >2
```

```text
PathRecovery
≠ add a third persistent path
```

처음에 빠진 치또이/국사/목표역이
나중에 실제로 살아난 경우 다시 후보가 될 수 있다.

## 6.2 structural shanten

ordinary structural shanten은 보조값으로 유지한다.

```text
Shanten_struct
```

그러나 이것만 속도축의 정본으로 쓰지 않는다.

### 왜?

```text
tenpai with every winning tile hard-dead
```

는 구조상 `0`이어도 실제 공개정보에서 그 대기를 완성할 수 없다.

## 6.3 Rule/Availability-aware deficiency

기계형 의미:

```text
AvailDef_R(h, path) =
  현재 hand와 R에서
  hard capacity 제약을 위반하지 않는
  legal agari template까지 필요한
  최소 tile-change distance
```

completion template `g`가 tile `p`를 추가로 `need_g(p)`장 요구할 때:

```text
need_g(p) ≤ upper_t^R(p)
```

를 만족하지 못하면 그 template은 hard-dead다.

```text
AvailDef = min over hard-feasible templates
```

이는 "그 패가 벽에 있을 확률"이 아니다.
단지 **공개정보상 불가능한 경로를 최단경로로 세지 않는다.**

## 6.4 Human compile — live block shanten

인간은 agari template을 전수열거하지 않는다.

표준손의 incomplete block `b`:

```text
Comp_R(b) = 룰상 b를 한 장으로 완성하는 tile types
LiveComp_t(b) = {x∈Comp_R(b) | upper_t^R(x)>0}
```

```text
R2* : |LiveComp|≥2
R1* : |LiveComp|=1
D0  : |LiveComp|=0
```

중첩:

```text
X*:
  둘 이상의 decomposition에 참여
  ∧ 각 decomposition에 live successor 존재
```

5블록 재평가에서는 `D0` incomplete block을 유효 taatsu로 세지 않는다.

즉:

```text
dead shortest block
→ shanten tie-break에서 뒤로 미루는 수준이 아니라
→ live block count 자체에서 제외
```

텐파이:

```text
FinishSet(h) = {x | x completes legal win ∧ upper_t^R(x)>0}
```

```text
FinishSet=∅
→ DEAD_TENPAI / wait rebuild required
```

0샹텐이라는 숫자만으로 live 1샹텐을 자동으로 이기지 못한다.

## 6.5 special paths

### Chiitoi

기존 pair는 유지하되:
- singleton을 pair로 만들려면 `upper(type)>0`,
- 새 pair type을 만들려면 필요한 추가 copies가 capacity 안에 있어야 한다.

hard capacity상 7 distinct pairs가 불가능하면 path를 제거한다.

### Kokushi

모든 missing terminal/honor type에:

```text
upper(type)>0
```

가 필요하다.

pair requirement도 capacity를 만족해야 한다.

하나라도 hard-dead면 현재 kokushi path는 제거한다.

## 6.6 one-step successor dominance

단순히 `AdvanceSet(A)⊃AdvanceSet(B)`라고 해서 A의 다음 상태가 더 좋다고 단정하지 않는다.
같은 개선패를 뽑아도:
- 목표역,
- 타점,
- 다음 형태,
- 후리텐/대기

가 달라질 수 있기 때문이다.

```text
AdvanceSet_R(h) =
  {x |
    upper_t^R(x)>0
    ∧ draw x makes AvailDef decrease
  }
```

후보 `a`에서 tile `x`를 뽑았을 때의 **hard successor signature**:

```text
SuccHardSig(a,x) = <
  GoalAlive_after,
  YakuAlive_after,
  PathFeasible_after,
  TargetAlive_after_if_VALUE,
  AvailDef_after,
  MinimumResultStatus_after
>
```

```text
MinimumResultStatus ∈ {BELOW,MEETS}
```

다음 component order만 hard comparison에 쓴다.

```text
TRUE > FALSE
MEETS > BELOW
smaller AvailDef is better
```

```text
HardSuccNonWorse(A,B) ⇔
  A.GoalAlive ≥ B.GoalAlive
  ∧ A.YakuAlive ≥ B.YakuAlive
  ∧ A.PathFeasible ≥ B.PathFeasible
  ∧ [VALUE이면 A.TargetAlive ≥ B.TargetAlive]
  ∧ A.AvailDef ≤ B.AvailDef
  ∧ A.MinimumResultStatus ≥ B.MinimumResultStatus
```

shape quality나 초과타점 같은 heuristic은
이 hard comparator에 넣지 않는다.

```text
SuccessorDom(a,b) ⇔
  ∀x∈AdvanceSet(b):
      x∈AdvanceSet(a)
      ∧ HardSuccNonWorse(
           SuccHardSig(a,x),
           SuccHardSig(b,x)
         )
  ∧ (
       AdvanceSet(b)⊂AdvanceSet(a)
       ∨ 어떤 shared x에서 hard strict better
     )
```

즉 A가:
- B의 모든 1-step 진행 tile type을 받아들이고,
- 같은 tile을 받았을 때도 hard successor state가 더 나빠지지 않으며,
- 추가 경로나 strict 우위를 하나 이상 가질 때만

A를 B보다 위에 둔다.

```text
AdvanceSet superset alone
⇏ dominance
```

사람은 이를 매 후보에 계산하지 않는다.
최종 동일 `AvailDef` 2후보가 형태상 정말 갈릴 때만:

> "B의 모든 전진패가 A에도 통하고, 같은 전진패에서 A의 Goal/Yaku/거리/최소결과가 더 나쁘지 않은가?"

를 확인한다.

확신할 수 없으면 `INCOMPARABLE`로 두고 ShapeSig/fallback으로 간다.

---

# 7. Candidate Evaluation

각 실제 합법 타패/compound action `a`:

```text
Eval(a) = <
  Legal,
  GoalAlive,
  YakuAlive,
  PathFeasible,
  TargetAlive,
  AvailDef,
  RonAccess,
  FinishSet,
  TargetCoverage,
  AdvanceSet,
  SuccessorMap,
  ShapeSig,
  ValueFloor,
  DoraKept,
  SafetyHard,
  DefenseFeatures,
  SafeLeaseStock,
  ControlEffect
>
```

```text
ShapeSig = <R2*, X*, Head, R1*, D0>
```

## 7.0 field semantics

`Eval`의 각 필드는 다음 의미로 고정한다.

### GoalAlive

```text
GoalAlive(a)=TRUE
```

iff action `a`의 즉시 rule transition이:
- `ForbiddenResult`를 즉시 강제하지 않고,
- 현재 `ℛ_t`의 Goal-consistent route 또는 사전 봉인 fallback 하나 이상을 남긴다.

```text
GoalAlive ≠ eventual success guarantee
```

### YakuAlive

```text
YakuAlive(a)=TRUE
```

iff action 뒤:
- 현재 open/closed 상태,
- RuleProfile,
- active path

와 정합적인 legal yaku witness가 적어도 하나 남는다.

문전이면 legal riichi possibility가 witness가 될 수 있다.
"도라만"은 yaku witness가 아니다.

### PathFeasible

```text
PathFeasible(a)=TRUE
```

iff active path 중 적어도 하나가:
- rule-legal,
- hard tile capacity와 모순되지 않는
agari template을 가진다.

```text
not CountEliminated
⇏ actual likely
```

### TargetAlive

VALUE role에서:

```text
TargetAlive(a)=TRUE
```

iff hard-feasible path 중
`MinimumResult`를 충족할 수 있는 score-transition witness가 하나 이상 남는다.

다른 role에서는 `NA/TRUE-by-scope`.

### RonAccess

텐파이일 때:

```text
RonAccess ∈ {
  DEAD,
  TSUMO_ONLY,
  RON_OPEN
}
```

- `DEAD`: live legal finish가 없음.
- `TSUMO_ONLY`: finish는 있으나 현재 furiten/yaku rule 때문에 ron 불가.
- `RON_OPEN`: 적어도 하나의 legal ron finish가 있음.

텐파이 전 `NA`.

### FinishSet

```text
FinishSet(a) =
  {tile type x |
    upper_R(x)>0
    ∧ x completes a legal win
  }
```

후리텐 여부와 "패종이 물리적으로 live"는 분리한다.

중요:

```text
|FinishSet(A)| > |FinishSet(B)|
⇏ A universally better
```

같은 tile type을 기다려도
두 손의:
- yaku,
- fu/value,
- target result

가 다를 수 있기 때문이다.

따라서 `FinishSet`은:
- dead wait 판정,
- RonAccess,
- TargetCoverage,
- scoped `ShapeFallback`

의 입력 feature이지,
**개수나 단순 포함관계만으로 보편 지배를 만들지 않는다.**

### TargetCoverage

```text
NONE / SOME / ALL
```

hard-feasible finish/path outcomes 중
`MinimumResult` 충족 범위.

```text
ALL
```

은 현재 표현 안의 모든 hard-feasible 결과가 목표를 충족한다는 뜻이지
미래 모든 실제 게임결과를 보장하지 않는다.

### ValueFloor

```text
ValueFloor(a)
```

은 현재 rule state와 visible/guaranteed yaku·dora·fu에서 얻는
**확정 하한 score band**다.

기본 제외:
- future ura,
- future ippatsu event,
- unknown rinshan,
- future kan-dora,
- 아직 얻지 않은 draw-dependent yaku.

현재 이미 확정된 last-tile/yaku condition은 RuleProfile에 따라 포함 가능하다.

### DoraKept

action 뒤 남는 현재 확정 dora/red-dora resource.
구조·Goal·Safety의 hard축을 이기지 못하고 scoped tie-break용이다.

### SafetyHard

```text
SafetyHard(a) =
  <FactSafe_i(a) | i∈ActiveThreatSet>
```

즉시론 hard fact만 담는다.

### DefenseFeatures

9절의 비현물 구조 evidence.
확률이나 hard safety로 승격하지 않는다.

### SafeLeaseStock

9.5의 expiry-aware future hard-safe physical stock.

### ControlEffect

11.3의 exact public rule transition.
존재 자체가 utility-positive를 뜻하지 않는다.

## 7.1 pools

```text
FastPool =
  Legal ∧ GoalAlive ∧ YakuAlive ∧ PathFeasible

ValuePool =
  FastPool ∧ TargetAlive

SafePool =
  Legal
```

## 7.2 attack representative aF

```text
aF:
  min AvailDef
  ▷ RonAccess
  ▷ FinishSet dominance/coverage when tenpai
  ▷ SuccessorDom
  ▷ R2* ▷ X* ▷ Head ▷ R1* ▷ fewer D0
  ▷ ValueFloor ▷ DoraKept
  ▷ DefenseFallback_{Σ_R}
  ▷ fixed code
```

`SuccessorDom`이 성립하지 않으면 강제로 우열을 만들지 않고 다음 축으로 간다.

## 7.3 value representative aV

```text
aV:
  TargetAlive
  ▷ min AvailDef
  ▷ TargetCoverage
  ▷ RonAccess
  ▷ target-valid FinishSet
  ▷ ValueFloor
  ▷ SuccessorDom under target path
  ▷ ShapeSig
  ▷ DoraKept
  ▷ DefenseFallback_{Σ_R}
  ▷ fixed code
```

## 7.3.1 ShapeFallback boundary

```text
ShapeFallback_{Σ_R}
```

은:
- `FinishSet`,
- `R2*/R1*/X*/D0`,
- head structure,
- DoraKept

같은 **hard-derived features를 입력**으로 받지만,
그 feature들의 일반적인 승률순서를 hard theorem으로 주장하지 않는다.

```text
hard feature
≠ hard policy ordering
```

4P/3P와 state scope별로 calibration할 수 있다.

profile이 해당 비교를 정의하지 않으면:

```text
SHAPE_INCOMPARABLE
```

로 두고 다음 scoped tie-break/fixed code로 간다.

## 7.4 defense representative aS

9절의 safety partial order를 쓴다.

최종:

```text
C_t = unique({aF,aV,aS})
|C_t|≤3
```

---

# 8. Threat Model

각 상대 i의 observable record:

```text
Opp_i = <
  Declaration,
  OpenValue,
  PushStamp,
  Genbutsu,
  OneHyp
>
```

hard fact와 policy trigger를 분리한다.

## 8.1 active set

```text
ActiveThreatSet W_t =
  {i | ThreatPolicy.ActiveRule(Opp_i,G,current rule state)=ACTIVE}
```

리치:

```text
Riichi_i → i∈W_t
```

는 강한 기본 trigger다.

그 외:
- open meld count,
- visible value,
- push behavior

는 `StrategyProfile`의 policy trigger이며
텐파이 fact가 아니다.

```text
¬ACTIVE ⇏ noten
ACTIVE ⇏ tenpai
```

## 8.2 KeyThreat

```text
KeyThreat =
  ThreatPolicy.KeyPriority(W_t, G, public facts)
```

우선순위는 사전고정한 profile rule로 정한다.

4P legacy baseline은:

```text
riichi status
▷ dealer
▷ boundary-critical visible value
▷ earlier active declaration
▷ seat tie-code
```

이다.

이 순서도 방총확률 정리가 아니라
**어느 위협을 먼저 보호할지 정하는 policy**다.

3P에는 자동 복사하지 않고
`ThreatPolicy_3P.KeyPriority`를 별도 calibration한다.

## 8.3 ThreatClass

```text
ThreatClass ∈ {NONE, ACTIVE, HEAVY}
```

```text
NONE   if W_t=∅
HEAVY  if ThreatPolicy.HeavyRule(W_t,G,public facts)=TRUE
ACTIVE otherwise
```

4P legacy baseline:

```text
HEAVY if
  |W_t|≥2
  or dealer riichi
  or one exact/visible active threat's deal-in
     can immediately destroy the current certified route
```

"destroy route"는 정확한 공개 score lower-bound로 판정할 수 있을 때만 쓴다.

3P `ThreatPolicy.HeavyRule`는 별도 profile이다.

## 8.4 HandState / PathWeak

```text
HandState =
  READY, if live legal tenpai
  LIVE,  if PathFeasible and AvailDef is near/currently progressing
  WEAK,  otherwise
```

"near"의 세부 human cutoff는 `StrategyProfile`이지만
dead structural tenpai를 READY로 부르지 않는다.

```text
PathWeak ⇔
  ¬PathFeasible
  or no Goal-consistent current route certificate
```


---

# 9. Safety Kernel — hard fact와 heuristic evidence 분리

## 9.1 fact-safe

상대 i:

```text
Genbutsu_i =
  own river types
  ∪ valid passed types after riichi
  ∪ rule-valid temporary-furiten types during expiry
```

```text
FactSafe_i(x)
→ immediate ron by i impossible
```

복수위협:

```text
CommonFactSafe(x)
⇔ ∀i∈W, FactSafe_i(x)
```

## 9.2 문제: evidence-family count

기존:

```text
SUPPORTED_2 > SUPPORTED_1 > UNKNOWN
```

는 suji·tile-count blocker·pair blocker처럼 서로 다른 의미의 근거를
"몇 계열인가"로 축약한다.

```text
2 evidence families ⇏ universally lower deal-in risk than 1 family
```

따라서 **근거 개수 자체를 safety scalar로 사용하지 않는다.**

## 9.3 Defense feature envelope

현물이 아닌 후보 x에 대해
공개정보로 제거된/남은 구조적 론 메커니즘을 feature로 만들 수 있다.

```text
DefenseFeatures_i(x) = <
  RyanmenElim,
  KanchanElim,
  PenchanElim,
  ShanponElim,
  TankiConstraint,
  ChiitoiConstraint,
  CountBlockers,
  RouteConstraints
>
```

이 feature는:
- hard impossibility가 생겼는지,
- 어떤 구조계열이 줄었는지

를 보여주지만:

```text
DefenseFeatures ≠ deal-in probability
```

이다.

두 feature의 "항목 수"를 세어 안전서열을 만들지 않는다.

### hard-dominance boundary

`FactSafe`가 아닌 두 패의 실제 위험우위를
공개 hard fact만으로 증명할 수 없는 경우가 대부분이다.

따라서:

```text
non-FactSafe defense ordering
→ DefenseFallback_{Σ_R}
```

로 보내고,
`DefenseFeatures`는 그 profile의 입력 feature다.

즉 H23 core는:
- suji가 몇 개,
- 벽이 몇 개

를 보편 안전정리로 승격하지 않는다.

offline calibration이 특정 state scope에서 순서를 지지하면
그 scope의 `DefenseFallback`에 compile한다.

## 9.4 safety certificate expiry

즉시 안전과 미래 안전을 분리한다.

```text
SafeCert_i(x) = <
  Basis,
  ValidNow,
  ExpiryEvent
>
```

예:
- 상대 자기 river의 x: 현 rule에서 해당 wait이면 furiten이므로 장기 hard basis.
- riichi 후 x를 통과시킨 경우: locked-wait/riichi-furiten rule에 따라 지속.
- 비리치 상대가 방금 x를 통과시켜 생긴 temporary furiten: 다음 draw에서 만료 가능.

따라서:

```text
SafeNow(x)
⇏ SafeAtNextSelfDiscard(x)
```

## 9.5 SafeLeaseStock — NextSafe 수리

자기 손의 물리패 수는 완전히 관측 가능하다.

후보 action `a` 뒤 남은 패 중,
**다음 내 강제 타패시점까지 hard-safe certificate가 살아 있음이 보장되는**
physical copies만 센다.

```text
SafeLeaseStock_W(a) =
  count {
    physical tile d in self hand after a |
    ∀i∈W:
      SafeCert_i(type(d)).Expiry
      survives until next-self-discard horizon
  }
```

인간표시:

```text
0 / 1 / 2+
```

temporary-furiten처럼 그 전에 만료할 수 있는 것은
미래 stock에 세지 않는다.

이는:

```text
future guaranteed safe-turn stock
```

의 하한이지
향후 모든 공격에 대한 안전보장은 아니다.

## 9.6 aS

보편 hard 우선은 하나만 둔다.

```text
CommonFactSafe(a)
```

이면 모든 현재 활성위협에 대해 즉시론 불가가 증명된다.

따라서:

```text
if any CommonFactSafe candidate exists:
    aS is chosen only from that subset
```

그 subset 안에서는:

```text
SafeLeaseStock
▷ hand preservation
▷ scoped Shape/Defense fallback
▷ fixed code
```

를 쓴다.

공통현물이 없으면:
- `KeyThreat` 현물 여부,
- 상대별 `SafetyHard` vector,
- `DefenseFeatures`,
- SafeLeaseStock,
- hand preservation

을 모두:

```text
DefenseFallback_{Σ_R}
```

의 입력으로 넘긴다.

즉:

```text
KeyThreatFactSafe
≠ globally hard-safer-than every nonfact tile
```

이다.

KeyThreat는 **policy attention/priority**이지
다른 상대의 론 가능성을 삭제하지 않는다.

`DefenseFallback`은:
- 4P/3P 별도,
- state scope 별도,
- outcome 전에 봉인,
- 실증 전 heuristic

이다.

profile이 비교를 정의하지 못하면
현재 fixed defense code로 종료하고
`DEFENSE_INCOMPARABLE` residual을 남길 수 있다.


---

# 10. A/B/D Controller

```text
πA:
  VALUE and aV exists → aV
  otherwise aF if exists
  otherwise aS
```

VALUE route가 깨졌으면:
- MustAct이면 살아 있는 aF를 위험표지와 함께,
- 아니면 aS + ROUTE_REBUILD.

```text
πD:
  aS
```

```text
πB:
  Goal/Yaku/Path live
  ∧ AvailDef non-worsening
  ∧ current discard passes
    profile-approved safety condition for every active threat
  → best non-dominated progress candidate
  else MustActGate=CONFIRMED_SCOPED ? πA : πD
```

`profile-approved safety condition`은
FactSafe 또는 scoped `DefenseFallback_{Σ_R}`에서 온다.
그 profile은 확률정리로 승격하지 않는다.

## 10.0 SafeProgress

```text
SafeProgress(a) ⇔
  GoalAlive(a)
  ∧ YakuAlive(a)
  ∧ PathFeasible(a)
  ∧ AvailDef(a) does not worsen under current role
  ∧ ∀i∈W_t:
       DefenseFallback.SafetyGate(i,a)=PASS
```

`DefenseFallback.SafetyGate`는:
- FactSafe,
- 또는 그 state scope에서 허가된 calibrated DefenseFallback

만 쓴다.

```text
UNKNOWN safety gate
≠ PASS
```

MustAct^ℛ가 아닌데 SafeProgress가 없으면
공격을 억지 승인하지 않는다.

## controller

```text
no threat:
  VALUE → aV
  SPEED → aF
  PRESERVE/BUILD → non-dominated aF/aS comparison

threat:
  PathWeak or HEAVY and not MustAct^ℛ → aS
  SafeProgress exists                → πB
  MustActGate=CONFIRMED_SCOPED and no SafeProgress → aF/aV + explicit risk
  otherwise                          → aS
```

---

# 11. Win / Call / Riichi / Kan / Kita

## 11.1 win

legal RON/TSUMO가 있으면 타패보다 먼저 본다.

기본:

```text
no strict certificate → WIN
```

### PassCert

화료 `y`를 거부하는 인증은 다음 최소형만 허용한다.

```text
PassCert_t(y) = <
  PASS terminal-rank enclosure,
  WIN(y) terminal-rank enclosure,
  rule / score / furiten assumptions,
  expiry
>
```

필요조건:

```text
Sound(PASS_enclosure)
∧ Sound(WIN_enclosure)
∧ every rank in PASS_enclosure
    strictly better than
    every rank in WIN_enclosure
```

즉:

```text
maxRankQuality(WIN)
<
minRankQuality(PASS)
```

에 해당하는 **엄격한 전칭 terminal dominance**가 있어야 한다.

```text
best-case PASS
≠ PassCert
```

```text
same best rank
≠ PassCert
```

```text
uncertain future upside
≠ PassCert
```

furiten/termination/rule state가 바뀌면 즉시 만료한다.

인증이 없거나 soundness가 불명확하면:

```text
WIN
```

을 선택한다.

## 11.2 chi/pon

```text
CALL = <call, mandatory_discard>
```

다음을 모두 통과:
1. legal in R,
2. YakuAlive,
3. GoalAlive,
4. VALUE면 TargetAlive,
5. AvailDef 개선 또는 목표필수 yaku/value 확정,
6. 직후타패의 safety를 숨기지 않음,
7. closed-hand rights loss 뒤에도 PASS보다 현재 profile ordering에서 우월.

πD에서는 비화료 call을 기본 금지.

## 11.3 control call / exact public-state effect

후로가 자기 손을 전진시키지 않아도
룰상 **정확한 외부 상태변화**를 만들 수 있다.

예:
- rule-profile상 ippatsu 소멸,
- 마지막패 draw order / haitei-houtei ownership 변화,
- 특정 상대 nagashi path의 hard destruction,
- 연장/종료에 직접 연결되는 공개 상태변화.

따라서 call을 공격진행만으로 완전히 필터링하지 않는다.

```text
ControlEffect(a) = exact rule-public transition caused by action a
```

```text
ControlCallEligible(a) ⇔
  a is legal call
  ∧ ControlEffect(a) is decision-relevant to G
```

그러나:

```text
exact effect exists
⇏ action is beneficial
```

강제 타패·문전상실·역/타점·안전비용이 있기 때문이다.

### ControlCert

다음이 hard하게 보이면 core가 직접 승인할 수 있다.

```text
ControlCert(a, PASS) ⇔
  exact desirable ControlEffect
  ∧ Goal/Yaku/Target/AvailDef/Safety hard axes
    are no worse than PASS
  ∧ one axis strict better
```

이 정도의 dominance가 없으면:

```text
ControlCallPolicy_{Σ_R}
```

로 보내며 offline calibration 대상이다.

control-only call은 새 4번째 상주후보를 만들지 않는다.

```text
offensive/value call → aF/aV pool
defensive/control call → optional-response aS/control slot
```

### optional abortive draw

룰이 선택적 도중유국 선언을 허용하면:

```text
AbortiveAction ∈ Legal_R
```

을 독립행동으로 본다.

```text
AbortiveTransition_R
```

으로:
- 점수,
- 친 연장,
- 본장/리치봉,
- 다음 국,
- 종료조건

을 정확히 계산한다.

```text
AbortiveDomCert
```

가 continue route를 hard하게 지배하면 선언,
반대면 계속한다.

둘 다 아니면:

```text
AbortivePolicy_{Σ_R}
```

가 정한다.

자동 발생하는 유국/사건은 action이 아니라 rule transition이다.

## 11.4 riichi/dama

hard branches:

```text
if riichi is required for legal yaku
  → RIICHI

if MinimumResult requires riichi han
  → RIICHI

if 1000-point payment itself kills GoalAlive
  → NO_RIICHI
```

그 외:
- hand lock,
- hidden wait,
- current dama value,
- threat,
- wait-change,
- route

trade-off는:

```text
RiichiPolicy_{Σ_R}
```

에서 결정한다.

기존 H9 3~8번째 가지는:

```text
Σ_4P legacy baseline
```

로 보존하되 **보편 정리로 취급하지 않는다.**

3P에는 자동 복사하지 않는다.

## 11.5 kan

모든 kan은:
- legal,
- current goal/yaku/path non-destruction,
- current liability,
- post-riichi wait rule

을 먼저 검사한다.

unknown rinshan / new dora는
선택 전에 guaranteed value로 넣지 않는다.

세부 선택은 `KanPolicy_{Σ_R}`.

## 11.6 Kita / Nuki — 3P rule-dependent

```text
KITA ∈ Legal_R
```

일 때만 후보.

`KitaSpec_R`가 명시할 것:
- north extraction allowed?
- dora value?
- replacement draw?
- closed status?
- ippatsu interaction?
- robbing/liability?
- furiten / kan-like effects?
- timing?

KITA는 discard가 아니라:

```text
KITA
→ guaranteed rule transition
→ replacement draw if rule says so
→ new decision
```

이다.

### KitaFreeCert

```text
KitaFreeCert(N) ⇔
  KITA legal
  ∧ removing N does not worsen
       GoalAlive/YakuAlive/AvailDef/TargetAlive
  ∧ active threat under SafeLeaseStock-critical state가 아님
  ∧ no profile-specific immediate liability
```

이 cert는 **내재 구조비용이 현재 hard축에서 검출되지 않았다**는 뜻뿐이다.

```text
KitaFreeCert
⇏ KITA globally better
```

공개신호·draw-order·상대반응·룰별 liability가 남을 수 있으므로
실제 선택은 항상 `KitaPolicy_3P`에서 한다.
cert가 없으면 더 보수적인 branch를 사용한다.

```text
KITA always good
```

도,

```text
KITA is just discard
```

도 하드코딩하지 않는다.

---

# 12. 3P Human Runtime Profile

3P는 4P의 공격성 계수 변경이 아니다.

## 12.1 hard differences

```text
PlayerCount=3
TileCapacity = R-specific
ScoreTransition = R-specific
TsumoPayment = R-specific
ChiAllowed = R-specific
KitaRules = R-specific
Round/termination = R-specific
```

현재 공개된 3P rulesets 사이에도
Kita·tsumo payment·honba·termination이 다를 수 있으므로
profile field가 빠지면 `RULE_PROFILE_INCOMPLETE`.

## 12.2 local sequence inference

H11의 숫자 ±2를 그대로 쓰지 않는다.

```text
Local_R(x) =
  x와 함께 R에서 legal sequence block을 만들 수 있는
  tile types의 local closure
```

예:
- 제거된 tile capacity=0인 sequence는 route evidence가 아니다.
- sequence가 룰상 불가능한 honor는 honor-only.

## 12.3 opponent vector

상대는 두 명뿐이다.

```text
SafetyHard / DefenseFeatures opponent dimension = 2
```

둘 다 ACTIVE면 multi-threat.

하지만 4P에서 `2 open melds`를 ACTIVE로 둔 threshold가
3P에서도 최적이라는 주장은 하지 않는다.

## 12.4 shared human card

카드는 그대로 다섯 장:

```text
G — GOAL
H — HAND
T — THREAT
S — SAFETY
C — CHOICE
```

새 3P 카드 없음.

C에 필요할 때만:

```text
KITA?
```

한 비트가 추가된다.

---

# 13. SOURCE Module — optional attention layer

## 13.1 hard facts

보존:
- TSUMOGIRI hand multiset unchanged,
- TEDASHI means discarded tile was in previous hand and unknown draw retained,
- same-tile copy swap means TEDASHI need not change multiset,
- riichi declaration source facts,
- post-riichi forced discard policy facts under R,
- capacity blocker.

모든 `upper`는:

```text
upper_t^R
```

를 쓴다.

## 13.2 soft source inference

```text
MeaningfulHD
WATCH
FOCUS
```

는 hard risk verdict가 아니다.

```text
FOCUS
→ QueryPriority=HIGH
```

만 만든다.

## 13.3 trigger mount

```text
Mount_SOURCE ⇔
  source visible
  ∧ active threat
  ∧ final candidates still unresolved
  ∧ source probe can still change a hard/public axis
```

## 13.4 SourceProbe

확인:
1. Genbutsu / temporary furiten,
2. `upper_R=0` blocker,
3. visible yaku/meld contradiction,
4. riichi source hard fact,
5. current DefenseFeatures / hand viability.

```text
HARD_CHANGED
→ affected core axis recompute

NO_HARD_CHANGE
→ core order 그대로

UNRESOLVED
→ core order + residual
```

```text
FOCUS alone never changes discard order.
```

---

# 14. SIGNAL Module — optional

기본:

```text
SIGNAL=OFF
```

H9/H23 baseline action이 진짜 리치·후로·visible value signal이면
그 signal effect는 공짜 부가효과로 존재한다.

### E0

룰상 동일한 물리복사본 선택으로
내 private hand/state/value/safety가 완전히 같고
공개 source만 달라지는 경우:

```text
E0 COPY_SOURCE
```

만 **후보**로 둘 수 있다.

그러나:

```text
E0 intrinsic equivalence
⇏ same opponent continuation
⇏ same game outcome
```

이므로 실제 override에는:

```text
SignalGuard_{Σ_R} = PASS
```

가 필요하다.

최소 fail-closed:

```text
SignalGuard=BLOCK if
  Policy=πD
  or HandRole=PRESERVE/HOLD
  or multi-responder/crossfire state is decision-relevant
  or current public signal의 상대반응 효과를 profile이 허가하지 않음
```

```text
SignalGuard=UNKNOWN → baseline
```

즉 E0는 "공짜 이득"이 아니라
**내재 hand state가 같은 공개신호 선택지**일 뿐이다.

precommitted randomization은 guard를 통과한 경우에만 사용한다.

### E1

`COLOR_STORY` 같은 상대반응 exploit은:

```text
EXPERIMENTAL_OFF_BY_DEFAULT
```

offline held-out validation 후
specific `StrategyProfile`에서만 enable.

### E2

goal/yaku/shanten/value/safety를 깎아 위협을 연출:

```text
FORBIDDEN
```

---

# 15. Offline Calibration Compiler

## 15.1 왜 필요한가

다음은 hard theorem이 아니다.

```text
Shape fallback order
Defense fallback
OpenMeld threat threshold
Riichi/Dama discretionary branch
Call/Kan/Kita discretionary policy
SOURCE FOCUS
SIGNAL E1
```

새 rule을 더 쓰기 전에 측정한다.

## 15.2 Calibration criterion seal

결과를 본 뒤 유리한 metric/scope를 고르지 않는다.

```text
CalibrationSeal = <
  rule profile,
  target heuristic,
  strata,
  primary rank-CDF axes,
  noninferiority margins,
  sample/seed budget,
  continuation policy,
  stopping rule,
  promotion rule
>
```

```text
Commit(CalibrationSeal)
before held-out outcomes
```

data partition:

```text
development
→ known red-team
→ holdout
→ fresh reserve
```

반복해서 본 holdout은 더 이상 holdout이 아니다.

## 15.3 paired evaluation

같은 observed state에서:
- baseline action,
- candidate action

을 가능한 한 같은 continuation seed / opponent policy 조건에서 비교.

한 번의 실제 결과를 원인증거로 쓰지 않는다.

## 15.4 metrics

4P:

```text
P(rank≤1)
P(rank≤2)
P(rank≤3)
```

3P:

```text
P(rank≤1)
P(rank≤2)
```

plus:
- legality regression,
- decision latency,
- human action complexity,
- per-stratum errors.

## 15.5 stratification

```text
4P / 3P
rule profile
HandRole
AvailDef
Threat count
dealer
early/mid/late
riichi/open/closed
score boundary class
```

한 strata에서만 좋아지면 scoped policy로 저장.

## 15.6 promotion

```text
heuristic
→ development logs/seeds
→ held-out
→ fresh red-team
→ human complexity audit
→ scoped StrategyProfile
```

새 feature가 들어오면:
- 무엇을 대체/삭제하는지
- active card가 늘어나는지

를 함께 기록한다.

---

# 15.7 Reliability Shell — deadline / reset / emergency

H23은 H5/H8의 고장격리·시간초과 장점을 compact runtime에 복구한다.
이 층은 **여섯 번째 인간 카드가 아니라 실행기 sentinel**이다.

## 15.7.1 Decision mode

```text
Mode_t ∈ {
  ROUTINE,
  CRITICAL,
  EMERGENCY
}
```

```text
ROUTINE if
  no new major threat
  ∧ route/profile stable
  ∧ no hard conflict
  ∧ current representative can be resolved inside routine budget

CRITICAL if any:
  new riichi / strong open threat / multi-threat
  terminal-rank boundary fork
  live tenpai / dead-tenpai rebuild
  route break or VALUE_ROUTE_BROKEN
  rule-special action {RIICHI,KAN,KITA,CONTROL_CALL,ABORTIVE}
  SOURCE hard-probe trigger
  RuleProfile / StrategyProfile version change
  FactReset recovery

EMERGENCY if
  MandatoryAction
  ∧ (
       deadline reached
       or normal decision graph unresolved
       or hard conflict cannot be isolated in time
       or required representation/state is unavailable
     )
```

선택응답에서 `PASS`가 합법이면 불확실성을 이유로
억지 EMERGENCY action을 만들지 않는다.

## 15.7.2 Resource budget

외부 deadline을 hard constraint로 둔다.

```text
DecisionBudget = <
  deadline,
  routine_budget,
  critical_budget,
  max_source_probe=1,
  max_representation_refine=1
>
```

우선순위:

```text
LEGALITY / LEGAL_WIN
> FACT CONSISTENCY
> TERMINAL GOAL
> CURRENT HARD SAFETY
> BASE REPRESENTATIVES
> OPTIONAL SOURCE/SIGNAL DETAIL
```

```text
budget exhausted
→ optional detail stop
→ if decision resolved: return
→ else ReliabilityFallback
```

완전동률을 없애려고 계산을 무한 연장하지 않는다.

## 15.7.3 Dirty dependency graph

항상 전체를 재계산하지 않는다.

```text
DepRoot = {
  R,
  ScoreState,
  KyokuState,
  SelfHand,
  VisibleInstanceLedger,
  DeclarationState,
  ThreatSet,
  SafetyExpiry,
  RouteSet,
  StrategyProfileVersion,
  SourceState
}
```

```text
Dirty(X) =
  dependents forward-reachable from changed root X
```

대표 무효화:

```text
new draw/discard/call
  → SelfHand/VisibleInstanceLedger
  → live blocks / AvailDef / PathRefreshTrigger / candidate reps

new riichi/call/kan/kita
  → Declaration/Threat/Safety/Source
  → controller / special resolver

score/honba/kyotaku/dealer/round change
  → Goal/Boundary/Route

RuleProfile change
  → all rule-conditional legality/count/score facts

temporary-furiten expiry
  → affected SafeCert / SafeLeaseStock only

StrategyProfile version change
  → policy outputs only; hard world facts unchanged
```

```text
HistoricalCache
≠ CurrentApplicability
```

## 15.7.4 Fact consistency sentinel

다음 중 하나면 normal inference를 중지한다.

```text
FactConflict if:
  KnownToSelf(p) > Cap_R(p)
  or impossible hand-size/event transition
  or score/kyoku state contradicts authoritative room state
  or mutually incompatible rule/profile versions are active
  or a hard fact and its negation are both live
```

```text
HardConflict
→ quarantine minimal known conflict
→ invalidate downstream derived fields
→ FactReset
```

모순에서 unrestricted explosion을 쓰지 않는다.

## 15.7.5 FactReset

```text
FactReset:
  1. discard soft OneHyp / FOCUS / SIGNAL reaction hypotheses
  2. expire temporary safety whose lifetime is uncertain
  3. reload RuleProfile + StrategyProfile version
  4. re-read score / round / honba / kyotaku / dealer / termination
  5. re-read self hand / open meld / riichi / furiten
  6. re-read public riichi / calls / kans / kita / dora / rivers
  7. rebuild current Genbutsu and VisibleInstanceLedger
  8. rebuild one minimal Goal/Route view and current H/T/S
  9. enter CRITICAL
 10. if mandatory deadline arrives before normal resolution:
       ReliabilityFallback
```

```text
Reset
≠ failure
```

오염 확산을 막는 복구다.

## 15.7.6 Action status / residual

행동의 절차상 상태와 타패 안전도를 분리한다.

```text
ActionStatus ∈ {
  NORMAL,
  SCOPED,
  EMERGENCY
}
```

```text
Residual = <
  GoalResidual?,
  ThreatSafetyResidual?,
  ModelOrSourceResidual?
>
```

각 칸은 현재 행동을 실제로 바꿀 수 있는 미해결사항 하나만 둔다.
그 이상은 `MANY`로 묶고 실전 추론을 중단한다.

```text
ActionStatus=NORMAL
⇏ FACT_SAFE

ActionStatus=EMERGENCY
⇏ minimum deal-in risk
```

## 15.7.7 ReliabilityFallback

먼저 화료분기는 정상 규칙대로 처리한다.

```text
if LegalWin:
    strict PassCert ? PASS : WIN
```

선택응답:

```text
if PASS∈Legal
   and no already-validated strict override exists:
      return PASS
```

강제 타패에서는 반드시 합법 패 하나를 반환한다.

```text
EmergencyDiscard:

  E0 = legal discards that are CommonFactSafe now

  if E0≠∅:
      choose by
        SafeLeaseStock
        ▷ already-known hand preservation
        ▷ fixed code
      status = EMERGENCY

  else if MustActGate=CONFIRMED_SCOPED
          and an already-computed GoalAlive role candidate exists
          and its hard legality/route certificate is still fresh:
      choose that candidate
      status = EMERGENCY
      residual includes UNSAFE_OR_UNRESOLVED

  else:
      E1 = legal discards FactSafe to current KeyThreat

      if E1≠∅:
          // KeyThreat is only a fallback attention policy here
          choose by
            current cross-threat DefenseFallback if already available
            ▷ SafeLeaseStock
            ▷ fixed code
      else:
          choose
            already-computed DefenseFallback maximum if available
            else fixed legal discard code

      status = EMERGENCY
      residual includes NO_COMMON_FACT_SAFE when applicable
```

마지막 fixed-code branch는 **합법성만 보증**한다.
`최소방총`, `안전`, `최적`이라고 부르지 않는다.

## 15.7.8 Authorization expiry

```text
Auth(action, state_version)
```

은 현재 관측상태 한 버전에만 유효하다.

다음이 오면 관련 승인을 만료한다.

```text
draw / discard / call / riichi / kan / kita
dora change
score/round transition
temporary-furiten expiry
RuleProfile or StrategyProfile version change
```

```text
OldAuth
≠ permission for new state
```

## 15.7.9 Output contract

```text
Decision = <
  action,
  ActionStatus,
  SafetyHard,
  Residual,
  RuleProfileVersion,
  StrategyProfileVersion
>
```

대국 중 사람은 `action`과 필요할 때 작은 residual만 보면 된다.
상세 audit record는 복기용이다.

---

# 16. Human One-Turn Runtime

```text
0. 화료?    strict PASS certificate 없으면 WIN.
1. 룰?      4P/3P RuleProfile과 현재 score transition.
2. 목표?    SPEED / VALUE / PRESERVE / BUILD.
3. 위협?    ACTIVE / HEAVY + 공통현물.
4. 손?      역≤2 + AvailDef/live blocks.
5. 세 장?   aF / aV / aS.
6. 정책?    공격 / 안전진행 / 수비.
7. 동률?    AdvanceSet 포함관계 → 필요하면 SOURCE hard-probe.
8. 특수?    RIICHI/KAN/KITA/도중유국/control-call은 profile gate.
9. 실행.    SIGNAL은 E0 외 기본 OFF.
```

짧은 암기:

> **샹텐 숫자보다 살아 있는 경로를 본다.  
> 근거 개수를 위험확률로 바꾸지 않는다.  
> 안전패는 종류보다 남은 안전 턴도 본다.  
> 3마는 별도 룰·전략 프로필이다.  
> 출처읽기는 확인순서고, 블러프는 기본 꺼둔다.**

---

# 16.1 SpecialActionResolver — bounded streaming

특수행동도 HUMAN-RT 후보상한을 우회하지 못한다.

먼저 class별로 **대표 하나 이하**만 만든다.

```text
SpecialClass ∈ {
  COMMIT,      // RIICHI
  TRANSFORM,   // KAN / KITA
  CONTROL      // CONTROL_CALL / ABORTIVE_DRAW
}
```

```text
CommitRep:
  legal RIICHI actions 중
  current HandRole의 aF/aV ordering으로 최대 1개

TransformRep:
  legal KAN/KITA actions 중
  hard gates 통과 후 `KanPolicy/KitaPolicy + SpecialPolicy.TransformOrder`로 최대 1개

ControlRep:
  legal CONTROL_CALL/ABORTIVE actions 중
  exact ControlEffect와 `ControlCallPolicy/AbortivePolicy + SpecialPolicy.ControlOrder`로 최대 1개
```

```text
SpecialRepSet ⊆ {
  CommitRep,
  TransformRep,
  ControlRep
}
```

하지만 세 대표를 base와 한꺼번에 4후보로 보여주지 않는다.

## streaming resolution

`StrategyProfile`은 outcome을 보기 전에:

```text
SpecialPolicy.ClassOrder
```

를 봉인한다.

```text
ResolveSpecial(base):

  incumbent ← base

  for class in SpecialPolicy.ClassOrder:
      challenger ← representative(class)

      if challenger absent:
          continue

      compare incumbent vs challenger using:
        Legal
        → hard Goal/Yaku/Target/AvailDef/Safety
        → exact ControlEffect where applicable
        → scoped class policy

      if challenger hard-dominates:
          incumbent ← challenger

      else if incumbent hard-dominates:
          keep incumbent

      else:
          use precommitted scoped StrategyProfile comparison
          if comparison unresolved:
              keep incumbent
              + residual SPECIAL_INCOMPARABLE

  return incumbent
```

따라서 사람에게 동시에 열리는 비교는 항상:

```text
incumbent vs challenger
```

두 개 이하이고,
일반 `aF/aV/aS≤3` 상한을 깨지 않는다.

```text
SpecialPolicy.ClassOrder
```

는 결과를 본 뒤 바꾸지 않는다.

### integration rule

- `CALL+mandatory_discard`의 일반 속도/가치효과는 ordinary action scan에서 이미 평가한다.
- 그 call의 **control-only value**만 `ControlRep`에서 추가 검문한다.
- RIICHI의 타패 자체는 `CommitRep`를 만들 때 기존 aF/aV ordering을 재사용한다.
- KAN/KITA replacement draw는 보지 않은 결과를 선반영하지 않는다.

```text
SpecialActionExists
⇏ OverrideBase
```


# 17. Total Action Function

```text
HumanRiichi_H23(o_t, DecisionBudget):

  ingest newest authoritative public event
  mark Dirty dependencies
  expire stale Auth / SafeCert / source stamps

  if FactConflict:
      FactReset()

  R ← validate RuleProfile
  Σ ← validate StrategyProfile
  Lmask ← authoritative legal mask if platform supplies one

  if R incomplete:
      if Lmask exists:
          Legal ← Lmask
          return legality-only ReliabilityFallback
                 with EMERGENCY / RULE_MODEL_INCOMPLETE
      else:
          return LEGALITY_UNRESOLVED
          // normal match play must load R before this state

  update only Dirty hard state:
      legality
      score/terminal state
      visible counts / Cap_R
      self hand
      declarations / genbutsu / safety expiry

  Legal ← Legal_R(o_t)

  if |Legal|=1:
      return the sole legal action

  if legal WIN exists:
      if strict terminal PassCert:
          return PASS
      return WIN

  Mode ← ROUTINE or CRITICAL from current triggers

  if deadline insufficient for normal resolution:
      goto ReliabilityFallback

  G ← build/refresh terminal-rank route ℛ_t, |ℛ_t|≤3

  H ← RefreshPathSet when triggered
       keep active paths≤2
       remove hard-dead paths
       compute AvailDef/live-block approximation

  W ← ActiveThreatSet from ThreatPolicy.ActiveRule
  KeyThreat ← ThreatPolicy.KeyPriority
  T ← ThreatClass
  S ← FactSafe + DefenseFeatures + SafeLeaseStock

  evaluate legal ordinary/compound actions
  stop optional refinements when first policy axis resolves decision

  aF ← attack representative if defined
  aV ← target-value representative if defined
  aS ← defense representative

  C ← unique({aF,aV,aS})

  if MustAct^ℛ would force attack under threat:
      MustActGate ← CounterRouteProbe within critical budget
  else:
      MustActGate ← FALSE

  base ← A/B/D controller(C,G,H,T,S,MustActGate)

  if SOURCE trigger
     and budget remains
     and source hard-probe can still change a core axis:
       run at most one SourceProbe
       if HARD_CHANGED:
           recompute only Dirty affected fields and base

  build at most one rep for each triggered SpecialClass
  a1 ← ResolveSpecial(base)

  if E0-equivalent public-signal alternative exists
     and SignalGuard_{Σ_R}=PASS
     and precommitted randomizer available
     and budget remains:
       may choose equivalent signal variant
  else:
       keep a1

  if normal action fully resolved:
      return <
        a1,
        NORMAL or SCOPED,
        SafetyHard(a1),
        Residual,
        R.version,
        Σ.version
      >

ReliabilityFallback:

  if optional state and PASS∈Legal
     and no fresh strict validated override:
       return PASS with EMERGENCY status

  if mandatory discard:
       return EmergencyDiscard

  otherwise:
       return fixed legal fallback action
       with EMERGENCY / UNVERIFIED residual
```

### Totality boundary

정상적으로 사전 load된 룰프로필에서:

```text
DecisionState
→ exactly one legal action
```

을 목표로 한다.

프로필/입력 자체가 모순되면:
- authoritative legal mask가 있으면 그 mask 안에서 legality-only emergency fallback,
- 없으면 `LEGALITY_UNRESOLVED`.

즉 룰도 legal mask도 없는데 합법성을 꾸며내지 않는다.

```text
Fallback legality
≠ strategic optimality
```

---


---

# 18. Bounded Recursive Red-Team

## Epoch 1 — H12 retained repairs

발견:
- ordinal terminal order vs hidden expected utility,
- dead local completion in R2/X,
- FOCUS direct risk authority.

수리:
- RankDom,
- live shape,
- FOCUS→query prior.

**KEPT**

## Epoch 2 — dead tenpai / dead shortest-route attack

반례:

```text
candidate A:
  structural shanten=0
  every winning type upper=0

candidate B:
  structural shanten=1
  live advance path exists
```

기존 lexicographic:

```text
Shanten first
```

이면 A가 먼저 선택될 수 있다.

수리:

```text
AvailDef_R
+ D0 not counted as live taatsu
+ DEAD_TENPAI rebuild
```

결과:

```text
STRUCTURAL_SHANTEN_NO_LONGER_SOVEREIGN
```

**PASS_SCOPED**

## Epoch 3 — safety evidence scalarization attack

반례:

```text
candidate x:
  2 evidence families
candidate y:
  1 evidence family that eliminates a broader actual wait mechanism set
```

`SUPPORTED_2 > SUPPORTED_1`은 보편적으로 정당화되지 않는다.

수리:

```text
evidence count removed from core order
→ (intermediate RonEnvelope idea rejected later)
→ DefenseFeatures + scoped DefenseFallback
```

**PASS_SCOPED**

## Epoch 4 — future safe resource attack

반례:

```text
A after discard: same common-genbutsu tile ×2
B after discard: one common-genbutsu tile
```

"safe tile TYPES"만 세면 A의 두 future safe discards를 1로 압축한다.

수리:

```text
NextSafeTypeCount
→ SafeLeaseStock physical copies clipped 0/1/2+
```

**PASS**

## Epoch 5 — 3P scope / hardcoded physics attack

반례:
- latest H6+ human formal core is 4-player.
- H11 uses 4 copies and numeric Local.
- common sanma variants can remove tile types and may or may not support Kita.
- scoring/tsumo/termination differ by profile.

수리:

```text
PlayerCount_R
Cap_R(p)
Legal_R
ScoreTransition_R
Local_R
StrategyProfile_3P
KitaSpec_R
```

**PASS_SCOPED**

## Epoch 6 — AdvanceSet false dominance

반례:

```text
A의 AdvanceSet이 B의 superset
```

이어도 같은 개선패를 뽑은 뒤 A의:
- 목표역,
- 타점,
- 다음 형태

가 더 나쁠 수 있다.

수리:

```text
AdvanceDom
→ SuccessorDom
```

shared draw마다 successor state non-worse를 요구한다.

**PASS_SCOPED**

## Epoch 7 — soft safety abstraction laundering

반례:

```text
Ron mechanism label이 적다
```

가 실제 compatible hidden-state mass가 작다는 뜻은 아니다.

수리:

```text
direct soft-risk ordering rejected
→ DefenseFeatures
→ scoped calibrated DefenseFallback
```

hard authority는 FactSafe에만 둔다.

**PASS_SCOPED**

## Epoch 8 — temporary-furiten future-stock bug

반례:

현재는 safe지만 상대의 다음 draw에서 temporary furiten이 끝나는 패을
`SafeLeaseStock`에 넣으면 다음 내 타패까지의 안전을 과장한다.

수리:

```text
SafeCert + Expiry
SafeStock → SafeLeaseStock
```

**PASS**

## Epoch 9 — omitted control actions

반례:
- call로 ippatsu가 rule-exact하게 끊김,
- 막판 call이 last-tile ownership을 바꿈,
- optional abortive draw가 route를 보존.

기존 "손 전진 call만" 필터는 이런 legal action의 가치를 표현하지 못한다.

수리:

```text
ControlEffect
ControlCert / ControlCallPolicy
AbortiveTransition / AbortivePolicy
```

새 상주후보를 만들지 않고 optional-response slot에 mount한다.

**PASS_SCOPED**

## Epoch 10 — compact-regression / special-resolution attack

공격 1:

```text
E0 intrinsic-equivalent signal
```

도 상대 continuation을 바꿀 수 있는데
compact version이 externality gate를 생략하면
"내 비용 0 → 게임 비용 0" 승격이 된다.

수리:

```text
E0
→ SignalGuard_{Σ_R}
UNKNOWN → baseline
```

공격 2:

special action을 평가·gate만 하고
base action과의 최종 resolution 순서를 명시하지 않으면
도중유국/control-call/KITA가 구현에서 유실될 수 있다.

수리:

```text
SpecialActionResolver
```

**PASS_SCOPED**

## Epoch 11 — semantic closure attack

compact runtime을 standalone으로 읽었을 때:
- `MustAct`,
- `HEAVY`,
- `KeyThreat`,
- `SafeProgress`,
- `GoalAlive/YakuAlive/TargetAlive/...`

가 사용되지만 정의가 외부 H6~H9에 남아 있으면
구현이 다시 과거 monolith에 의존한다.

수리:
- bounded `RouteCert/ℛ_t`,
- scoped `MustAct^ℛ`,
- `ActiveThreatSet/KeyThreat/ThreatClass`,
- `SafeProgress`,
- 모든 `Eval` field semantics

를 current compact runtime 안으로 복구했다.

**PASS**

## Epoch 12 — reliability compaction regression

공격:

H23 이전 compact 후보는 정상상태 의미론은 닫혔지만,
과거 H5/H8이 갖고 있던:
- deadline,
- dirty invalidation,
- FactReset,
- hard conflict isolation,
- emergency totality,
- authorization expiry

가 빠져 있었다.

반례:

```text
상대 리치 직후 시간이 거의 없음
+ SOURCE/AvailDef 계산이 미완료
+ 반드시 한 장 버려야 함
```

이면 정상 알고리즘이 계산완료를 암묵 전제해
실전 총성이 약해진다.

또:

```text
temporary-furiten expiry
```

뒤 오래된 SafeCert를 쓰거나,
사실모순 뒤 derived inference를 계속 쓰는 회귀가 가능했다.

수리:

```text
ReliabilityShell =
  Mode
  + DecisionBudget
  + DirtyDependency
  + FactConsistency
  + FactReset
  + ReliabilityFallback
  + ActionStatus/Residual
  + AuthExpiry
```

새 인간 카드는 추가하지 않았다.

**PASS_SCOPED**

## Epoch 13 — bounded search / legality honesty

### attack A — MustAct route omission

`MustAct^ℛ`는 bounded route set에 상대적이다.
안전한 상방경로가 생성되지 않았다는 이유만으로
위험공격을 "강제"하면 route-generation bias가 공격으로 증폭된다.

수리:

```text
MustAct^ℛ
→ CounterRouteProbe
→ MustActGate
```

위협 아래 강제공격에는 `CONFIRMED_SCOPED`를 요구한다.

### attack B — path pruning lock-in

`PathSet≤2`가 영구고정이면
처음 빠진 치또이/국사/목표역이
나중에 살아 있는 유일경로가 되어도 복구하지 못한다.

수리:

```text
PathCatalog_R
+ PathRefreshTrigger
+ RefreshPathSet
```

상한은 여전히 2.

### attack C — special-candidate overflow

`aF/aV/aS≤3` 뒤
RIICHI/KAN/KITA/CONTROL/ABORTIVE를 한꺼번에 다시 비교하면
HUMAN-RT 후보상한을 우회한다.

수리:

```text
class representative≤1
+ streaming incumbent/challenger resolution
```

동시 비교는 2개 이하.

### attack D — legality from missing rules

```text
incomplete RuleProfile
```

인데도 "legal fallback guaranteed"라고 하면
합법성 근거가 없다.

수리:

```text
Complete(R)
or AuthoritativeLegalMask
```

둘 다 없으면 `LEGALITY_UNRESOLVED`.

**PASS_SCOPED**

## Epoch 14 — semantic name closure / PassCert

공격:

standalone runtime이:
- `TransformPolicy`,
- `ControlPolicy`,
- `SpecialClassOrder`

를 사용하면서 StrategyProfile에 정의하지 않으면
구현자가 결과를 보고 임의 규칙을 채울 수 있다.

수리:

```text
SpecialPolicy = <
  ClassOrder,
  TransformOrder,
  ControlOrder,
  IncomparableFallback
>
```

로 하나로 통합했다.

또 `PassCert`를 이름만 남기면
"패스가 좋아 보인다"는 전망이 엄격한 화료거부 인증으로 세탁될 수 있다.

수리:

```text
PassCert
= sound PASS enclosure
  strictly dominates
  sound WIN enclosure
```

best-case / same-rank / speculative upside는 인증이 아니다.

**PASS**

## Epoch 15 — physical-count double-accounting attack

공격:

```text
upper = Cap - PublicCount - SelfCount
```

에서 `PublicCount`가 내 공개 meld를 포함하고
`SelfCount`도 내 소유패 전체를 포함하는 구현이면
같은 physical tile을 두 번 뺄 수 있다.

또 called discard는:
- discard event history,
- meld history

양쪽에 나타나기 때문에 event log를 그대로 합산해도 이중계산이 생긴다.

`upper=0`은:
- live shape,
- blocker,
- opponent hypothesis,
- safety feature

전부에 영향을 주므로 이 오류는 전파범위가 크다.

수리:

```text
KnownToSelf
= SelfConcealed
+ PublicUnique
```

```text
upper
= Cap_R - KnownToSelf
```

그리고:

```text
VisibleInstanceLedger
```

로 event provenance와 physical counting을 분리했다.

**PASS**

## Epoch 16 — action identity / finish / safety typing

### attack A — action-class identity laundering

`RouteCert.first`가 `action class`인데
`P^+(a)`가 exact action처럼 비교하면:

```text
DISCARD(3m)
DISCARD(7p)
```

같은 서로 다른 현재행동이 하나로 합쳐질 수 있다.

수리:

```text
FirstActionRef = exact current compound ActionID
```

### attack B — FinishSet size as hidden ukeire utility

live finish tile type이 더 많아도
공통 finish에서 yaku/value/goal result가 더 나쁠 수 있다.

수리:

```text
FinishSet
→ hard feasibility feature
→ scoped ShapeFallback input
```

단순 개수/포함으로 보편 지배하지 않는다.

### attack C — KeyThreat hard-order leakage

한 패가 핵심위협에게 현물이어도
다른 활성위협의 론패일 수 있다.

수리:

```text
CommonFactSafe
```

만 보편 hard 최우선으로 유지하고,
그 밖의 상대별 hard vector/KeyThreat/soft features는
`DefenseFallback_{Σ_R}`로 보낸다.

**PASS**

## Epoch 17 — abstract-contract closure

공격 1:

```text
ℛ={ρ | Verify_R(ρ)}
```

인데 `Verify_R`가 current compact file에서 정의되지 않으면
route certification 기준을 구현자가 임의로 채울 수 있다.

수리:

```text
VerifyRoute_R
```

를 legality / exact score transition / rank / terminal / assumption / expiry로 닫았다.

공격 2:

StrategyProfile에는 `ThreatPolicy` 하나만 있는데
본문에서 `ThreatPriority`, `HeavyPolicy`를 별개 함수처럼 쓰면
policy registry가 닫히지 않는다.

수리:

```text
ThreatPolicy = <ActiveRule,KeyPriority,HeavyRule>
```

로 통합했다.

`DefenseFallback.SafetyGate`도 같은 방식으로 schema를 명시했다.

공격 3:

`SuccessorDom`이 undefined `RoleNonWorse`와 `LiveShape`를 hard comparator에 쓰면
heuristic shape order가 다시 hard dominance로 승격될 수 있다.

수리:

```text
SuccHardSig
+ HardSuccNonWorse
```

로 Goal/Yaku/Path/Target/AvailDef/MinimumResult만 hard 비교한다.

**PASS**

## Epoch 18 — profile version closure

공격:

runtime output / dependency invalidation은:

```text
R.version
Σ.version
```

을 쓰는데 profile schema에 `Version`이 없으면
stale cache/Auth의 동등성 기준이 구현자 재량이 된다.

수리:

```text
RuleProfile.Version
StrategyProfile.Version
```

을 명시하고,
`CalibrationVersion`과 strategy runtime version을 분리했다.

- rule version change → rule-conditional state 전부 invalid.
- strategy version change → policy/route-policy/special-order invalid.
- observed hard fact 자체는 strategy version change로 삭제하지 않는다.

**PASS**

18번째 epoch 뒤 새 **즉시 material patch**는 현재 정적 공격에서 발견되지 않았다.

---

# 19. Frozen Regression

```text
Terminal rank order                 PRESERVED
Ordinal lottery overclaim           REMOVED
Legal action totality               PRESERVED under loaded RuleProfile
Full score transition               PRESERVED / generalized
4P legality                         PRESERVED_SCOPED
3P human runtime                    ADDED_SCOPED
Hard tile counts                    GENERALIZED Cap_R
Exact hidden-hand claim             FORBIDDEN
In-game probability                 FORBIDDEN
Structural shanten                  PRESERVED as sub-feature
Dead wait sovereignty               REMOVED
Availability-aware path             ADDED
aF/aV/aS≤3                          PRESERVED
Genbutsu hard safety                PRESERVED
Evidence-family scalar count        REMOVED
Soft defense scalarization          REMOVED
Defense features                    PROFILE-SCOPED
NextSafe type-count compression     REPLACED by SafeLeaseStock
Multi-threat handling               PRESERVED
Call+discard compound action        PRESERVED
Control-only call                   ADDED_SCOPED
Optional abortive draw              ADDED_SCOPED
Special action resolution            EXPLICIT
Standalone field semantics           CLOSED
Deadline / emergency totality          RESTORED
FactReset / dirty invalidation          RESTORED
Authorization expiry                    EXPLICIT
Path recovery under hard death           ADDED
MustAct counter-route challenge           ADDED
Special candidate active bound            RESTORED
Legality guarantee without basis          REMOVED
Special policy undefined names               CLOSED
Strict win-pass certificate                   CLOSED
Physical tile-count partition                  CLOSED
Route first-action typing                      CLOSED
FinishSet scalar/dominance overclaim            REMOVED
KeyThreat hard safety ordering                  REMOVED
Route verification predicate                   CLOSED
Threat/defense policy interfaces                CLOSED
Successor hard dominance                       CLOSED
Rule/Strategy profile versioning                CLOSED
MustAct/Threat/KeyThreat              EXPLICIT
Riichi heuristic                    PROFILE-SCOPED
Kan heuristic                       PROFILE-SCOPED
Kita                               RULE/PROFILE-SCOPED
H11 hard source facts               PRESERVED
FOCUS direct discard override       REMOVED
SOURCE                              OPTIONAL
SIGNAL E0                           OPTIONAL + EXTERNALITY_GUARDED
SIGNAL E1                           DISABLED_BY_DEFAULT
Human visible cards                 STILL 5
```

---

# 20. Remaining Residuals

현재 남은 것은 설계로 더 문장을 붙인다고 해결되기 어렵다.

1. `AvailDef_R`의 인간 근사는 exact engine과 불일치할 수 있다.
2. `DefenseFeatures`는 실제 deal-in probability가 아니며 profile calibration이 필요하다.
3. 4P legacy `ThreatPolicy`·`RiichiPolicy`는 empirical.
4. 3P policy thresholds는 별도 실증이 필요하다.
5. crossing rank distributions는 LotteryProfile 없이는 진짜 비교불능이다.
6. SOURCE soft signal의 실제 rank gain은 미검증.
7. SIGNAL E1 causal response effect는 미검증.
8. opponent policy / wall stochasticity에 대한 인간 runtime의 global optimality는 증명하지 않는다.
9. 4P/3P 각 플랫폼 rule profile을 잘못 입력하면 결과도 잘못된다.
10. control-call/abortive discretionary choices도 profile별 실증이 필요하다.
11. human deadline under extreme UI/event-loss conditions can still degrade to legality-only fallback.
12. full behavioral superiority는 paired log/simulation 전까지 미증명.

```text
NO_NEW_STATIC_MATERIAL_PATCH_FOUND
EMPIRICAL_CALIBRATION_PENDING
```

---

# 21. Validation Plan

## 21.1 property tests

반드시:
- 같은 physical tile이 discard-event와 called meld history에 모두 나타나도 `KnownToSelf`에 1회만 셈.
- `PublicUnique ∩ SelfConcealed = ∅`.
- `KnownToSelf(p)≤Cap_R(p)` invariant 위반 시 FactReset.
- RouteCert의 FirstActionRef는 exact current compound ActionID.
- FinishSet 개수/단순 포함만으로 action dominance를 만들지 않음.
- 복수위협에서 CommonFactSafe 외 KeyThreat safety는 profile policy로만 사용.
- VerifyRoute_R가 FirstActionRef/ScoreTransition/rank/terminal/assumption/expiry를 모두 검사.
- ThreatPolicy의 Active/Key/Heavy 함수가 profile 안에 모두 정의됨.
- SuccessorDom hard comparator에 shape heuristic/초과타점을 넣지 않음.
- RuleProfile/StrategyProfile 모두 Version을 가지며 cache/Auth가 version-bound.
- CalibrationVersion을 runtime StrategyProfile.Version과 혼동하지 않음.
- `Cap_R=0` tile을 live completion으로 사용하지 않음.
- all waits dead tenpai를 live tenpai로 부르지 않음.
- 3P removed tiles가 Local_R / shanten route에 침투하지 않음.
- FactSafe면 immediate ron candidate에서 제외.
- DefenseFeatures의 개수/폭을 보편 확률서열로 몰래 바꾸지 않음.
- SafeLeaseStock은 expiry가 next-self-discard를 넘는 physical self copies만 센다.
- SOURCE alone이 hard action order를 바꾸지 않음.
- E0는 SignalGuard=PASS일 때만 override.
- E1 disabled profile에서 signal override 없음.
- rule-inlegal CHI/KITA/KAN/ABORTIVE 반환 없음.
- triggered special action은 SpecialActionResolver를 거쳐 base와 resolution됨.
- MustAct^ℛ/HEAVY/KeyThreat/SafeProgress가 current file 안에서 total하게 정의됨.
- Eval field 하나도 과거 H6~H11 정의를 암묵 참조하지 않음.
- deadline 도달 시 optional state는 PASS 또는 fresh strict override만 반환.
- mandatory discard deadline에서 EmergencyDiscard가 항상 legal discard를 반환.
- FactConflict 뒤 soft/source inference를 재사용하지 않음.
- temporary-furiten expiry가 SafeCert/SafeLeaseStock을 Dirty 처리.
- 새 state version에서 old Auth를 재사용하지 않음.
- active PathSet이 hard-dead/VALUE-broken되면 PathCatalog를 재검색하며 ≤2 유지.
- MustAct force-attack은 CounterRouteProbe 없는 CONFIRMED로 승격되지 않음.
- special action 비교에서 동시에 유지하는 visible candidates≤2.
- Complete(R)도 authoritative legal mask도 없으면 legal-totality claim 금지.
- SpecialPolicy의 모든 class/within-class order가 profile version에 정의됨.
- PassCert 없이 legal win을 speculative future 때문에 거부하지 않음.

## 21.2 empirical tests

우선순위:
1. availability-aware distance vs original shanten order,
2. scoped DefenseFallback vs legacy SUPPORTED_2/1,
3. SafeLeaseStock vs legacy NextSafeTypes,
4. 4P riichi/threat heuristic,
5. 3P threat/riichi/kita,
6. SOURCE FOCUS,
7. SIGNAL E1.

---

# 22. Research / External Evidence Boundary

외부 연구는 다음 용도로만 쓴다.

- availability-aware deficiency/shanten 계산 아이디어,
- large-scale rollout infrastructure,
- 3P를 별도 decision problem으로 다루는 근거,
- black-box Mahjong agent에서 인간해석 가능한 local rule을 추출하는 방법.

```text
Paper supports method
≠ H23 heuristic proven
```

현재 참고:
- Yan, Li & Li (2021), *A Fast Algorithm for Computing the Deficiency Number of a Mahjong Hand*.
- Nishimori et al. (2026), *Mahjax*.
- Li et al. (2025), *Mxplainer*.
- Zhao & Holden (2022), *Building a 3-Player Mahjong AI using Deep Reinforcement Learning*.
- current WRC 3P rules as one example of materially different sanma rules.

---

# 23. Construction Status

```text
ConstructionStatus(H23) =
  SELF_CONTAINED_COMPACT_RUNTIME
  + RULE_ALGEBRA_4P_3P
  + TERMINAL_ORDINAL_HONEST
  + AVAILABILITY_AWARE_HAND_DISTANCE
  + SOFT_DEFENSE_FEATURES_SCOPED
  + EXPIRY_AWARE_SAFE_STOCK
  + CONTROL_ACTION_COMPLETE
  + SPECIAL_ACTION_RESOLVER
  + E0_EXTERNALITY_GUARD
  + SEMANTIC_CLOSURE
  + RELIABILITY_SHELL
  + DEADLINE_AWARE_TOTALITY
  + PATH_RECOVERY
  + COUNTER_ROUTE_CHALLENGE
  + BOUNDED_SPECIAL_RESOLUTION
  + LEGALITY_BASIS_EXPLICIT
  + SPECIAL_POLICY_CLOSED
  + STRICT_PASS_CERT
  + UNIQUE_INSTANCE_TILE_ACCOUNTING
  + EXACT_ROUTE_ACTION_IDENTITY
  + FINISH_FEATURE_NONSCALAR
  + COMMON_FACT_ONLY_HARD_DEFENSE
  + ROUTE_VERIFY_CONTRACT
  + POLICY_INTERFACE_CLOSURE
  + HARD_SUCCESSOR_DOMINANCE
  + PROFILE_VERSION_CLOSURE
  + SOURCE_AS_QUERY_PRIOR
  + OPTIONAL_SIGNAL
  + CALIBRATION_READY
  + HUMAN_VISIBLE_CARDS_5
  + EMPIRICAL_SUPERIORITY_NOT_YET_PROVEN
```

```text
RECURSIVE_IMPROVEMENT = STOP
```

이번 누적 run은 18 bounded epochs에서 멈춘다.

이유:

```text
남은 핵심 residual이
새 static rule 부족보다
empirical calibration / platform rule data / lottery preference
문제로 이동했기 때문.
```

```text
Stop ≠ proof of optimal mahjong
```
