# Peer Design Review Notes — D2 Detailed Design

**Project:** World Action Models for Cross-Embodiment Robot Control  
**Team:** Joshua Jerin and Krrish Thakku Suresh  
**Review date:** October 14, 2026  
**Evidence status:** No external reviewing-team notes or names were present in the supplied project files or Canvas submission history. The action items below come from a rubric-based internal pre-submission design audit and must not be represented as external peer feedback. Replace the review-source field if the team has actual in-class notes.

## Received review converted into action items

| ID | Criterion | Review source and issue | Action | Owner | Priority | Due |
|---|---|---|---|---|---|---|
| PR01 | Completeness | Internal rubric audit: C5 is described as having no new registry tables, but the design also requires durable ACK, fault, and audit evidence. Persistence ownership and failure behavior are not fully explicit. | Add a C4-owned append-only event record for session, command, ACK, watchdog, stop, and log-health events. State that C5 emits events but does not own durable storage. Add a crash/restart test to EC16/EC20. | Joshua (C4) with Krrish (C5) | Must-fix | Before implementation task breakdown |
| PR02 | Feasibility | Internal rubric audit: the 20 ms heartbeat and 50 ms command lifetime may be sensitive to user-space scheduling, JSON serialization, and driver latency. No measured jitter budget exists yet. | Build a loopback timing prototype and report median, p95, p99, and worst-case sense/infer/check/deliver latency. Keep hardware disabled until command-expiry and watchdog margins are validated under load. | Joshua and Krrish | Must-fix | Before hardware authorization |
| PR03 | Clarity | Internal rubric audit: AC-01.3 says below-threshold runs are “not evaluated,” while D2 proposes `evaluated_below_threshold`. The current wording could create incompatible status handling. | Define one evaluation-status enum and update the report schema, traceability row, C4 authorization gate, and UI wording. Preserve the score while separately recording hardware eligibility. | Joshua (C4/C1) | Must-fix | Before API freeze |
| PR04 | Risk | Internal rubric audit: volatile camera buffers and image-free logs reduce privacy risk, but temporary SDK files, caches, crash dumps, backups, and model-training artifacts are not covered by a complete data-flow inventory. | Create a camera-data flow diagram and retention/access checklist covering acquisition, shared memory, preprocessing, training, logs, caches, backups, and deletion verification. C1 approval must cover every retained copy. | Joshua (C1) | Must-fix | Before collecting new demonstrations |
| PR05 | Risk | Internal rubric audit: endpoint workspace checks do not prove that the intervening path or braking trajectory remains safe. This is correctly listed as G3 but needs a concrete closure test. | Add swept-volume and stopping-envelope simulation tests, measured braking-distance tests, exclusion-zone checks, and fault-injection cases. Require recorded approval before any physical trial. | Joshua (C4) | Must-fix | Before first physical trial |
| PR06 | Feasibility | Internal rubric audit: matched candidate/baseline evaluation is well specified, but unknown base-model pretraining exposure can invalidate a strong “unseen robot” claim. | Use three exposure labels (`known_seen`, `documented_unseen`, `unknown`), limit claims when exposure is unknown, and record supporting model-card or dataset evidence in the evaluation report. | Krrish (C2/C3) | Worth considering | Before final evaluation report |

## Feedback summary

**Most-discussed criterion:** Risk. Two of the six issues directly concern physical or privacy risk, and the timing issue also affects safe fail-closed behavior. The highest-priority design concern is proving that the entire commanded and braking path stays safe, not merely that an endpoint passes a limit check.

**Changes prompted by this audit:**

1. Clarify durable event ownership between C4 and C5 and extend EC16/EC20 with persistence-recovery tests.
2. Add a timing/jitter prototype as evidence for NF03 and NF04.
3. Resolve evaluation-status terminology in AC-01.3 and the C4 gate.
4. Expand the privacy review from raw images to every cache, backup, crash dump, and derived artifact.
5. Turn G3 into explicit swept-path, stopping-distance, and fault-injection acceptance tests.

## Issues raised while reviewing other teams

No verifiable notes from reviews of other teams were included in the available files. This field cannot be completed honestly without the reviewed team/project and the actual recommendations. If the team has classroom notes, add two entries in this format before submission:

1. **Reviewed team/project:** _[actual team]_ — **Issue:** _[actual issue]_ — **Recommendation:** _[actual recommendation and why it mattered]_.
2. **Reviewed team/project:** _[actual team]_ — **Issue:** _[actual issue]_ — **Recommendation:** _[actual recommendation and why it mattered]_.

## Team validation checklist

- [ ] Confirm or replace the review source with actual reviewing-team names.
- [ ] Add the two real issues raised while reviewing another team.
- [ ] Confirm owners and dates for PR01–PR06.
- [ ] Approve the status-enum correction and persistent-event ownership.
- [ ] Do not claim hardware readiness until PR02 and PR05 have evidence.
