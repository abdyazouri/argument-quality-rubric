# Argument Quality Assessment: A Rubric-Based Methodology Combining Classical Argumentation Theory and Multi-Criteria Decision Analysis

### Abstract

Assessing the quality of a written argument is a recurring problem across debate education,
policy analysis, peer review, and increasingly, the evaluation of AI-generated reasoning. Purely
holistic ("how good is this argument, 1-10?") scoring is fast but not reproducible: two reviewers
rarely agree on what "good" meant, and neither can explain their score to a third party. This
document proposes a rubric-based methodology that (1) decomposes argument quality into
dimensions grounded in classical argumentation theory -- Toulmin's model of argument structure and
Walton's argumentation schemes -- and (2) aggregates those dimensions into a single defensible
score using techniques from Multi-Criteria Decision Analysis (MCDA). A companion repository,
[`rubric-eval-agreement`](https://github.com/IV7L/rubric-eval-agreement), implements and
empirically validates the measurement side of this methodology (inter-rater agreement, quadratic
weighted kappa, bootstrap confidence intervals) on a synthetic dataset built in the same spirit as
the rubric proposed here.

This document contains no code. It is a methodology write-up: what to measure, why those
measurements are theoretically grounded, and how to combine them defensibly.

---

## 1. Introduction

Argument quality is not one thing. A long-standing line of work in computational argumentation
(Wachsmuth et al., 2017) decomposes it into three broad dimensions:

- **Cogency** -- the logical quality of the argument itself: are the premises acceptable, relevant,
  and sufficient to support the claim?
- **Effectiveness** -- the rhetorical quality: does the argument persuade its intended audience?
- **Reasonableness** -- the dialectical quality: does the argument engage honestly with the
  disagreement it is part of, including opposing positions and their strongest form?

A rubric that only measures one of these (most commonly, only effectiveness, since persuasiveness
is easiest to proxy with surface features) systematically rewards arguments that are easy to read
but logically weak, or logically sound but dialectically evasive. The methodology below treats all
three as first-class, separately scored dimensions.

## 2. Theoretical foundations

### 2.1 Toulmin's model (Cogency)

Toulmin (1958) decomposes an argument into six interlocking parts: **Claim** (the conclusion being
argued for), **Grounds/Data** (the evidence offered), **Warrant** (the usually-implicit principle
connecting grounds to claim), **Backing** (support for the warrant itself, when the warrant is
contested), **Qualifier** (the strength with which the claim is asserted -- "probably", "almost
certainly"), and **Rebuttal** (the conditions under which the claim would not hold).

For rubric design, the Toulmin model gives a direct, checkable decomposition of cogency: a scorer
can ask, independently, "is the claim clearly stated?", "is the warrant stated or left to be
assumed, and is it sound?", "does the argument acknowledge its own rebuttal conditions?" --
rather than forming one impression of "logical quality" all at once.

### 2.2 Walton's argumentation schemes (Reasonableness)

Walton (1996) catalogues recurring patterns of defeasible, everyday reasoning -- "argument from
expert opinion", "argument from consequences", "argument from analogy", and dozens more -- each
paired with a fixed set of **critical questions** that test whether that specific pattern was used
soundly. An argument from expert opinion, for instance, is only as strong as its answers to
questions like "is the expert actually an expert in this field?" and "do other experts agree?".

This is the piece that operationalizes reasonableness: instead of a generic "did the argument
consider counterarguments?" item, the scorer first identifies which scheme the argument is using,
then applies that scheme's specific critical questions. A scheme-blind rubric cannot do this; it
can only ask generic questions that apply weakly to every argument and strongly to none.

### 2.3 Multi-Criteria Decision Analysis (Aggregation)

Toulmin and Walton together produce several separately-scored criteria (cogency sub-scores,
scheme-specific critical-question answers, a reasonableness score, an effectiveness score). MCDA
(Saaty, 1980; Belton & Stewart, 2002) is the standard toolkit for turning multiple, sometimes
competing criteria into one defensible decision or score:

- **Weighted sum**: each criterion gets an explicit weight (ideally elicited from stakeholders, not
  set unilaterally), and the final score is their weighted average.
- **Sensitivity analysis**: because weights are inherently somewhat subjective, MCDA practice is to
  re-run the aggregation under several plausible weight sets and report whether the ranking of
  arguments is stable -- a defensible final score reports this robustness check rather than hiding
  behind a single fixed weight vector.
- **Outranking methods** (e.g. ELECTRE, PROMETHEE) as an alternative to weighted sums when criteria
  are not easily comparable on a common scale, or when a small score difference should not flip a
  ranking.

The reason to borrow MCDA here, rather than inventing an aggregation formula, is that the "how do
I weigh several good-but-not-perfectly-comparable criteria" problem is already a mature field with
known failure modes (e.g. rank reversal) and known mitigations.

## 3. Rubric design

Combining the two theories above, each criterion is scored 0-4 against a written behavioral
anchor -- the same anchor-based design used in established rubric-scoring literature (e.g. the
ASAP-AES essay-scoring guides), chosen specifically because anchors make a score defensible to a
third party: "this got a 2, not a 3, because the warrant was present but not explicitly connected
to the grounds" is a checkable claim, "this felt like a 2" is not.

| Dimension | What it checks | Anchor style (abbreviated) |
|---|---|---|
| **Claim clarity** (Cogency) | Is the central claim identifiable and unambiguous? | 0: no identifiable claim -- 4: claim is explicit, unambiguous, and appropriately qualified |
| **Grounds-warrant connection** (Cogency) | Is the link between evidence and claim stated, or only assumed? | 0: grounds stated with no connection to the claim -- 4: the warrant is explicit and sound |
| **Rebuttal awareness** (Cogency) | Does the argument acknowledge conditions under which it would not hold? | 0: claim presented as unconditionally true -- 4: explicit, accurate rebuttal conditions stated |
| **Scheme-critical-question coverage** (Reasonableness) | Of the critical questions attached to the argument's identified scheme, how many are addressed? | 0: none addressed -- 4: all materially relevant questions addressed |
| **Opposing-position engagement** (Reasonableness) | Does the argument engage the strongest form of the opposing position, or a weaker version (straw-manning)? | 0: opposing position ignored or misrepresented -- 4: strongest opposing position engaged directly |
| **Audience-appropriate clarity** (Effectiveness) | Is the argument expressed in a way its stated audience could follow? | 0: inaccessible to the stated audience -- 4: clear and well-structured for that audience |

Each dimension is scored independently; a strong Effectiveness score never substitutes for a weak
Cogency score, which is the entire point of separating them.

## 4. Evaluation methodology

A rubric is only as good as the agreement it produces between independent scorers. The companion
repository [`rubric-eval-agreement`](https://github.com/IV7L/rubric-eval-agreement) implements the
measurement methodology this document assumes:

- **Blind double-scoring**: two scorers assess the same argument with no access to each other's
  scores.
- **Precise mode**: each (argument, dimension) pair is scored three independent times, resolved to
  one consensus score by a documented, deterministic tie-break rule.
- **Quadratic weighted kappa**, computed two independent ways as a cross-check, with bootstrap
  confidence intervals rather than a single point estimate.
- **Disagreement diagnostics**: which dimension has the lowest agreement, and whether disagreement
  concentrates on genuinely ambiguous cases rather than being noise -- the signal that tells you
  which dimension's anchors need to be rewritten before the rubric is trusted at scale.

That repository validates the measurement methodology on synthetic data (a generic domain,
support-reply quality, with the same dimension/anchor/precise-mode/kappa structure described
above). The rubric in Section 3 of this document is the same design pattern applied to argument
quality specifically.

## 5. Worked illustration

Consider a short, generic policy argument: *"The city should add a protected bike lane on Main
Street, because a nearby city saw a 30% drop in cyclist injuries after installing one."*

- **Claim clarity**: 4 -- the claim ("the city should add a protected bike lane on Main Street") is
  explicit and unambiguous.
- **Grounds-warrant connection**: 2 -- the grounds (another city's injury statistic) are given, but
  the warrant ("what worked there will work here") is left implicit and unexamined -- no mention of
  whether Main Street's traffic conditions resemble the comparison city's.
- **Rebuttal awareness**: 1 -- no acknowledgment that the comparison city's result might not
  generalize (different traffic volume, different lane design, confounding factors in the same
  period).
- **Scheme-critical-question coverage** (this is an *argument from example/analogy* in Walton's
  catalogue): 1 of the scheme's critical questions -- "is the cited case actually similar in the
  relevant respects?" -- is left unaddressed, so the score here is low (e.g. 1/4).
- **Opposing-position engagement**: 0 -- no opposing concern (e.g. effect on traffic congestion or
  parking) is mentioned at all.
- **Effectiveness**: 3 -- the argument is clear and easy to follow for a general audience, even
  though it is logically thin.

An aggregate score computed as a plain average would land around 1.8/4 -- but *why* is immediately
legible from the table, which is the actual deliverable: not the number, the auditable breakdown
behind it.

## 6. Limitations and future work

- **Scheme identification is itself a judgment call.** Before scheme-specific critical questions
  can be applied, a scorer (human or automated) must first correctly identify which of Walton's
  dozens of schemes the argument is using -- a step this document does not operationalize and
  which is its own open problem in computational argumentation.
- **Weight elicitation is not addressed here.** Section 2.3 recommends eliciting criterion weights
  from stakeholders and sensitivity-testing them; this document proposes the rubric's criteria, not
  a specific weight set or elicitation process for a specific deployment.
- **The worked example in Section 5 is illustrative, not a validation study.** A real validation
  would need a representative argument sample, multiple independent scorers, and the agreement
  methodology in Section 4 run to completion -- exactly what the companion repository demonstrates
  on synthetic data, and what a real deployment would need to run on real, human-scored arguments
  before the rubric is trusted.
- **Calibration is an ongoing process, not a one-time step.** Per Section 4's diagnostics, any
  dimension with persistently low inter-rater agreement needs its anchors rewritten and re-tested,
  not just flagged once.

## References

- Toulmin, S. E. (1958). *The Uses of Argument*. Cambridge University Press.
- Walton, D. (1996). *Argumentation Schemes for Presumptive Reasoning*. Lawrence Erlbaum Associates.
- Wachsmuth, H., Naderi, N., Hou, Y., Bilu, Y., Prabhakaran, V., Thijm, T. A., Hirst, G., & Stein,
  B. (2017). Computational Argumentation Quality Assessment in Natural Language. *Proceedings of
  the 15th Conference of the European Chapter of the Association for Computational Linguistics
  (EACL)*.
- Saaty, T. L. (1980). *The Analytic Hierarchy Process*. McGraw-Hill.
- Belton, V., & Stewart, T. (2002). *Multiple Criteria Decision Analysis: An Integrated Approach*.
  Kluwer Academic Publishers.
- Cohen, J. (1968). Weighted kappa: Nominal scale agreement with provision for scaled disagreement
  or partial credit. *Psychological Bulletin*, 70(4), 213-220.
