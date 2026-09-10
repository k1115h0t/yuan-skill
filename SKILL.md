---
name: academic-paper-review
description: Reviews academic-paper front sections (Abstract, Introduction, Related Work, and Problem-Formulation/Method boundaries) for argument structure, problem-method mapping, paragraph continuity, taxonomy coherence, section ownership, explicit referents, experimental evidence, and professor-comment alignment; supports blind regression review.
---

# Academic Paper Review Skill — Front-Section Review v0.16

## 1. Purpose

This skill reviews the front sections of an academic paper using the manuscript and, when available, professor/supervisor annotations as the primary evidence.

The review is not a grammar pass. Its main job is to reconstruct the paper's argument, identify why a reader or reviewer is forced to infer missing logic, detect when a section performs the wrong rhetorical job, and convert repeated professor comments into reusable review rules.

The supported modules are:

- Abstract: A01-A10.
- Introduction: I01-I05.
- Related Work: R01-R05.
- Cross-cutting prose/logic rules: X01-X03.
- Section architecture and ownership: S01-S03.

Language polishing is secondary and must occur only after higher-level logic, evidence, and section-role checks.

## 2. Inputs and evidence discipline

Use, when available:

- Original manuscript PDF or source text.
- Revised manuscript PDF or source text.
- Professor/supervisor annotated PDF or comment document.
- Explicitly supplied experimental tables, figures, or appendices needed to verify claims.

Treat supplied materials as the source of truth. Do not silently invent claims, experiments, motivations, contributions, section intent, or causal relationships.

When a professor comment is ambiguous, state the ambiguity and identify the strongest interpretation supported by the local context.

### Blind regression mode

When the task explicitly requests blind review or regression testing:

- analyze only annotation-free manuscript text and explicitly permitted evidence;
- do not open, recover, quote, infer, or search for withheld annotations;
- do not use prior professor comments as evidence for the current decision;
- run every applicable rule independently;
- keep evaluator-only labels and comments outside this Skill file.

## 3. Module routing

Route by supplied sections.

- Abstract only: run A01-A10 plus X01-X03.
- Introduction only: run I02-I05 plus X01-X03; mark I01 not evaluable without the Abstract.
- Abstract + Introduction: run both modules separately, then I01 cross-section comparison.
- Related Work: run R01-R05 plus X01-X03 and S01-S03 where section placement is relevant.
- Problem Formulation / Threat Model / Method opening: run S01-S03 plus X01-X03; use Introduction or Related Work findings only when adjacent section logic is available.
- Whole front section: run all applicable modules. Do not let one module's finding substitute for another module's required checks.

When professor comments are supplied, map each comment to the rule that best explains the root cause. One comment may map to more than one rule, but identify one primary rule.

---

# 4. Cross-cutting rules

These rules apply across Abstract, Introduction, Related Work, Problem Formulation, and Method openings.

## X01 — Ambiguous or lazy anaphora / unclear referent

Trigger when a pronoun or compact reference such as:

- this / that / these / those;
- it / they / them / their;
- both / both types / such;
- the former / the latter;
- this issue / this vulnerability / this setting / this behavior;

forces the reader to reconstruct the referent, especially when two or more candidate antecedents exist or when the intended noun phrase is short enough to repeat explicitly.

This rule reflects a strict academic-writing preference: do not make the reader resolve a reference when repeating the exact technical noun is cheap and clearer.

Hard test:

1. Replace the pronoun/reference with the intended noun phrase.
2. If the sentence becomes materially clearer with little cost, flag X01.
3. If two antecedents are plausible, flag X01 strongly.
4. Do not flag a pronoun merely because it is a pronoun. A local, unique, unmistakable referent is normally acceptable.
5. Stricter front-section pattern: when a paragraph-opening definition names the paper's technical object and the next sentence begins with a bare plural pronoun such as `They`, repeat the technical noun if that sentence establishes domain, use, scope, or motivation. In this position, the explicit noun improves topic anchoring at negligible cost even if the antecedent is grammatically recoverable.
6. Do not generalize item 5 to every immediate `it/they`: a singular pronoun that continues one unmistakable method subject in the same local operation remains acceptable.
7. ABSTRACT-LABEL ANAPHOR: treat compact labels such as `this vulnerability`, `this issue`, `this risk`, or `this concern` more strictly when the preceding sentence contains several candidate causes, settings, or security concepts. If replacing the label with a concrete noun phrase forces a choice among those candidates, trigger X01 even when the intended general topic is inferable. Do not trigger when the immediately preceding clause explicitly names one unique antecedent, as in `this interface` after a sentence that has just defined the interface.
8. REFERENTIAL RELABELING: if the antecedent is stated as one argumentative category (for example, a `security concern`, deployment condition, or integrity problem) and the next sentence silently renames it as `this vulnerability` or `this issue`, require the concrete threatened object or mechanism to remain explicit. A reader should not have to infer whether the label refers to the condition, the concern, or the causal weakness.
9. RULE-INDEPENDENCE GUARD: run X01 independently of X02/A03. A stronger causal or role-conflation finding does not suppress a referent finding. In particular, bare summaries such as `both types` or `these two` should still trigger X01 when the exact paired nouns are absent or expensive to reconstruct, even if the same sentence also triggers X02.
10. EXCERPT-BOUNDARY GUARD: if an anaphoric phrase occurs at the very start of a supplied excerpt and its antecedent is simply outside the excerpt, do not convert missing excerpt context into a definite X01 finding. Mark X01 uncertain only when the omitted context prevents adjudication. Trigger X01 only when the supplied text itself establishes ambiguity, competing antecedents, or a misleading relabeling. This guard does not apply to anaphors such as `this vulnerability` that follow a supplied sentence containing several candidate concepts.

Output:

- quote the ambiguous reference;
- name the plausible antecedent(s);
- recommend the exact noun phrase or a more explicit construction.

## X02 — Concept-role conflation or invalid causal framing

Trigger when a sentence mixes concepts that play different roles in the argument and therefore states an invalid or misleading relationship.

Typical roles include:

- task / release setting;
- threat or protected object;
- privacy/security requirement;
- utility or fidelity objective;
- observed limitation;
- method mechanism;
- evaluation metric;
- evidence.

Examples of risky structures:

- treating a utility objective as if it were the privacy requirement itself;
- claiming that "releasing useful data requires preserving X" without distinguishing protection from utility preservation;
- moving from a broad threat statement to a specific method requirement without an intermediate premise;
- using "therefore" after statements that do not establish the category or scope of the conclusion.

Role-ledger test:

| Phrase | Role |
|---|---|
| What is being done? | task/setting |
| What must be protected? | security/privacy requirement |
| What should remain useful? | utility objective |
| What currently fails? | limitation/gap |
| What changes that failure? | method mechanism |
| What proves it? | evidence |

If one sentence silently swaps roles or derives one role from another without a bridge, trigger X02.

Represent failures as:

P -> [missing role/causal premise] -> Q

Output the missing premise or recommend splitting the sentence so that protection, utility, gap, and mechanism are stated separately.

## X03 — Empty meta-navigation or author-side stage directions

Trigger on prose that mainly tells the reader what the next section will do, announces obvious document structure, or narrates the author's writing process without adding scientific content.

Examples:

- "The next section formalizes..."
- "We next discuss..."
- "The remainder of this section is organized as follows..."

Do not trigger when navigation is genuinely necessary for a complicated proof/algorithm dependency or mandated by venue style.

Default decision for short research papers: DELETE or relocate to a section opening only if it materially improves navigation.

---

# 5. Abstract module

## 5.1 Core model

Reconstruct the Abstract as:

Background (B) -> Problem (P) -> Gap (G) -> Method (M) -> Results (R) -> Conclusion (C)

Test:

- B -> P: Does the background actually establish the research problem?
- P -> G: Is the missing capability/limitation explicit?
- G -> M: Does each major design respond to the stated gap?
- M -> R: Do reported experiments test the claimed contributions?
- R -> C: Does the conclusion stay inside the evidence boundary?

If the reader must infer an unstated premise, mark the transition weak.

## A01 — Low-information, removable, merge-only, or generic-definition sentence

Run on every Abstract sentence.

Deletion test:

1. Delete the sentence.
2. Repair simple anaphora.
3. Ask which unique proposition disappears.
4. If no paper-specific problem, gap, mechanism, contribution, evidence boundary, or constraint is lost, mark DELETE-CANDIDATE.

Standalone-worthiness test:

- one narrow proposition immediately consumed by the next/previous sentence -> MERGE-ONLY;
- two or more independent substantive propositions -> normally KEEP unless another rule triggers.

Hard pattern: generic opening definitions are not automatically useful. If the next paper-specific problem remains fully intelligible after repaired deletion, delete/merge the generic definition.

Exception: keep a generic-looking sentence only when it supplies the unique concrete causal antecedent used by the next claim.

Hard merge pattern: if a sentence only defines a reference object or generic state that the next sentence immediately consumes (for example, `trusted samples establish normal behavior` followed by a method measuring deviation from `this reference`), rerun the deletion test after replacing the anaphor with the concrete noun phrase. If the scientific proposition is preserved, mark the first sentence MERGE-ONLY or DELETE-CANDIDATE rather than keeping it merely as an antecedent supplier.

Output: KEEP / MERGE-ONLY / DELETE-CANDIDATE and the exact unique proposition count.

## A02 — Motivation built on a weak conditional scenario

Trigger when the paper's main motivation is built from multiple hedged assumptions such as may/might/often/could plus "under this setting" rather than an objective technical or deployment chain.

Distinguish:

- existence/prevalence assumption;
- resource/access assumption;
- technical variability;
- ordinary epistemic caution.

Do not mechanically flag uncertainty about attack/sample variability.

Output the current conditional chain and the objective chain that should replace it.

## A03 — Logical connector or summary bridge without sufficient logic

Trigger on because/therefore/thus/hence/consequently or equivalent summary moves when Q does not follow locally from P.

Hard patterns:

1. A sentence that invents abstract labels for preceding concrete facts and then uses those labels to derive a conclusion is weak unless the labels add a real causal mechanism.
2. ABSTRACT-GAP SEQUENCING: if an Abstract moves from a broad task/privacy/utility statement directly to `existing methods do X, which causes Y`, test whether the preceding text has actually established why X is technically inadequate for the synthesis objective. If the key consistency criterion, reconstruction difficulty, or other missing premise appears only after the prior-work limitation, flag the bridge rather than treating adjacency as explanation.
3. RENAMED-RISK BRIDGE: changing concrete facts into labels such as `opaque provenance`, `trigger-dependent behavior`, `heterogeneity`, or `complexity` and then writing `therefore this creates a risk/need` does not by itself supply a mechanism. Trigger when the conclusion depends on the relabeling rather than an explicit consequence chain.
4. RULE-INDEPENDENCE GUARD: run A03 independently of X02. If an Abstract first states a broad requirement in a conceptually flawed or conflated way and then immediately presents `existing methods do X, which causes Y`, still test whether the missing technical criterion linking X to Y was established. An X02 finding in the earlier sentence does not resolve or suppress a separate A03 sequencing gap.

Represent as:

P -> [missing premise] -> Q

or:

concrete facts -> renamed labels -> connector -> conclusion

## A04 — Problem-solution mapping failure

For every major problem/gap P_i and every major design D_j, build:

| Problem/gap | Design | Exact bridge text | Mechanistic reason | Status |

Status:

- Explicit;
- Inferable-only;
- Ambiguous;
- Unmapped.

A knowledgeable reader being able to guess the mapping is not enough. Inferable-only, ambiguous, and unmapped are findings.

## A05 — Concept overload

Trigger when several new modules/components are named before the reader understands the core causal mechanism.

Prefer:

problem -> mechanism -> major design names

over a dense inventory of component names.

## A06 — Procedure listing instead of mechanism explanation

Trigger when method prose is dominated by first/then/next/finally and explains order without purpose.

Rewrite logic as:

purpose -> operation -> consequence.

## A07 — Orphan or locally unmotivated method component

For every new component:

| Component | Nearest need | Exact local bridge | Output/signal | Status |

Status:

- explicitly motivated;
- role-only;
- distant-only;
- name-inferable;
- pipeline-orphaned.

A component name that sounds self-explanatory does not count as motivation.

## A08 — Subjective, vague-comparative, or unsupported evaluation

Trigger on author-side labels such as effective, robust, superior, competitive, promising, significant, interpretable, reliable, or "more sensitive" when comparator, measured quantity, and scope are not explicit.

Prefer measured evidence to value labels.

If the manuscript already contains quantitative evidence, recommend the concrete number(s) or bounded comparison instead of "competitive", "effective", or similar language.

## A09 — Opaque experimental scope

Trigger on "multiple datasets", "various settings", "extensive experiments", "N settings", etc. when the composition is unclear.

Prefer quantity + composition when supported, e.g. number of datasets, attacks, models, parameter grid, or repetitions.

## A10 — Claim-evidence mismatch

Trigger when the Abstract advertises a property that the paper does not directly establish, such as:

- lightweight;
- scalable;
- robust;
- generalizable;
- low-cost;
- data-efficient;
- small / limited / few trusted samples;
- few-shot.

Hard distinction for resource phrases such as `small trusted set`, `limited trusted clean data`, or `few trusted samples`:

1. If the phrase merely states an operating condition — for example, the defender has a fixed trusted set of a stated size — treat it as a neutral assumption and do not infer data efficiency.
2. If the wording presents scarcity itself as an advantage — for example, `only a small set is required`, `works with limited data`, or `suitable for low-resource deployment` — require direct supporting evidence such as trusted-set-size sensitivity, sample-efficiency analysis, or another bounded experiment.
3. Using one fixed trusted-set size does not by itself establish that the method is data-efficient or that the set is objectively `small`.
4. A later limitation that says the method `requires a limited number of trusted samples` does not retroactively validate an Abstract capability claim.
5. NEUTRAL-PREPOSITIONAL CONDITION: wording of the form `a framework ... with a small/limited trusted set` is a neutral operating condition when it merely names what data the defender has. In that form, and without scarcity-as-advantage markers such as `only`, `requires just`, `works with limited data`, `data-efficient`, or `suitable for low-resource deployment`, mark A10 checked-no-trigger rather than uncertain. Do not demand a trusted-set sensitivity study merely to justify the existence of that operating condition.

Recommend weakening/removing the property, rewriting it as a neutral operating assumption, or adding direct evidence.

## 5.2 Abstract mandatory execution

Pass 1 — high-recall ledgers:

- A01: every sentence.
- A02: full motivation block.
- A03: every explicit connector and summary bridge.
- A04: every problem against every major design.
- A05/A06: complete method narrative.
- A07: every new component.
- A08: every evaluative/comparative phrase.
- A09: every experiment-scope expression.
- A10: every headline advantage.
- X01: every nontrivial anaphoric reference.
- X02: every task/protection/utility/gap/mechanism causal statement.

Pass 2 — conservative adjudication:

- distinguish explicit text from expert inference;
- do not clear A04/A07 without exact bridge text;
- do not clear A08 merely because related experiments exist;
- do not turn a purely stylistic preference into a logic finding.

Final Abstract coverage must state Triggered / Checked-no-trigger / Uncertain for A01-A10 and X01-X02.

---

# 6. Introduction module

## 6.1 Core structure

Reconstruct:

Context / real-world problem
-> broad research or solution landscape
-> selected subproblem / method family
-> concrete limitation
-> research gap
-> proposed approach
-> contributions / evidence preview

## I01 — Abstract-Introduction role duplication

Compare Abstract motivation/problem propositions with Introduction paragraph 1.

Label each Introduction P1 proposition SAME / MIXED / NEW.

Compute approximate weighted overlap:

R = (SAME + 0.5 * MIXED) / total propositions.

Trigger I01 when:

- R is about 0.70 or higher; and
- at least three SAME/MIXED propositions replay the Abstract's chain in substantially the same order.

Compact-opening exception: when the supplied Abstract/Introduction comparison contains only a short opening block, trigger I01 with two strongly equivalent propositions if they cover essentially the whole supplied opening, preserve the same order, and add little or no NEW rhetorical work. Do not use this exception when the Introduction adds a distinct deployment perspective, task definition, causal mechanism, or other independent proposition.

Compression test:

Compress all repeated propositions to one bridge sentence. If little substantial new rhetorical work remains, trigger I01.

Do not treat a new citation, example, or noun as new rhetorical work unless it adds an independent proposition.

## I02 — Premature narrowing / broken broad-to-narrow funnel

Label paragraph scope:

- L0 domain/application;
- L1 threat/broad problem;
- L2 broad solution/research landscape;
- L3 selected family/subproblem;
- L4 specific limitation;
- L5 this paper's solution.

Trigger when the Introduction jumps from L1 to L3/L4 without enough L2 positioning, unless the scope restriction is explicitly justified.

Hard funnel pattern: `broad threat/risk -> one practical defense is [specific family]` is an L1 -> L3 jump when no broader defense landscape or explicit scope-selection rationale appears between them. Local coherence is not sufficient: the reader must also understand where the chosen family sits among the plausible defense directions. A sentence that first names the broader landscape and then narrows to the selected family is a negative control.

## I03 — Paragraph-edge logical discontinuity

For every P_i -> P_{i+1}:

1. Let X = final substantive proposition of P_i.
2. Let Y = first substantive proposition of P_{i+1}.
3. Classify edge:
   - EXPLICIT-CAUSAL;
   - EXPLICIT-SCOPE;
   - TOPIC-ONLY;
   - UNSTATED.
4. Ask: "Why does Y follow now?"

If the answer requires missing R, trigger I03:

X -> [missing R] -> Y

Transition words and repeated topic nouns do not by themselves establish the bridge.

Hard setting-to-limitation pattern: a paragraph that only defines defender access, trusted data, or the deployment setting does not by itself motivate a specific limitation of a downstream interface such as binary suspicious/non-suspicious decisions. Before introducing that limitation, the text should establish the missing interface premise — for example, that suspicious-sample detection is the mechanism connecting the trusted reference to model updating and that its output is what purification receives. Without that bridge, trigger I03 even when both paragraphs concern the same defense setting.

## I04 — Challenge-to-contribution closure

Trigger when the Introduction explicitly states numbered/parallel challenges, gaps, or research questions and later lists contributions, but the contribution list does not visibly close those challenges.

Build:

| Challenge/gap | Contribution(s) responding to it | Exact response wording | Status |

Status:

- closed explicitly;
- partial;
- many-to-one but explicit;
- contribution-only;
- challenge-only;
- inferable-only.

A contribution list is not a separate inventory. It should make the reader see how the paper answers the challenges already raised.

If there are two challenges and four contributions, do not require a 2x2 numerical match; require explicit logical grouping/closure.

## I05 — Task-setting placement and self-introduction timing

Trigger when the Introduction's target task/defense setting/objective is explained too late, or when an extra "This paper focuses on..." paragraph interrupts a chain that has already narrowed to the concrete gap.

Expected order:

target setting/objective established early enough
-> relevant limitations derived
-> concrete gap
-> proposed work introduced directly

Two hard patterns:

1. LATE-TASK-DEFINITION:
   the manuscript discusses limitations of a task before clearly defining that task's operational objective or defender capability.

2. GAP-THEN-BACKTRACK:
   the text has already established a specific gap, then backs up to re-explain the general task/setting instead of introducing the proposed approach.

3. LIMITATION-THEN-TASK-RESET:
   the text has already stated a concrete observation or limitation (for example, one static view cannot characterize diverse triggers) and then inserts `This paper focuses on [task]` plus a generic task objective. If that task definition is a prerequisite for interpreting the limitation, move it earlier; if it is not, introduce the proposed work directly instead of resetting the funnel.

Output the current order and the corrected rhetorical order.

## 6.2 Mandatory Introduction ledger

Create:

| Paragraph | Main function | Scope level | New beyond Abstract | Link from previous | Status |

If Abstract is available, also create:

| Abstract proposition | Introduction proposition | SAME/MIXED/NEW | I01 status |

Run:

- I01 on P1 when evaluable;
- I02 on the complete scope ladder;
- I03 on every paragraph edge;
- I04 on every explicit challenge/gap list vs contribution list;
- I05 on task-setting and self-introduction placement;
- X01-X02 throughout.

---

# 7. Related Work module

Related Work should position the literature and derive the research gap objectively. It is not the Method section and should not become a second Introduction.

## R01 — Taxonomy, title, and coverage consistency

Trigger when a subsection introduces a taxonomy (for example four defense categories) but:

- develops only one category without explaining the narrowing;
- the subsection title names one category while the paragraph claims to cover several;
- categories are mixed at different abstraction levels;
- terminology such as attack, defense, detection, mitigation, cleansing, repair, and intervention is used without clear parent-child or sibling relationships.

Taxonomy ledger:

| Category/term | Parent concept | Sibling concepts | Actually covered? | Title compatible? |

If the taxonomy says "four categories", either cover the four categories proportionally, split them, or explicitly state why the subsection narrows.

## R02 — Objective literature voice / premature "our method" positioning

Trigger when Related Work repeatedly says "our method", "our evaluation", "our framework", or explains the proposed method while still surveying prior work.

Preferred pattern:

prior work -> capability -> limitation/gap

Then, at the end of a subsection or the Related Work section, a short positioning sentence may explain how the current paper differs.

Hard rules:

1. Do not use the paper's own mechanism as the main evidence that prior work is limited.
2. A single concise current-paper positioning sentence is allowed after the paragraph or subsection has objectively established the prior-work behavior/capability and the comparison axis. It need not first prove a deficiency if the sentence merely states a high-level difference and does not infer that prior work is inadequate. Trigger R02 when the current-method sentence itself supplies an unstated limitation, requirement, or negative inference about prior methods, even if `our` appears only once.
3. When the same sentence also gives scoring operators, branch behavior, fusion logic, or other internal mechanics, R04 may co-trigger; R02 remains the primary voice/positioning issue when the objective gap has not yet been established.
4. EVALUATION-DESIGN LEAKAGE: a Related Work sentence such as `this motivates our evaluation`, `we evaluate under`, or `our experiments cover` normally shifts from literature synthesis into the current paper's experimental design. Flag R02 unless the sentence is necessary to state a high-level comparison boundary and does not enumerate the paper's scenarios, attacks, datasets, metrics, or evaluation protocol. Move detailed evaluation justification to the experimental setup.
5. SELF-CONTRAST-AS-GAP: a pattern such as `Prior methods do X. Our method does not require Y; instead it does Z` is not an objective literature gap when the prior sentence has not established that Y is actually an assumption or limitation of those methods. Trigger R02 even when the current-method sentence is concise; concision does not convert an author-supplied contrast into literature evidence.

## R03 — Literature-transition and term-relation discontinuity

For each paragraph/subsection edge, apply a Related-Work "why now?" test.

A valid edge should make one of these relations explicit:

- attack evolution -> need for defense coverage;
- defense landscape -> selected cleansing family;
- selected family -> known limitation;
- limitation -> next family or unresolved gap.

Trigger when "therefore" or a subsection change jumps between levels without establishing the relation, or when the reader must infer how two named research categories relate.

Hard specificity pattern: evidence or conclusions stated for a broad `defense` landscape do not automatically establish requirements for one narrower family such as training-data cleansing. A transition of the form `attack/trigger/label mapping affects defense results -> therefore cleansing methods must handle ...` needs an explicit bridge showing why that evidence applies to cleansing and how the named trigger properties affect its operational objective. Without that relation, trigger R03 even if the concluding requirement sounds plausible.

Output:

X -> [missing relation] -> Y

and name the relation needed: parent-child, contrast, narrowing, consequence, or complement.

## R04 — Method-detail leakage into Related Work

Trigger when Related Work contains implementation-level details of the proposed method:

- exact internal branch behavior;
- detailed scoring logic;
- algorithm sequence;
- component interactions;
- selection/fusion mechanism;
- design parameters better explained in Method.

Allowed:
one or two high-level positioning sentences needed to distinguish the paper from the nearest work.

Disallowed:
a mini-Method section embedded in literature review.

Recommendation:
move mechanism detail to Method and keep only the comparison axis or research gap.

## R05 — Subsection proportionality and compression

Trigger when a Related Work subsection is materially longer or more detailed than needed to establish its role in the literature argument.

Compression test:

1. Identify the subsection's required rhetorical output.
2. Remove examples/details that do not change that output.
3. If the same gap/position remains, recommend compression.

Do not flag detail merely because the subsection is long. Flag detail that does not change taxonomy, gap, or positioning.

## 7.1 Related Work execution

Before comments, create:

| Subsection | Claimed scope | Literature role | Taxonomy/coverage | Own-method leakage | Transition quality | Compression |

Coverage line:
R01-R05 + X01-X03 + relevant S-rules.

---

# 8. Section architecture and ownership

## S01 — Section-role ownership

Each section should perform its own job.

Default roles:

- Introduction: establish context, landscape, gap, approach, contributions.
- Related Work: organize prior literature and derive positioning/gaps.
- Problem Formulation / Threat Model: define task, actors, capabilities, assumptions, inputs/outputs, notation, evaluation objective if needed.
- Method: explain the proposed mechanism, components, algorithms, and implementation logic.
- Experiments: report setup, evidence, comparisons, ablations, and results.

Trigger when content clearly belongs to a later/earlier section.

Examples:

- detailed proposed-method mechanics in Related Work -> move to Method;
- method-specific operational steps in Problem Formulation -> move to Method;
- generic task definition repeated after the Introduction has already narrowed to the gap -> move earlier or compress.

## S02 — Section-opening and section-closing discipline

Trigger when:

- a section opening contains unnecessary meta prose before the substantive problem;
- a subsection ends with an unrelated preview of the author's method;
- a closing sentence does not prepare the next rhetorical move.

Prefer a substantive opening. Use a closing transition only when it carries a real logical relation.

Co-trigger rule: when the closing sentence of Related Work replaces a substantive literature/gap closure with author-side staging such as `our method has ...; Section 3 presents ...; Section 4 evaluates ...`, trigger S02 for the failed section closing. X03 may co-trigger for empty navigation and R02 may co-trigger for premature own-method voice; neither substitutes for S02 when the failure is specifically at the section boundary.

## S03 — Boundary between problem definition and solution design

Problem Formulation should define what must be solved, under what assumptions, and what output is required.

Method should define how it is solved.

Trigger when problem formulation starts justifying or detailing specific internal components before the problem/defender/attacker model is complete, or when Method must retroactively define basic task assumptions that should have appeared earlier.

Boundary ledger:

| Statement | Problem/assumption | Solution/design | Correct section? |

---

# 9. Professor-comment interpretation protocol

For every professor comment, produce:

1. Comment target: exact phrase/sentence and location.
2. Surface issue: what appears wrong locally.
3. Root cause: deeper logic, evidence, organization, or section-role problem.
4. Primary rule ID.
5. Secondary rule ID(s), if useful.
6. Revision criterion: what must become true for the comment to count as resolved.

Do not reduce a structural comment to copy-editing.

When the professor says "logic does not hold", explicitly reconstruct the attempted inference and missing premise.

When the professor says "do not use this/they/it", first test X01 rather than treating it as a universal ban on pronouns.

---

# 10. Revision comparison protocol

When earlier and revised versions exist, label each prior issue:

- Resolved.
- Partially resolved.
- Reframed.
- Unresolved.
- New issue introduced.

A lexical change is not enough. The underlying logic, role, evidence, or section placement must be fixed.

Hard revision check — lexical weakening is not resolution:

- `small` -> `limited`;
- `effective` -> `useful`;
- `robust` -> `stable`;
- replacing one causal connector with a softer connector.

If the underlying proposition and its evidence gap remain the same, label the issue Partially resolved, Reframed, or Unresolved rather than Resolved. Re-run the original rule against the revised proposition, not merely against the revised adjective or connector.

---

# 11. Required output format

Use only the parts applicable to the supplied sections.

## A. Overall diagnosis

One concise structural judgment per in-scope section.

## B. Argument / section reconstruction

- Abstract: B -> P -> G -> M -> R -> C.
- Introduction: L0-L5 scope chain.
- Related Work: taxonomy/landscape -> limitations -> unresolved gap.
- Problem/Method boundary: task/assumptions -> output -> method.

## C. Comment-by-comment analysis

| Location | Original text | Professor comment | Surface issue | Root cause | Primary rule | Revision criterion |

## D. Required ledgers

Include the module-specific ledgers required above.

## E. Claim-evidence consistency

| Claim | Evidence in manuscript | Supported / weak / unsupported | Recommendation |

## F. Revision priorities

- Must fix: invalid logic, unsupported claim, wrong section ownership, broken challenge-contribution closure, broken taxonomy.
- Should fix: weak transition, premature self-positioning, concept overload, unnecessary repetition, compression.
- Optional polish: wording, syntax, concision, terminology after logic is fixed.

## G. Suggested rewrite

Only rewrite when the user requests revision text or when replacement text is explicitly part of the task.

Do not invent new experiments, claims, contributions, datasets, baselines, or causal conclusions.

---

# 12. Review priorities

1. Logical validity and concept-role correctness.
2. Section ownership and rhetorical order.
3. Problem/challenge -> method/contribution correspondence.
4. Claim-evidence consistency.
5. Literature taxonomy and transition coherence.
6. Information density and necessity.
7. Experimental clarity and quantitative evidence.
8. Explicit referents and local readability.
9. Grammar and stylistic polish.

Do not begin with grammar when a higher-level problem exists.

---

# 13. Professor-style patterns learned from the current evidence set

The current annotation set supports the following recurring preferences:

- Prefer explicit technical nouns over ambiguous short references when the referent is not uniquely obvious.
- Do not let a connector word or pronoun carry missing logic.
- Distinguish the privacy/security requirement from the utility/fidelity objective instead of conflating them in one causal claim.
- In Abstract results, prefer direct quantitative evidence to vague author-side evaluation.
- The Introduction should not replay the Abstract opening nearly verbatim.
- Explicitly close stated challenges/research questions with corresponding contributions.
- Define the target task/defense setting early enough; once the concrete gap is reached, introduce the proposed work rather than backtracking to generic setup.
- In Related Work, keep taxonomy levels and section titles coherent.
- Related Work should describe prior work and gaps objectively; minimize "our method/our evaluation" until a concise closing positioning sentence.
- Do not put detailed proposed-method mechanics in Related Work or Problem Formulation.
- Compress literature discussion that does not change the taxonomy, gap, or positioning.
- Remove empty meta-navigation unless it performs a necessary structural function.

These are learned preferences from the supplied manuscripts and annotations, not universal laws. Future professor comments may refine or override them.

---

# 14. Non-invention constraints

The skill must not:

- invent professor intent when the annotation is ambiguous;
- invent experimental evidence;
- assume a design solves a stated problem without textual support;
- infer superiority from one metric unless the manuscript makes that comparison;
- turn a local professor preference into a universal academic rule without marking it as a learned preference;
- strengthen a claim beyond source evidence;
- move text between sections without explaining which section role is violated.

When uncertain, state:

"The current materials do not establish this point clearly."

Then specify what evidence or bridge would be needed.

---

# 15. Regression protocol

Keep regression fixtures and expected labels outside this reviewer-facing Skill file.

Blind regression should include:

- positive and negative controls for X01 and X02;
- Abstract cases for A01-A10;
- Introduction cases for I01-I05;
- Related Work cases for R01-R05;
- section-boundary cases for S01-S03;
- repeated borderline runs to assess stability;
- synthetic controls that change vocabulary while preserving argument structure.

For professor-comment regression, score:

- comment recall: did the rule set identify the professor's underlying issue?
- rule precision: did it avoid flagging nearby text that the comment does not support?
- section-placement accuracy: did it identify where the material belongs?
- explanation quality: did it reconstruct the missing causal or rhetorical link rather than merely echoing the comment?

Decision-boundary guardrails for blind runs:

- A02: ordinary scientific uncertainty or one local hedge is not enough; the motivation itself must depend on a stack of weak hypothetical premises.
- A04: do not require one-to-one cardinality. Do not trigger when each stated problem is explicitly answered, even if one design answers multiple problems or one problem requires multiple designs.
- A05: naming several components is not enough when the core mechanism is already clear and each name is interpretable in that mechanism.
- A06: sequence words such as `first` and `then` are not enough when purpose and consequence are also explicit.
- A07: do not flag a component whose nearest need and output are stated locally.
- A09: a number such as `48 settings` is acceptable when its composition is stated in the same or immediately adjacent sentence.
- S01/S03: a high-level preview or cross-reference is not a section-ownership violation; trigger only when substantive solution detail occupies the wrong section or task assumptions are deferred into Method.
- S02: a section-closing sentence is acceptable when it carries a substantive narrowing, contrast, or gap; do not require deletion merely because it prepares the next section.

The evaluator may compare blind outputs with withheld annotations only after each run has finished.
