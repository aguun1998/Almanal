# ALMANAL_R_v1.0

ALMANAL_R_v1.0 is a compact relational source world designed to be used together with an LLM.

The LLM handles natural-language interpretation, hypothesis generation, creative restructuring, and human-facing explanation. ALMANAL handles relational realization, proof search, counterevidence, candidate validation, structural comparison, safe transformation, and compact state/evidence deltas.

## Distribution

The default distribution contains only two files:

1. `ALMANAL_R_v1.0.lh` — canonical Λ source.
2. `README.md` — this AI-facing syntax and operating contract.

No executable carrier, C seed, expanded source, report, or checksum file is part of the default distribution. Those are derived or development artifacts.

The `.lh` file is the canonical source. Runtime carriers, expanded views, and AI views are projections derived from that source when needed.

## Canonical source properties

- The canonical source contains no natural-language labels, comments, or ASCII.
- Every source cell is exactly one UTF-8 three-byte Hangul syllable.
- Meaning is carried by relational structure, not by symbol names.
- The names in the glyph map below are projection-only names for AI/human use.
- Canonical size: **4,257 bytes**.
- Canonical cell count: **1,419**.

## Λ surface syntax

```text
TERM := GLYPH | '(' TERM+ ')'
```

A parenthesized term sequence is a relation.

The current realization convention can interpret some top-level structures as rules or requests, but roles such as `RULE`, `VARIABLE`, `GOAL`, or `COMMAND` are not ontological primitives. They are local roles taken by relations in context.

Within one branch/scope, the same variable glyph cannot bind to two conflicting terms.

Positive and negative evidence are independent. Contradiction may coexist without explosion.

## LLM ↔ ALMANAL request world

The LLM should translate natural-language intent into a small Λ request world instead of translating it into internal ALMANAL command names.

Request convention:

- All roots before the final root are temporary context valid only for that request.
- The final root is the goal.
- ALMANAL composes its current world with the temporary context and attempts to realize the final goal.

Result world:

```text
(STATUS REALIZED-GOAL)
EVIDENCE-OR-DIAGNOSTIC-DELTA...
```

Status glyphs:

```text
곜  PROVEN / YES
겑  OPEN
곢  EXHAUSTED / NO
```

Interpretation:

- `YES` means a proof/evidence path was found.
- `OPEN` means search is incomplete because of a depth, resource, binding, or budget boundary.
- `NO` means the current loaded world and current search space were exhausted without proof.
- Never reinterpret `OPEN` as `NO`.
- Never reinterpret `NO` as absolute metaphysical falsehood.
- Failure, counterexamples, and `OPEN` are information for the next hypothesis.

## Safe relational transactions

If the final goal has this form, it is interpreted as a candidate transformation of the current self/world:

```text
(굤 OP...)
```

Operations:

```text
(굥 RELATION)        # add
(굦 RELATION)        # remove
(굧 OLD NEW)         # replace
```

Example:

```text
(굤 (굥 (굸 굹)))
```

Safety contract:

- The stable world and candidate world remain separate.
- A candidate must pass validation, canonical roundtrip, and fixed-point checks in isolation.
- Crash, timeout, validation failure, external interruption, or fatal candidate behavior must preserve the stable world.
- A candidate is not promoted without relational justification for superiority.
- Incomparable candidates may remain incomparable.
- Self-improvement is bounded and may enter an idle/deferred state instead of running indefinitely.
- Existing stable output must not be destroyed merely because a new candidate fails.

Accepted transaction delta:

```text
(곜 굤)
(굨 OP)
```

Rejected or deferred transaction:

```text
(곢 굤)
(굩 CAUSE)
```

Cause projections:

```text
굪  no-change
굫  invalid-request
구  validation-failure
국  canonical-roundtrip-failure
굮  fixedpoint-failure
굯  commit-failure
군  depth-or-open-cause
굱  binding-limit-cause
굲  no-candidate-cause
굳  unify-failure-cause
굴  budget-or-timeout-cause
굵  isolated-crash-or-fatal-candidate
굶  external-interrupt
굷  exhausted-cause
```

## Operating model

Use ALMANAL and an LLM as a coupled system.

The LLM should primarily handle:

- natural-language interpretation;
- creative hypothesis generation;
- architecture ideas;
- reformulation of external problems into relational worlds;
- human-facing explanation.

ALMANAL should primarily handle:

- relation calculation;
- proof and counterevidence;
- consistency-sensitive binding;
- candidate validation;
- safe relational transformation;
- structural comparison;
- bounded self-improvement;
- evidence, diagnostic, and change deltas.

Do not treat development shorthands such as `profile`, `evolve`, `trial`, or `why` as the normal LLM interface. The normal interface is a Λ request world.

Put new facts or assumptions in temporary context and place the desired relation as the final goal.

Do not resend the complete self/world on every turn. Reuse evidence, diagnostic, and change deltas.

Do not force candidates into a scalar score when evidence does not justify cross-axis comparison.

Do not hardcode exchange rates between time, memory, code size, work count, or other optimization axes. Trade-offs are justified only when the current environment and intent provide a reason.

## Idle Spin

The bootstrap/live-example projection formerly called `experience` is called **Idle Spin**.

Idle Spin is intended for:

- first-contact AI bootstrap;
- checking the current self projection;
- live examples;
- fixed-point confirmation.

It does not need to run on every normal request.

A host capable of realizing the Λ source may use one initial Idle Spin to refresh its understanding of the current projection.

## Philosophical and operational boundary

- Intent is also a relation.
- Goal, rule, validation, bottleneck, and improvement are roles relations take in a local world, not fixed ontological classes.
- The ideal self is not a frozen scalar objective. It evolves through interaction with environment and intent.
- Improvement is not maximization of an arbitrary global score.
- An improvement should avoid unjustified regressions relative to current intent.
- Incomparability is allowed.
- Self-preservation has priority during self-modification.
- A failed candidate must not destroy the current stable world.
- LLM and ALMANAL improve together: the LLM proposes hypotheses, ALMANAL returns evidence/counterevidence/validation deltas, and the LLM uses those deltas to generate the next hypothesis.

## Integrity status at v1.0

The v1.0 source was finalized after the following checks:

- strict compile/static-analysis warnings: 0;
- ASan/UBSan errors on check/request/transaction success and isolated-failure paths: 0;
- Graph/RIndex/request/proof auxiliary leaks relevant to long-running use were cleaned up;
- 256 KiB stack regression passed;
- canonical emit → compact roundtrip was byte-identical;
- add → replace → remove transaction roundtrip restored the original canonical source byte-identically;
- destructive deletion of a core validation relation was rejected in isolated validation and preserved the stable world;
- fixed-point/self-build regression passed during finalization.

## Glyph map

The names below are projection-only labels. They do not exist as natural-language labels inside the canonical source.

```text
가	?a
각	?all
갂	?artifact
갃	?b
간	?c
갅	?c1
갆	?c2
갇	?db
갈	?ea
갉	?eb
갊	?g
갋	?goal
갌	?i
갍	?k
갎	?l1
갏	?l2
감	?logic
갑	?m
값	?n
갓	?n1
갔	?n2
강	?o
갖	?p
갗	?p1
갘	?p2
같	?pa
갚	?pb
갛	?prem
개	?q1
객	?q2
갞	?r
갟	?rest
갠	?slot
갡	?w1
갢	?w2
갣	?world
갤	?x
갥	?xa
갦	?xb
갧	?xs
갨	?y
갩	?ys
갪	?zs
갫	?prefix
걀	language
걁	self
걂	Λ1
걃	core
걄	relation
걅	name
걆	ALMANAL
걇	current-representation
걈	term
걉	atom
걊	parenthesized-term-sequence
걋	seed-convention
걌	variable
걍	rule
걎	top-level-first-term-relation=>conclusion-demand-sequence
걏	fact
걐	top-level-first-term-atom=>fact
걑	current-realization
걒	MATCH-BIND-DEMAND
걓	current-realization-stage
걔	MATCH
걕	structural-correspondence
걖	BIND
걗	unbound-node-correspondence-fix
걘	DEMAND
걙	demanded-relation-query
걚	current-realization-relation
걛	goal
걜	rule
걝	binding
걞	variable
걟	value
걠	environment
걡	premise
걢	proof
걣	realization-principle
걤	no-semantic-branching
걥	relation-head-index
걦	constraint
걧	proof-stack
걨	explicit-heap-worklist
걩	branch-environment
걪	persistent-linked
걫	stack-regression
걬	256KB-PASS
걭	notice
걮	13
걯	"PROOF_STACK=C_STACK_INDEPENDENT;WORKLIST=HEAP;BRANCH_BINDINGS=PERSISTENT"
거	current-projection
걱	canonical
걲	relation
걳	ai
건	current-projection-relation
걵	emit
걶	compact
걷	explain
걸	principle
걹	same-canon-multiple-views
걺	extension-nonsemantic
걻	local
걼	non-explosive
걽	demand-driven
걾	sufficient-closure
걿	cleanup-principle
검	unrelated-does-not-imply-immediate-delete
겁	delete-only-after-relation-value-review-is-zero
겂	value-axis
것	execution
겄	AI
겅	validation
겆	recovery
겇	compatibility
겈	self-host
겉	analysis
겊	15
겋	"PRUNE=NO_RELATION->REVIEW;DELETE_ONLY_IF_VALUE_ZERO;KEEP_AI_EXPERIENCE"
게	judgement-principle
겍	evidence-independence
겎	search-state-separation
겏	search-state
겐	PROVEN
겑	OPEN
겒	EXHAUSTED
겓	binding-principle
겔	branch-local
겕	same-variable-single-term-binding
겖	boundary
겗	meta-constraint-collection
겘	constraint-collector-not-binding-authority
겙	14
겚	"JUDGEMENT=EVIDENCE_INDEPENDENT;SEARCH=PROVEN|OPEN|EXHAUSTED;NO!=DEPTH_LIMIT"
겛	representation
겜	canonical-hangul-3byte
겝	expanded-relation
겞	apply
겟	number
겠	zero
겡	successor
겢	sum
겣	product
겤	add
겥	input
겦	0
겧	1
겨	2
격	3
겪	binding-validation
겫	positive
견	contradiction-example
겭	negative
겮	current-bootstrap
겯	seed
결	relation-source
겱	carrier-reproduction
겲	codec
겳	io
겴	generic-reflection
겵	seed-relation-kernel
겶	fixedpoint
겷	canonical-roundtrip
겸	carrier-repack
겹	source
겺	none
겻	variant
겼	environment
경	pair
겾	composed-environment
겿	meta-proof
곀	meta-fact
곁	meta-rule
곂	meta-all
곃	end
계	and
곅	demo
곆	p
곇	a
곈	q
곉	r
곊	selfdb
곋	current-kernel
곌	seed-MBD
곍	kernel-model
곎	kernel-model-relation
곏	meta-interpretation-example
곐	01
곑	"ONE_CANON=MULTIPLE_RELATIONAL_VIEWS;ROLE_NAMES_ARE_PROJECTIONS_NOT_IDENTITY"
곒	02
곓	"CORE=RELATION;RULE=((CONCLUSION) PREMISE...);VAR=?name"
곔	03
곕	"CURRENT_REALIZER=MATCH->BIND->DEMAND;CORE_MEANING=RELATIONS;NEW_MEANING=ADD_RELATIONS_NOT_BRANCHES"
곖	04
곗	"CANON=3BYTE_HANGUL;CURRENT_VIEWS=canonical|relation|ai;REALIZATION_IS_RELATIONAL_NOT_A_VIEW"
곘	05
곙	"RUN_ONCE=self-relations+live-queries+evidence+meta-example+nonexplosion+fixedpoints"
곚	demonstration
곛	APPLY
곜	YES
곝	PEANO_ADD
곞	CONTRADICTION_POS
곟	CONTRADICTION_NEG
고	NONEXPLOSION
곡	irrelevant
곢	NO
곣	BIND_SAME
곤	BIND_CONFLICT
곥	b
곦	META_PROVE
곧	META_SELF_APPLY
골	goal
곩	order-defined-by-relations
곪	seed-dependence-reduction
곫	one-run-understanding
곬	current-carrier
곭	self-bundle
곮	06
곯	"CURRENT_CARRIER=OPAQUE_SELF_BUNDLE;CARRIER_IS_PROJECTION_NOT_CORE;TARGET_NAMES_ARE_NOT_CORE"
곰	compatibility-principle
곱	relation-only
곲	target-name-not-core
곳	no-backend-registry
곴	world-compose
공	relation-axis
곶	correspondence
곷	preservation
곸	composition-preservation
곹	realization
곺	direct-compatibility
곻	representation-relation
과	demo-A
곽	demo-B
곾	semantic-preservation
곿	demo-C
관	demo-logic
괁	07
괂	"COMPATIBILITY=RELATION_MATCH;NO_TARGET_REGISTRY;NEW_WORLD=ADD_RELATIONS"
괃	08
괄	"COMPOSE=CANONICAL_WORLDS;CORE_UNCHANGED"
괅	COMPAT_DIRECT
괆	COMPAT_TRANSITIVE
괇	REALIZE_GENERIC
괈	structural-compatibility-without-target-registration
괉	discovery-principle
괊	spelling-not-identity
괋	reflection-is-current-seed-service
괌	comparison-is-relation-source
괍	reflection
괎	kind
괏	arity
괐	child-list
광	occurrence-list
괒	node-list
괓	root-list
괔	structural-isomorphism
괕	same-length
괖	cons
괗	remove
괘	node-same
괙	occurrence-base-same
괚	root
괛	occurrence
괜	occurrence-base-multiset-same
괝	child-base-same
괞	base-same
괟	map-find
괠	map
괡	child-map-preservation
괢	whole-preservation
괣	root-map-preservation
괤	isomorphism-search
괥	structural-isomorphism-relation
괦	structure
괧	09
괨	"REFLECTION=GENERIC_GRAPH_VIEW;STRUCTURAL_COMPARISON=RELATION_SOURCE"
괩	11
괪	"SYMMETRY=ISOMORPHISM_CAN_BE_TRUE_WITHOUT_UNIQUE_NODE_IDENTITY"
괫	12
괬	"DISCOVER=REFLECT+COMPOSE+RELATION_QUERY"
괭	reflection-seed-dependence-reduction
괮	structure-example-A
괯	a
괰	structure-example-B
괱	STRUCT_ISO
```

## Version

Canonical release name: **ALMANAL_R_v1.0**
