---
name: academic-paper-review
description: Reviews academic-paper abstracts for argument structure, problem-method mapping, component motivation, experimental scope, and claim-evidence consistency; supports professor-comment interpretation and blind regression review.
---

# Academic Paper Review Skill — Abstract Module v0.7 Candidate

## 1. Purpose

This skill reviews the abstract of an academic paper using the author's manuscript and, when available, supervisor/professor annotations as the primary evidence.

The skill does not merely polish language. It reconstructs the abstract's argument chain, identifies why a comment was triggered, distinguishes surface wording problems from deeper reasoning problems, and converts repeated reviewer preferences into reusable review rules.

## 2. Inputs

Required when available:

- Original manuscript PDF.
- Revised manuscript PDF, if a revision exists.
- Professor/supervisor annotated PDF or comment PDF.

The skill must treat the uploaded materials as the source of truth. It must not silently add claims, experimental evidence, motivations, or contributions that are not supported by the manuscript or comments.

### Blind regression mode

When the task explicitly requests a blind review or regression test, the manuscript text must be analyzed without professor/supervisor annotations, comments, highlighted notes, review summaries, or any other hidden review material. In blind mode:

- use only the manuscript text and any explicitly permitted non-annotation evidence;
- do not open, recover, infer, quote, or search for annotations/comments;
- do not use prior reviewer comments as evidence for the current decision;
- report findings independently from the A01-A10 rules;
- if sanitized manuscript text is supplied, treat that sanitized text as the complete review input.

Blind-mode results may later be compared with withheld reviewer comments by the evaluator, but the reviewer itself must not see those comments during the run.

## 3. Scope of this module

This module handles the abstract only.

It should identify:

- background and problem framing;
- research motivation;
- problem-to-solution correspondence;
- method narrative;
- experimental-scope reporting;
- claim-evidence consistency;
- unnecessary content;
- overclaiming and subjective evaluation;
- logical gaps and hidden premises;
- whether revisions actually address prior comments.

Language polishing is secondary and must occur only after the logical structure is reviewed.

## 4. Core review model

Reconstruct the abstract as:

Background (B) -> Problem (P) -> Gap (G) -> Method (M) -> Results (R) -> Conclusion (C)

Then inspect the five transitions:

B -> P: Does the stated background actually establish the research problem?
P -> G: Does the manuscript explain what is missing or inadequate?
G -> M: Does each major design choice clearly respond to the stated gap?
M -> R: Do the reported experiments test the claimed method contributions?
R -> C: Is the conclusion supported by the reported evidence without overclaiming?

If a transition requires the reader to infer an unstated premise, mark it as a logical gap.

## 5. Abstract issue taxonomy

### A01 — Low-information, removable, merge-only, or generic-definition sentence

Trigger:
A sentence is correct but its UNIQUE INFORMATION is too small to justify a standalone abstract sentence. A01 is about information value, not grammar and not merely whether two sentences can be combined syntactically.

Run two separate tests.

Test A — deletion test:
Delete the sentence, minimally repair anaphora (for example "this reference"), and ask whether the paper-specific problem, gap, mechanism, contribution, result, or evidence boundary becomes materially weaker. If not, trigger A01.

Test B — standalone-worthiness test:
If deletion loses a small but necessary proposition, ask whether that proposition is only a definition, bridge, or reference-establishment statement that the adjacent sentence immediately operationalizes. If the full unique content can be absorbed into the adjacent sentence as a short modifier/clause without loss of argument structure, trigger A01 as a merge-only candidate.

Do NOT trigger A01 merely because a sentence is grammatically mergeable. A sentence carrying two or more independent constraints, assumptions, result dimensions, or access conditions normally has enough unique information to stand alone even if it could be combined stylistically. Likewise, a method-introduction sentence that names the proposed method AND states its core principle may perform the G -> M transition and should not be flagged solely for mergeability.

Do not treat anaphoric dependence as evidence of information value. If a later sentence says "this reference", "this setting", "this score", or similar, mentally replace the pronoun with its antecedent before testing deletion.

Apply a generic-background test to opening sentences. A textbook-style definition of the field, threat, or task is not automatically necessary. If the following paper-specific problem remains fully understandable after deleting the definition, trigger A01.

HARD PATTERN A01-GENERIC-DEFINITION:
If an opening sentence mainly defines a standard threat/task behavior and the next sentence states the paper-specific difficulty, perform repaired deletion as the default. If replacing an anaphor such as "such samples/this attack" with the concrete noun leaves the paper-specific problem fully intelligible, classify the definition as DELETE-CANDIDATE unless one of the exceptions below applies. Do not preserve it merely because it is useful background.

CAUSAL-ANTECEDENT EXCEPTION:
KEEP the otherwise generic-looking sentence when it supplies the only concrete behavioral premise that the immediately following problem/risk sentence actually reasons from. This exception requires logical dependence, not mere referential dependence.

Use this test:
1. Rewrite/remove simple anaphora first.
2. Delete the candidate sentence.
3. Ask whether the next problem/risk claim still has its concrete causal antecedent, rather than merely still having a named entity to refer to.
4. If the next claim explicitly summarizes or reasons from a specific behavior/property introduced only in the candidate sentence, KEEP it as a causal antecedent and review the next bridge under A03.
5. If the next sentence remains a complete paper-specific problem using only the noun identity (e.g. replacing "such samples" with "poisoned samples"), the exception does not apply and the generic definition remains a DELETE-CANDIDATE.

Examples of the distinction:
- Referential-only dependency: "Backdoors do X... Such samples are hard to cleanse because their loss/features are not stable anomalies." If the cleansing difficulty remains intact after replacing "such samples" with "poisoned samples", the definition is deletable.
- Causal dependency: "A backdoored model behaves normally on clean inputs but changes under a hidden trigger. [Next sentence] This trigger-dependent behavior creates deployment risk." The behavior sentence supplies the concrete premise consumed by the risk claim; keep it, while separately testing whether the risk bridge is logically explicit under A03.

To invoke the exception, quote the exact downstream phrase that consumes the candidate sentence's specific behavior/property. A pronoun alone is insufficient.

HARD PATTERN A01-REFERENCE-ESTABLISHMENT:
If a sentence's sole substantive proposition is of the form "trusted/clean/reference samples establish/define normal behavior, a reference, or a baseline" and the immediately adjacent sentence operationalizes that same reference by measuring deviation, distance, risk, or comparison against it, classify the standalone sentence as MERGE-ONLY. Exception: KEEP only if the reference-establishment sentence contains an independent construction method, constraint, quantitative setting, or contribution that cannot be preserved as a short clause in the operational sentence.

These hard patterns take priority over a vague judgment that a sentence is "helpful context" or "a necessary premise." The reviewer must cite the exception if overriding a hard pattern.

For stability, explicitly count unique propositions in each candidate sentence:
- 0 unique propositions after repaired deletion -> delete candidate.
- 1 narrow proposition that is immediately consumed/operationalized by an adjacent sentence -> merge-only candidate.
- 2+ independent substantive propositions -> normally not A01; evaluate under other rules instead.

Review questions:
1. After repaired deletion, exactly which unique proposition disappears?
2. Is that proposition paper-specific and necessary, or generic/background knowledge?
3. Is it immediately operationalized by the next/previous sentence so that a short clause preserves everything?
4. How many independent substantive propositions does the sentence carry?
5. Does the sentence perform a genuine discourse transition or impose independent constraints that justify a standalone sentence?

Output:
Classify each sentence as KEEP / MERGE-ONLY / DELETE-CANDIDATE. For MERGE-ONLY or DELETE-CANDIDATE, state the exact unique proposition and why it does not justify a standalone sentence.

### A02 — Motivation built on a weak conditional scenario

Trigger:
Words such as may, might, could, often, potentially, possibly, or "under this setting" are used to carry the main research motivation.

Do not flag uncertainty mechanically. Distinguish uncertainty about an observation from uncertainty that defines the entire reason the paper is needed.

HARD PATTERN A02-COMPOUND-HYPOTHETICAL-SETTING:
Trigger A02 when the abstract builds the method's motivating setting from two or more hedged existence/prevalence/resource assumptions (for example "models may contain X", "deployers often have only Y", "users may lack Z") and then immediately proposes the method "under this setting" or equivalent wording, without first establishing an objective deployment/technical chain that exists independently of how often those assumptions happen.

The concern is not the individual words may/often. The concern is this structure:
hedged condition A + hedged condition B (+ condition C) -> "under this setting" -> we propose the method.
This makes the method appear useful only if the author's chosen scenario happens to hold.

A02 should NOT trigger merely because a technical property is uncertain across samples/attacks, such as "poisoned samples may not form stable anomalies", when the sentence describes a concrete failure mode of an already established task. It also should not trigger when the abstract first states an objective deployment constraint, such as "the released artifact does not reveal the training process", and uses uncertainty only to describe attack behavior within that objective constraint.

For every motivating hedge, label it as one of:
- EXISTENCE/PREVALENCE assumption;
- RESOURCE/ACCESS assumption;
- TECHNICAL VARIABILITY statement;
- ordinary epistemic caution.

Review questions:
1. If the hedged prevalence/existence claims are removed, does an objective problem still remain?
2. Is "under this setting" merely naming a scenario assembled by the authors, or does it follow from an externally grounded deployment constraint?
3. Are multiple hedges jointly carrying the P/G transition?
4. Can the motivation be rewritten as an objective chain such as deployment condition -> audit/technical limitation -> security/engineering consequence -> need for the method?

Output:
When triggered, show the current conditional chain and the missing objective chain. Do not suggest simply deleting may/often; explain what objective deployment or technical premise must replace them.

### A03 — Logical connector or abstract-summary bridge without sufficient logic

Trigger:
Because, therefore, thus, hence, consequently, or equivalent expressions connect two statements that do not follow directly OR appear to follow only after the author compresses prior concrete statements into newly introduced abstract labels.

A connector can be formally plausible and still fail this rule. Pay special attention when the antecedent contains newly coined summary nouns or nominalized labels such as "opaque provenance", "trigger-dependent behavior", "representation abnormality", "security concern", or similar abstractions that were not established as terms beforehand. The reviewer must test whether these labels clarify the causal mechanism or merely force the reader to decode earlier sentences again.

HARD PATTERN A03-RENAMING-BRIDGE:
If a sentence (i) introduces one or more new abstract/nominalized labels that merely rename facts stated in the immediately preceding sentence(s), and (ii) uses therefore/thus/hence/consequently or an equivalent summary move to derive a problem/risk/conclusion, trigger A03 unless the new labels add an explicit causal mechanism that was absent before. Logical plausibility is not enough. The issue is that "concrete facts -> new labels -> conclusion" forces the reader to decode the labels before seeing the causal relation.

To override this hard trigger, quote the exact words in the new labels/sentence that add a mechanism rather than simply summarize prior facts. If no such words can be quoted, A03 is triggered.

Review questions:
1. Can a first-time reader derive Q from the concrete preceding facts without supplying an unstated premise R?
2. Does the connector rely on newly introduced abstract nouns that rename previous facts rather than state the causal mechanism directly?
3. If those abstract labels are expanded back into the preceding concrete facts, is the causal relation still explicit and economical?

Output:
For a missing premise, show P -> [missing premise] -> Q. For an abstract-summary bridge, show "concrete facts -> newly coined labels -> connector -> conclusion" and explain whether the labels should be removed in favor of a direct causal sentence.

### A04 — Problem-solution mapping failure

Trigger:
The abstract states one or more technical problems/gaps and later introduces major designs, but the correspondence is not EXPLICITLY TRACEABLE in the abstract text.

Passing A04 requires more than "a knowledgeable reader can infer the relationship." For every major Problem -> Design pair, the reviewer must be able to cite textual evidence from the abstract that establishes BOTH:
1. the target difficulty/deficiency; and
2. why this design addresses that specific difficulty.

Build this matrix:

Problem/gap | Design | Exact bridge text | Mechanistic reason stated in abstract | Status

Status rules:
- Explicit: the abstract itself states or locally signals the correspondence and mechanism.
- Inferable-only: the pairing is plausible from domain knowledge or distant context, but no local bridge/mechanistic reason is stated. Trigger A04.
- Ambiguous: more than one problem/design pairing is possible. Trigger A04 strongly.
- Unmapped: a major problem or design has no counterpart. Trigger A04 strongly.

When multiple problems and multiple designs exist, do not pass the mapping merely because a one-to-one assignment can be guessed after reading the whole abstract. The intended pairing must be recognizable without reverse engineering the method.

Output:
For every non-Explicit pair, quote the problem text and design text, state what bridge is missing, and give a revision criterion that would make the mapping explicit.

### A05 — Concept overload

Trigger:
Several new method concepts are introduced in a short span, especially if the reader must remember many unfamiliar modules before understanding the main mechanism.

Review question:
Can the method be explained first as one causal mechanism before naming implementation components?

Output:
Identify which concepts are core and which can be delayed, merged, or removed from the abstract.

### A06 — Procedure listing instead of mechanism explanation

Trigger:
The method paragraph is dominated by "first / then / next / finally" style sequencing.

Review question:
Does each step explain why it exists and what problem it solves, or only what happens next?

Output:
Rewrite the method logic as purpose -> operation -> consequence.

### A07 — Orphan or locally unmotivated method component

Trigger:
A module name or technical component appears before the abstract provides an EXPLICIT LOCAL BRIDGE explaining why it is needed and how it connects to the current mechanism.

The standard is stricter than semantic inferability. A reader being able to guess a component's purpose from domain knowledge, from a problem stated several sentences earlier, or from the component's name is NOT enough. Each major component should be locally anchored by wording that makes its target and role visible at first reading.

For every new component, build a component ledger:
Component | Nearest preceding problem/need | Exact local bridge text | Output/signal it contributes | Status

Status rules:
- Explicitly motivated: a nearby clause/sentence states the specific need, links the component to that need, AND gives enough mechanism to explain why this component is an appropriate response.
- Role-only bridge: the sentence only says what evidence the component "captures/measures/probes" (for example local evidence or high-frequency evidence) but does not connect that evidence type to the previously stated difficulty or explain why this view is needed. Trigger A07. A role label is not a rationale.
- Distant-only: a relevant problem exists earlier, but the component appears without a local bridge. Trigger A07.
- Name-inferable: the purpose is guessed mainly from the component name (e.g. spatial occlusion sounds local, high-frequency suppression sounds frequency-related). Trigger A07 strongly.
- Pipeline-orphaned: the component's output is not connected to what came before/after. Trigger A07 strongly.

HARD PATTERN A07-BRANCH-INTRODUCTION:
When a generic operation such as "controlled perturbations" or "multi-view evidence" is followed by named branches/modules, each branch must be introduced with an explicit reason tied to a stated difficulty. "Branch A captures local evidence, whereas Branch B captures non-local evidence" is insufficient by itself if the abstract has not locally said why both evidence types are needed for the stated trigger/problem heterogeneity. In that case classify both branches as ROLE-ONLY or DISTANT-ONLY, not Explicitly motivated.

Review questions:
1. What exact prior words establish the specific need for this component?
2. What exact nearby words say that this component addresses that need?
3. Could a first-time reader identify the pairing without using domain knowledge or reverse engineering later sentences?
4. Is the component's output/significance connected to the next pipeline step?
5. If the component were renamed to a neutral label (Module A), would its role still be obvious from the abstract? If not, the prose is relying on the name to carry logic.

Output:
Mark non-explicit components as "locally unmotivated" and quote the missing bridge. Do not clear A07 merely because a plausible mapping exists somewhere in the abstract.

### A08 — Subjective, vague-comparative, or unsupported evaluation

Trigger:
Words such as effective, efficient, robust, superior, promising, interpretable, significant, sensitive, reliable, or comparative forms such as "more sensitive" are used as author evaluation rather than as a precisely defined measured finding.

Do not ban these words mechanically, but do not let later experiments retroactively make a vague label precise. A comparative/evaluative phrase in the abstract must have an identifiable comparator, measured quantity, and scope. "More sensitive" must answer: more sensitive than what, to what signal, and by what observable criterion? "Interpretable" must identify what is interpretable and what evidence or mechanism makes that interpretation traceable.

Review questions:
1. Is the phrase a measured finding or an author-side label?
2. For a comparative adjective, are comparator + metric/observable + evaluation scope explicit?
3. Can the phrase be replaced by the concrete ablation, comparison, or controlled result already present in the manuscript without losing information?
4. Does the evidence establish the named property itself, or only a downstream performance change from which the property is being inferred?

Output:
Prefer evidence over labels. If comparator/metric/property is undefined, flag the wording even when a related ablation exists; recommend stating the observable effect instead.

### A09 — Opaque experimental scope

Trigger:
The abstract reports "N settings", "multiple datasets", "various models", "extensive experiments", or similar scope descriptions without enough composition information.

Review question:
Would a first-time reader know what the experimental count is made of?

Output:
Prefer quantity + composition, e.g. dataset count x attack count x model count when supported by the paper.

### A10 — Claim-evidence mismatch

Trigger:
The abstract presents a property as a key strength, such as small trusted set, lightweight, low-cost, scalable, robust, generalizable, few-shot, or data-efficient.

Review question:
Does the method or experiment explicitly establish this property?

Output:
If not, mark the wording as an unsupported selling point and recommend weakening, removing, or adding evidence.

## 5.1 Mandatory exhaustive execution protocol

The issue taxonomy is not a menu. Execute every applicable check systematically. Use a TWO-PASS process so that detection is separated from adjudication.

### Pass 1 — high-recall candidate ledger

Number the abstract sentences S1...Sn and create internal ledgers before deciding which issues are severe.

Sentence ledger:
- Run A01 on every sentence and classify KEEP / MERGE-ONLY / DELETE-CANDIDATE with unique-proposition count.
- Run A03 on every explicit connector and every abstract-summary bridge.
- Record every capability/advantage/evaluative phrase for A08/A10.

Problem-design ledger:
- Extract every stated problem/gap P1...Pk.
- Extract every major design/component D1...Dm.
- For each plausible P-D pair, quote the exact bridge text. If there is no bridge text, mark INFERABLE-ONLY rather than silently passing it.

Component ledger:
- For every D1...Dm, quote the nearest preceding need and the local bridge that motivates it.
- If purpose is understood mainly from the component name or domain knowledge, mark NAME-INFERABLE or DISTANT-ONLY.

Experiment ledger:
- Record each experiment-scope phrase for A09 and decompose counts where the manuscript supports it.
- Record each headline property/advantage for A10 and locate direct evidence in the manuscript.

### Pass 2 — conservative adjudication

Before free-form judgment, apply all HARD PATTERN rules. A hard-pattern match is the default decision. It may be overridden only when the reviewer quotes the exact exception evidence required by that hard rule. This priority is intended to reduce run-to-run drift on borderline sentences.

Convert ledger entries into findings using these rules:
- Missing explicit bridge is not a clean pass. It is at least a possible concern; if multiple problems/designs are present or the module appears abruptly, elevate to a strong hit.
- Do not promote a sentence to A01 merely because it can be combined grammatically; require the A01 unique-information criteria.
- Do not downgrade a vague evaluative phrase merely because related experiments exist; the named property itself must be defined and evidenced.
- Distinguish "text explicitly establishes X" from "X can be inferred by an expert." The latter is weaker and should be reported when this reviewer style emphasizes not making readers infer logic.

### Coverage requirements

- A01: every sentence.
- A02: the motivation block.
- A03: every causal/summary bridge.
- A04: every major problem/gap against every major design.
- A05/A06: the complete method narrative.
- A07: every newly introduced component.
- A08: every evaluative/comparative phrase.
- A09: every experiment-scope expression.
- A10: every headline capability/advantage claim.

Do not terminate because several severe findings have already been identified. The final coverage matrix must state Triggered / Checked-no-trigger / Uncertain for every A01-A10 rule and cite the exact sentence(s) checked. For every Checked-no-trigger decision under A04 or A07, include the exact bridge text that justifies the pass; if no such text can be quoted, it cannot be Checked-no-trigger.

## 6. Professor-comment interpretation protocol

For every professor comment, produce five layers:

1. Comment target: exact sentence/phrase and location.
2. Surface issue: what appears wrong locally.
3. Root cause: the deeper argument, structure, evidence, or information-design problem.
4. Generalizable rule: which abstract-review rule this comment represents.
5. Revision criterion: what must become true for the comment to count as resolved.

Do not treat the professor's wording as a simple copy-edit instruction when the surrounding context shows a deeper reasoning issue.

## 7. Revision comparison protocol

When both an earlier and a revised abstract exist:

For each prior comment, label the revision as:

- Resolved: the underlying problem is removed.
- Partially resolved: wording changed but the logical or evidential issue remains.
- Reframed: the original issue was avoided by restructuring the passage.
- Unresolved: the same underlying problem remains.
- New issue introduced: the revision fixes one problem but creates another.

A lexical change is not sufficient evidence of resolution.

## 8. Required output format

### A. Overall diagnosis

Give a concise judgment of the abstract's main structural weakness, not a generic language-quality statement.

### B. Argument-chain reconstruction

Show:
B -> P -> G -> M -> R -> C

Mark missing or weak links explicitly.

### C. Comment-by-comment analysis

Use columns:

Location | Original text | Professor comment | Surface issue | Root cause | Rule ID | Revision criterion

### D. Problem-solution matrix

Use columns:

Problem/gap | Method response | Evidence in abstract | Mapping quality

### E. Claim-evidence consistency table

Use columns:

Claim | Evidence in manuscript | Supported / weak / unsupported | Recommendation

### F. Revision recommendations

Separate into:

- Must fix: logical break, unsupported claim, incorrect mapping, misleading experiment description.
- Should fix: concept overload, unnecessary sentence, weak transition.
- Optional polish: wording, concision, syntax, terminology.

### G. Suggested rewrite

Only produce a rewritten abstract or replacement sentences if requested, or if the task explicitly asks for revision text.

The rewrite must not introduce new experimental claims, new contributions, new datasets, new baselines, or new causal conclusions that are absent from the source manuscript.

## 9. Review priorities

Priority order:

1. Logical validity.
2. Problem-method correspondence.
3. Claim-evidence consistency.
4. Information density and necessity.
5. Experimental clarity.
6. Overclaiming control.
7. Language and style.

Do not begin with grammar corrections when a higher-level problem exists.

## 10. Current professor-style patterns learned from the provided examples

From the supplied DPRC and MARU examples, the observed review preference is:

- do not force the reader to infer why a module exists;
- do not let a connective word substitute for an actual causal chain;
- do not explain basic terminology if it consumes abstract space without advancing the argument;
- do not turn implementation detail into the main method narrative;
- do not advertise a property unless the paper actually demonstrates it;
- do not use author-side value labels when direct evidence is available;
- make experiment counts interpretable;
- when multiple problems are stated, make their corresponding designs traceable.

These patterns are evidence from the current example set, not universal academic-writing laws. If future professor comments contradict or refine them, update this section rather than forcing new examples into old rules.

## 11. Non-invention constraints

The skill must not:

- invent missing professor intent when the comment is ambiguous;
- invent experimental evidence to justify an abstract claim;
- assume a design solves a stated problem unless the manuscript supports the relationship;
- infer superiority from a single metric without the paper making that comparison;
- treat stylistic preference as a universal rule when it appears specific to this professor or manuscript;
- rewrite a claim more strongly than the source evidence supports.

When uncertain, output: "The current materials do not establish this point clearly." and identify what evidence would be needed.

## 12. Test case for this module

Use the MARU abstract pair as a regression test.

Expected behavior:

- detect that "48 settings" is less informative than a decomposition into datasets, attacks, and models;
- identify that introducing many method terms without a causal spine creates concept overload;
- recognize that "small trusted clean set" can become an unsupported headline claim if no corresponding evidence is established;
- distinguish a wording change from a genuine repair of the underlying logic;
- treat the revised abstract as improved where it converts module listing into a reference -> deviation -> risk -> threshold -> margin -> purification chain, while still allowing new logical issues to be flagged.
