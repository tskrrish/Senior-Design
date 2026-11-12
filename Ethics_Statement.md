# Ethics and Responsible-Use Statement

**Project:** World Action Models for Cross-Embodiment Robot Control  
**Team:** Joshua Jerin and Krrish Thakku Suresh  
**Advisor:** Huu Tri Nguyen  
**Approved:** November 11, 2026

This statement applies the team’s privacy, security, licensing, attribution, and AI-use rules to the design documented in D0–D2. The inspected repository is currently documentation-only: it contains no deployable application, collected dataset, trained checkpoint, robot SDK, or dependency manifest. Therefore, controls described below are requirements that must be implemented and verified before data collection or hardware operation; they are not claims that a production system already exists.

## 1. Privacy summary

The system is divided into C1 experiment setup, C2 model adaptation/inference, C3 simulation/evaluation, C4 motion safety, and C5 robot interface. D1 proposes a local SQLite registry with large artifacts stored as files, while D2 proposes local sockets and OS-level peer identity. The following policy covers every documented category crossing a D0–D2 interface.

| Data item | Source and purpose | Consent | Storage and access | Retention and deletion |
|---|---|---|---|---|
| RGB camera frames; frame/image reference, shape, type, ID, and timestamp | Camera/C5; live inference and approved demonstrations | Before any recording, the operator or participant receives a written notice describing purpose, fields, retention, access, and withdrawal. Recording requires affirmative consent; non-consenting people are excluded from the capture area. | Volatile bounded buffers by default. Approved recordings are stored only in private UC/team storage, readable by Joshua, Krrish, and the advisor when needed. Raw images are never written to ordinary logs. | Unselected/rejected frames: delete within 7 days. Accepted de-identified demonstrations: retain through final grading plus 90 days, then delete raw images, caches, backups, and local copies. Retain only non-identifying aggregate results and hashes. Withdrawal before de-identification triggers deletion within 7 days. |
| Joint position/velocity, sensor times, robot state, observations, and actions | Robot/C5; inference, reproducibility, and safety checks | Covered by the approved lab trial notice; no human identity is required. | Private project storage and volatile session memory; least-privilege access for team/advisor. | Step-level trial data and safety logs: one year after project completion for audit, then delete. Aggregated non-personal metrics may remain in the repository. |
| Robot ID, URDF path/hash, joint count/order, limits, configuration, and approval metadata | C1/C4; embodiment validation and safe operation | Asset owner/license authorization, not human-subject consent | Versioned local registry and project files; C1/C4-owned writes | Retain with the reproducibility package for one year; retain hashes and public documentation afterward only if licensing permits. |
| Dataset/demo IDs, paths/hashes, dimensions, step counts, validation status, rejection reason, reviewer/method/time | C1; training provenance, validation, and privacy eligibility | Dataset-specific consent/license recorded before status becomes `ready` | Private files plus local registry; team/advisor only | Rejected source files: 7 days. Accepted de-identified records: final grading plus 90 days. Minimal provenance hashes and non-identifying audit results: one year. |
| Request/job/checkpoint IDs; model/settings/code hashes; weights path/hash; status | C1/C2; reproducibility and inference | Use permitted only when source model/data licenses allow it | Private project storage; no credentials embedded in paths or configuration | Failed/partial weights: delete within 7 days. Final approved checkpoint and provenance: one year, then delete unless separately approved for research release. |
| Evaluation IDs, seeds, tasks, outcomes, failure reasons, retries, reports, and hashes | C3; matched evaluation and honest scoring | No personal data expected; dataset rights still apply | Registry and versioned JSON reports | Retain one year; public release contains aggregate results and non-identifying provenance only. |
| Trial/session IDs, operator pseudonym, timestamps, command sequence, heartbeat, ACK, decision/event codes, stop reason | C4/C5; authentication, replay rejection, and safety audit | Authorized lab operator acknowledges logging before trial | Metadata-only logs in private storage; raw imagery excluded | Retain one year for safety audit, then delete. Incident records may be retained longer only with advisor/UC approval. |
| SDK temporary files, caches, crash dumps, backups, and derived artifacts | Tools/runtime; secondary copies | Same authorization as the source data | Approved project directories only; no personal cloud sync | Included in the 7-day/90-day deletion jobs. Deletion is verified by an inventory of working copies, caches, and backups. |

**Minimization decision.** The optional persistent free-text operator `note` field from D1 is removed because unstructured text could contain names or unrelated personal information and is not needed for safety. It is replaced by an operator pseudonym and fixed reason codes. Public contact details are not copied into runtime records. Camera data remains volatile unless C1 records affirmative consent, source rights, a privacy review, and an expiration date.

**End-of-project plan.** At final grading plus 90 days, the documentation lead inventories team machines, UC storage, caches, backups, exported datasets, and checkpoints; deletes raw/identifiable data; records the deletion date and verifier; and retains only licensed source, non-identifying aggregate reports, hashes, and required course records. Any research continuation requires a new advisor-approved retention decision and renewed consent where applicable.

## 2. Security summary

The design is local-first and fail-closed. No default credentials are permitted. Components run under dedicated OS service identities; Unix-domain sockets are owned by the service group with mode `0660`; a user-supplied JSON name is never treated as authentication. Only the C4 safety service may authorize a trial, and only C5 may communicate with hardware.

| Self-audit question | Team answer / required mechanism |
|---|---|
| Default credentials? | None. Installation requires explicit service accounts and group membership; missing configuration prevents startup. Manufacturer defaults must be changed before connection. |
| Unauthenticated endpoints? | None are allowed. Local peers are checked through OS credentials and socket permissions. A remote interface is out of scope unless separately reviewed. |
| Authentication/session lifetime? | OS peer authentication plus a session bound to one trial, robot, configuration hash, and monotonically increasing sequence. A session ends on stop, fault, disconnect, or after 15 minutes, whichever occurs first; reauthorization is required. The documented 50 ms command lifetime is a safety freshness limit, not an identity-token lifetime. |
| Secret storage? | Secrets are excluded from source, configuration examples, logs, checkpoints, and datasets. If a future SDK requires a credential, it must be supplied through an OS keyring or protected environment file outside the repository, readable only by the service account. |
| HTTPS/TLS? | Current design uses same-host Unix sockets, so HTTPS is not applicable. Any future remote endpoint must use TLS 1.3 with certificate validation; plaintext HTTP and self-signed production certificates are prohibited. |
| Rate limiting? | Trial start/authentication requests: 5 per minute per OS identity. Administrative requests: 20 per second. The live command path is limited to the designed 20 Hz cycle and rejects replayed, stale, or out-of-session messages. |
| Logging and failure behavior? | Metadata-only audit records include session, command, ACK, watchdog, stop, and log-health events. Queue overflow, write failure, unknown live errors, stale state, lost heartbeat, or invalid ACK stops the trial and requires fresh authorization. |
| Hardware safety boundary? | Hardware remains disabled until swept-path, stopping-distance, E-stop, timing-tail, watchdog, and fault-injection evidence passes and is approved. Simulation success alone never authorizes motion. |

**Git-history audit performed November 11, 2026.** All fetched refs in the inspected clone were searched across 10 commits and 13 unique historical blobs for PEM private-key headers, AWS `AKIA` access-key IDs, GitHub token prefixes, and quoted password/API-key/client-secret assignments. The scan returned **zero matches**. This bounded pattern scan does not cover unfetched/deleted refs, untracked files, secrets embedded in images, or runtime credentials; it therefore supports, but does not prove, the absence of secrets. No credential values were tested or exposed.

## 3. Licensing summary

The repository’s own license is **MIT**. The copyright and permission notice must remain in copies or substantial portions. The repository currently has no implementation manifest, so the table distinguishes the existing artifact from planned or candidate dependencies. A component may not be added until its exact version, source, transitive dependencies, and asset/model/data terms are recorded.

| Component / asset | Version/status | SPDX/license | Type | Obligation and decision |
|---|---|---|---|---|
| Senior-Design repository documents | Inspected commits through `470ffafe` | MIT | Permissive | Preserve copyright and license notice. Team confirms authorship/permission for submitted documents. |
| Python | Design target 3.11+; pin before implementation | Python-2.0 | Permissive | Preserve required notices when redistributing Python. |
| SQLite | Planned; pin/build provenance before implementation | blessing / public-domain dedication | Permissive | Preserve the blessing/attribution when distributed; no server license conflict. |
| PyTorch | Planned framework; version not yet pinned | BSD-3-Clause | Permissive | Preserve copyright, conditions, and disclaimer. |
| MuJoCo | Planned simulator; version not yet pinned | Apache-2.0 | Permissive | Include license/NOTICE and mark modifications. |
| NumPy | Planned; version not yet pinned | BSD-3-Clause | Permissive | Preserve license notice. |
| Pinocchio | Planned; version not yet pinned | BSD-2-Clause | Permissive | Preserve copyright, conditions, and disclaimer. |
| Pydantic v2 | Planned; exact version not yet pinned | MIT | Permissive | Preserve MIT notice. |
| pytest | Planned test dependency; version not yet pinned | MIT | Permissive | Preserve MIT notice if redistributed. |
| Hypothesis | Planned test dependency; version not yet pinned | MPL-2.0 | Weak copyleft | Use as an unmodified development dependency. If MPL-covered files are modified/distributed, publish those files and preserve notices; it does not relicense separate project files. |
| statsmodels | Optional; only if Wilson rule is selected | BSD-3-Clause | Permissive | Preserve notice; omit if the team retains the point-estimate rule. |
| Octo / openpi / OpenVLA | Candidates only; none selected | MIT / Apache-2.0 / MIT code; OpenVLA weights may use non-SPDX Llama terms | Mixed | Not a dependency until selected. Review code and weight licenses separately; do not assume the repository MIT license covers model weights. |
| Dataset, URDF/meshes, robot SDK/driver, creative assets | None selected in inspected repository | Unknown until selected | Unresolved | Do not import, train on, distribute, or publish until source, version, author, license, consent, and use restrictions are recorded. |

**Compatibility decision.** The planned permissive dependencies are compatible with an MIT-licensed project when their notices are preserved. Hypothesis is the only identified copyleft dependency and is limited to testing; no MPL-covered source is currently modified or distributed. Candidate model weights and future datasets/assets are separate licensing decisions and can block release even when the surrounding code is MIT.

## 4. Attribution list

- **Team work:** Joshua Jerin and Krrish Thakku Suresh, *World Action Models for Cross-Embodiment Robot Control*, D0–D2 design documents and this statement. Advisor: Huu Tri Nguyen.
- **Project repository and license:** https://github.com/joshuajerin/Senior-Design and its `LICENSE` file.
- **Planned software sources:** Python — https://www.python.org/; SQLite — https://www.sqlite.org/; PyTorch — https://github.com/pytorch/pytorch; MuJoCo — https://github.com/google-deepmind/mujoco; NumPy — https://github.com/numpy/numpy; Pinocchio — https://github.com/stack-of-tasks/pinocchio; Pydantic — https://github.com/pydantic/pydantic; pytest — https://github.com/pytest-dev/pytest; Hypothesis — https://github.com/HypothesisWorks/hypothesis; statsmodels — https://github.com/statsmodels/statsmodels.
- **Candidate model sources:** Octo — https://github.com/octo-models/octo; openpi — https://github.com/Physical-Intelligence/openpi; OpenVLA — https://github.com/openvla/openvla. These are candidates, not claims of current use.
- **Adapted code:** No adapted external code was found in the documentation-only repository. Any future adapted snippet must identify author, URL, license, changed sections, and retained notice before merge.
- **Datasets and non-code assets:** None are selected or distributed in the inspected repository. Future datasets must record collection method, consent, restrictions, and license; icons, fonts, images, URDFs, and meshes must record creator, source URL, and license.
- **Review evidence:** `Peer_Review_Notes.md` is an internal rubric audit, not independent external peer feedback, and is attributed accordingly.

This same attribution list is linked from README Section 5. The biography document belongs to Krrish and is not attributed to Joshua.

## 5. AI-use disclosure

> “Generative AI tools may be used for drafting and code assistance but not for the design decisions or written analysis that are assessed in this course.”

| Tool | Used by | Task | Column | How verified |
|---|---|---|---|---|
| OpenAI assistant | Joshua Jerin | Formatting, organizing evidence already present in D0–D2, table layout, proofreading, and PDF preparation | Generally OK | Joshua compared the draft with the repository, assignment wording, and audit output; factual corrections were made before approval. |
| OpenAI assistant | Krrish Thakku Suresh | Minimal formatting and wording cleanup of the team’s supplied text | Generally OK | Krrish reviewed the final text and confirmed that formatting did not change the team’s decisions or technical meaning. |

No generative-AI output is presented as a substitute for the team’s architecture, data model, priorities, risk judgments, or design rationale. The team supplied and approved those decisions; AI assistance was limited to organization, formatting, evidence extraction, and proofreading.

**Joshua Jerin — typed confirmation (approved November 11, 2026):** I confirm that the architecture, data model, priorities, ethical judgments, and design rationale are the team’s own work.  
**Krrish Thakku Suresh — typed confirmation (approved November 11, 2026):** I confirm that the architecture, data model, priorities, ethical judgments, and design rationale are the team’s own work.
